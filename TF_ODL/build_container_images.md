# Building Container Images

### Cài đặt Kolla-ansible for development

#### Install dependency

1. For CentOS, install EPEL

```
sudo yum install epel-release
```

2. For Ubuntu, update package

```
sudo apt-get update
```

3. Install Python dependency:

For CentOS:

```
sudo yum install python-devel libffi-devel gcc openssl-devel libselinux-python
```

For Ubuntu:

```
sudo apt-get install python-dev libffi-dev gcc libssl-dev python-selinux python-setuptools
```

4. Install **pip**

For CentOS:

```
sudo yum install python-pip
```

For Ubuntu:

```
sudo apt-get install python-pip
```

5. Install Ansible

For CentOS:

```
sudo yum install ansible
```

For Ubuntu:

```
sudo apt-get install ansible
```

#### Install Kolla-ansible

1. Clone **kolla** và **kolla-ansible** từ git

```
git clone https://github.com/openstack/kolla
git clone https://github.com/openstack/kolla-ansible
```

2. Install requirement của **kolla** và **kolla-ansible**

```
sudo pip install -r kolla/requirements.txt
sudo pip install -r kolla-ansible/requirements.txt
```

3. Tạo thư mục **/etc/kolla/**

```
sudo mkdir -p /etc/kolla
sudo chown $USER:$USER /etc/kolla
```

4. Copy configuration files

- To **/etc/kolla** from kolla-ansible repo (globals.yml, passwords.yml) 

```
cp -r kolla-ansible/etc/kolla/* /etc/kolla
```

- To directory hiện tại từ inventory file của **kolla-ansible**

```
cp kolla-ansible/ansible/invetory/* . 
```
 
### Generating kolla-build.conf

**kolla-build.conf** chứa thông tin tùy chỉnh khi build image được lưu ở **etc/kolla/kolla-build.conf** và được copy đến **/etc/kolla/**

```
pip install tox
cd kolla/
tox -e genconfig
```

### Building kolla images

Thực hiện `python tools/build.py` để build image. Mặc định các image được build dựa theo CentOS image, hoặc được chỉ định distro với option `-b`

```
python tools/build.py -b ubuntu
```

Các distro hỗ trợ building image gồm:
- centos
- debian
- oraclelinux
- rhel
- ubuntu

Có thể chỉ định build tập con các image theo string, và các dependency của image.

```
python tools/build.py keystone
``` 

Tập các image có thể được xác định trong section **profiles** của file **kolla-build.conf** hoặc chỉ định bởi tham số **--profile** qua dòng lệnh. Kolla cung cấp một số kiểu xác định profile:
- infra infrastructure-related images 
- main core OpenStack images 
- aux các image phụ trợ như trove, magnum, ironic 
- default minimal set các image cho việc deploy 

```
[profile]
magnum = magnum,heat
```

hoặc 

```
python tools/build.py --profile magnum
```

hoặc 

```
[DEFAULT]
profile = magnum
```

Ta có thể push image được build tới Dockerhub repository hoặc local registry như sau

```
python tools/build.py -n myrepo --push
```
và

```
python tools/build.py --registry 192.168.60.12:5000 --push 
```

### Build OpenStack from source

Khi build image, có hai tùy chọn được chỉ định với option `-t`
- **binary** : OpenStack install từ apt/yum (default)
- **source** : OpenStack được cài đặt từ source code 

Vị trí của source code được ghi trong **etc/kolla/kolla-build.conf** có thể là: **url**, **git** hoặc **local**. **local** có thể là directory hoặc tarball của source.

```
[glance-base]
type = url
location = https://tarballs.openstack.org/glance/glance-master.tar.gz

[keystone-base]
type = git
location = https://git.openstack.org/openstack/keystone
reference = stable/mitaka

[heat-base]
type = local
location = /home/kolla/src/heat

[ironic-base]
type = local
location = /tmp/ironic.tar.gz

```

### Dockerfile Customisation 

Từ phiên bản Newton, kolla hỗ trợ Jinja2 cho phép chỉnh sửa Dockerfile được sử dụng để tạo Kolla image. Việc này làm rõ ràng quá trình build image, các extra package, tweaking setting, plugin.

#### Generic Customisation 

Cho phép chỉnh sửa bất kỳ **{% block ... %}**, vị trí các block được xác định trong các **Dockerfile.j2** trong cây *https://github.com/openstack/kolla/tree/master/docker*. Sử dụng file j2 để ghi đè các block khi build image.
**template-override.j2**

```
{% extends parent_template %}

# Horizon
{% block horizon_redhat_binary_setup %}
RUN useradd --user-group myuser
{% endblock %}
```

Thực hiện build image ghi đè tùy chọn của horizon

```
python tools/build.py --template-override template-overrides.j2 horizon
```

#### Package Customisation 


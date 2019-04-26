# Building Container Images

### Cài đặt Docker

- [Docker on Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
- [Docker on CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)

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
sudo pip install tox
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

Build các image chứa chuỗi string được truyền cùng các dependency:

```
python tools/build.py keystone
``` 

Ta có thể build tập các image được định sẵn trong section **profiles** của file **kolla-build.conf** hoặc truyền qua tham số với option **--profile** khi thực hiện build image. Các tùy chọn được xác định trong file *kolla/etc/kolla/kolla-build.conf*
- infra infrastructure-related images 
- main core OpenStack images 
- aux các image phụ trợ như trove, magnum, ironic 
- default minimal set các image cho việc deploy 

```
[profileis]
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
 
Packet là một phần của image có thể được ghi đè, thêm và xóa. Để thêm packet của horizon ta tạo file **template-overrides.j2**

```
{% extends parent_template %}

# Horizon
{% set horizon_packages_append = ['iproute'] %}
```

để build lại horizon image, ta thực hiện
```
python tools/build.py --template-override template-overrides.j2 horizon
```

Ta có thể thực hiện ghi đè, thêm, remove package bằng cách thêm tùy chọn trong kolla-build.conf
- **override**: thay thế default package với custom list
- **append**: thêm package tới default list
- **remove**: remove packet từ default list
#### Using a different base image

Ta có thể sử dụng base image khác qua command
```
python tools/build.py --base-image registry.access.redhat.com/rhel7/rhel --base rhel
```

#### Plugin Functionality 

Ta có thể thêm và cài đặt plugin cho các service bằng cách thêm vào hai block **image_name_footer** và **footer**. **image_name_footer** chỉ định thay đổi cho image, **footer** thể hiện thay đổi áp dụng cho mọi Dockerfile.
Để thêm plugin networking-cisco cho **neutron_server** image, ta có thểm thêm vào file **template-override**

```
{% extends parent_template %}

{% block neutron_server_footer %}
RUN git clone https://git.openstack.org/openstack/networking-cisco \
    && pip --no-cache-dir install networking-cisco
{% endblock %}
```

Những thay đổi sẽ được thêm vào mục lưu trữ **plugins-archive**.Để thêm plugin qua cấu hình ta thêm section trong /etc/kolla/kolla-build.conf theo format

```
[<image>-plugin-<plugin-name>]
```
```
[neutron-server-plugin-networking-cisco]
type = git
location = https://git.openstack.org/openstack/networking-cisco
reference = master
```
Sau khi buil, ta sẽ thấy thay đổi trong thư mục lưu trữ 

```
plugins-archive.tar
|__ plugins
    |__networking-cisco
```
và trong template


```
{% block neutron_server_footer %}
ADD plugins-archive /
pip --no-cache-dir install /plugins/*
{% endblock %}

```

#### Additions Functionality

Dockerfile còn cho phép cài đặt bổ sung vào image như thêm jenkins job build metadata. Các bổ sung được lưu tại **additions-archive**. 
**plugins-archive** được thêm khi cài đặt plugin còn **additons-archive** được thêm bởi Kolla user.

Để thêm bổ sung cho image, ta thêm section trong file **kolla-build.conf** theo format 

```
[<image>-additions-<additions-name>]
```
VD ta thêm **jenkins**

```
[neutron-server-additions-jenkins]
type = local
location = /path/to/your/jenkins/data
```
Cấu trúc thư mục thay đổi 

```
additions-archive.tar
|__ additions
    |__jenkins
```

Ta có thể tự tạo **additions-archive.tar** thay cho cách trên. Template sẽ thay đổi như sau

```
{% block neutron_server_footer %}
ADD additions-archive /
RUN cp /additions/jenkins/jenkins.json /jenkins.json
{% endblock %}
```
## Pull images

Ta có thể pull image từ local registry bằng cách thiết lập "docker_registry" "172.22.2.81:5000" trong "**/etc/kolla/globals.yml**"

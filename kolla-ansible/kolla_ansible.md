# Kolla ansible

Tài liệu mô tả quá trình cài đặt kolla-ansible và deploy trên máy chủ centos hoặc ubuntu thông qua một máy host.

## Cài đặt gói phụ thuộc 

### CentOS

1. Cài đặt EPEL 

```
sudo yum install epel-release
```

2. Cài đặt python và các phụ thuộc

```
sudo yum install python-devel libffi-devel gcc openssl-devel libselinux-python
```
#### Cài đặt phụ thuộc sử dụng môi trường ảo

1. Cài đặt virtualenv

```
sudo yum install python-virtualenv
```
2. Tạo môi trường ảo và kích hoạt

```
virtualenv /path/to/virtualenv
source /path/to/virtualenv/bin/activate
```
3. Cài đặt Ansible 

```
pip install ansible
```
#### Cài đặt phụ thuộc không sử dụng môi trường ảo

1. Cài đặt pip 

```
sudo yum install python-pip
```
2. Cài đặt Ansible

```
sudo yum install ansible
```





### Ubuntu

1. Cập nhật package

```
sudo apt-get update
```
2. Cài đặt python và các phụ thuộc 

```
sudo apt-get install python-dev libffi-dev gcc libssl-dev python-selinux python-setuptools
```
#### Cài đặt phụ thuộc sử dụng môi trường ảo

1. Cài đặt virtualenv

```
sudo apt-get install python-virtualenv
```
2. Tạo môi trường ảo và kích hoạt

```
virtualenv /path/to/virtualenv
source /path/to/virtualenv/bin/activate
```
3. Cài đặt Ansible 

```
pip install ansible
```

#### Cài đặt phụ thuộc không sử dụng môi trường ảo

1. Cài đặt pip 

```
sudo apt-get install python-pip
```
2. Cài đặt Ansible

```
sudo apt-get install ansible
```


## Install Kolla-ansible

Ta có thể cài đặt Kolla-ansible qua pip hoặc thực hiện clone repo trên github 

### Cài đặt Kolla-ansible để deployment hoặc evaluation

1. Cài đặt kolla-ansible

- Không sử dụng virtualenv 
```
sudo pip install kolla-ansible
```
- Sử dụng virtualenv 

```
pip install kolla-ansible
```
2. Tạo **/etc/kolla** và thực hiện phân quyền 

```
sudo mdir -p /etc/kolla
sudo chown $USER:$USER /etc/kolla
```
3. Copy **globals.yml** và **passwords.yml** tới thư mục **/etc/kolla** 

4. Copy **all-in-one** và **multinode** tới thư mục hiện tại 

### Cài đặt Kolla-ansible cho development

1. Clone repo **kolla** và **kolla-ansible** từ git

```
git clone https://github.com/openstack/kolla
git clone https://github.com/openstack/kolla-ansible
```
2. Cài đặt requirements của **kolla** và **kolla-ansible** 

- Không sử dụng môi trường ảo 

```
sudo pip install -r kolla/requirements.txt
sudo pip install -r kolla-ansible/requirements.txt
```

- Sử dụng môi trường ảo virtualenv 

```
pip install -r kolla/requirements.txt
pip install -r kolla-ansible/requirements.txt
```
3. Tạo **/etc/kolla**

```
sudo mkdir -p /etc/kolla
sudo chown $USER:$USER /etc/kolla
```
4. Copy file **globals.yml** và **passwords.yml** tới **/etc/kolla**

```
cp -r kolla-ansible/etc/kolla/* /etc/kolla
```
5. Đi tới thư mục chứa repo kolla ansible, thực hiện copy file inventory **all-in-one** và **multinode**

```
cp kolla-ansible/ansible/inventory/* .
```

## Cấu hình Ansible 

Cấu hình Ansible cho môi trường của bạn

## Cấu hình ban đầu 

### Inventory 

Các file này chỉ định host và group, xác định role, thông tin truy cập các node. Sử dụng **all-in-one** để cài đặt tất cả trên một node, sử dụng **multinode** khi cài đặt trên nhiều node.
Ví dụ với **multinode**

```
[control]
10.0.0.[10:12] ansible_user=ubuntu ansible_password=foobar ansible_become=true
# Ansible supports syntax like [10:12] - that means 10, 11 and 12.
# Become clause means "use sudo".

[network:children]
control
# when you specify group_name:children, it will use contents of group specified.

[compute]
10.0.0.[13:14] ansible_user=ubuntu ansible_password=foobar ansible_become=true

[monitoring]
10.0.0.10
# This group is for monitoring node.
# Fill it with one of the controllers' IP address or some others.

[storage:children]
compute

[deployment]
localhost       ansible_connection=local become=true
# use localhost and sudo
```

- Kiểm tra cấu hình inventory, thực hiện test ping 

```
ansible -i multinode all -m ping
```

#### Kolla passwords 

Password được sử dụng trong quá trình triển khai được lưu tại **/etc/kolla/passwords.yml**. Ban đầu các password để trống, ta phải thêm password bằng tay hoặc tự động bằng cách chạy

```
kolla-genpwd
```

với development
```
cd kolla-ansible/tools
./generate_passwords.py
```

#### Kolla globals.yml

Chứa các file cấu hình cho Kolla-ansible, chứa các tuỳ chọn cho triển khai Kolla-ansible 
- Kolla cung cấp các lựa chọn cho distro Linux trong container
  - Centos
  - Ubuntu
  - Oraclelinux
  - Debian
  - RHEL

```
kolla_base_distro: "centos"
```
- Chỉ định nguồn image 
  - **binary**: sử dụng repo từ apt, yum
  - **source**: sử dụng nguồn lưu trữ, git repo hoặc thư mục local

```
kolla_install_type: "source"
```
- Chỉ định Docker tag 

```
openstack_release: "rocky"
```
- Phiên bản OpenStack

```
openstack_release: "rocky"
```
- Networking
  - Network internal interface (management)

```
network_interface: "eth0"
```
  - Neutron external network có thể vlan, flat; interface cần được active mà không có địa chỉ IP. 

```
network_external_interface: "eth1"
```
  -  floating IP cho mangagement traffic, bắt buộc phải được thiết lập nếu **kolla_enable_tls_external** được set yes, mặc định sử dụng địa chỉ **network_interface**

```
kolla_internal_vip_address: "10.1.0.244"
```

- Thực hiện enable, disable các service

Các giá trị được comment là giá trị mặc định, ta có thể set cài đặt các service hay không bằng cách set **yes** hoặc **no**.

```
enable_cinder: "yes"
```

## Deployment

Sử dụng inventory file là **all-in-one** hoặc **multinode**

### Với cài đặt cho deployment hoặc ovaluation

1. Bootstrap server

```
kolla-ansible -i ./multinode bootstrap-servers
```
2. Thực hiện kiểm tra các host 

```
kolla-ansible -i ./multinode prechecks 
```

3. Thực hiện cài đặt OpenStack

```
kolla-ansible -i ./multinode deploy
```

### Với cài đặt development

1. Bootstrap server 

```
cd kolla-ansible/tools
./kolla-ansible -i ../../multinode bootstrap-servers
```
2. Kiểm tra các host

```
./kolla-ansible -i ../../multinode prechecks
```
3. Cài đặt OpenStack
```
./kolla-ansible -i ../../multinode deploy
```

## Sử dụng OpenStack 

1. Cài đặt OpenStack CLI 

```
pip install python-openstackclient python-glanceclient python-neutronclient
```

2. OpenStack sử dụng openrc file chứa thông tin đăng nhập cho admin, user. Thực hiện tạo các file openrc 

  - Với deployment hoặc evaluation

```
kolla-ansible post-deploy 
. /etc/kolla/admin-openrc.sh
```

  - Với development

```
cd kolla-ansible/tools
./kolla-ansible post-deploy
. /etc/kolla/admin-openrc.sh
```
3. Chạy script để tạo network, image 

  - Với deployment hoặc evaluation, chạy script **init-runonce**
    - trên CentOS
```
. /usr/share/kolla-ansible/init-runonce
```

    - trên Ubuntu 

```
. /usr/local/share/kolla-ansible/init-runonce
```

  - Với development
```
. kolla-ansible/tools/init-runonce
```



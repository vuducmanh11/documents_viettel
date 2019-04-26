Tài liệu mô tả các edit cần thiết để cài đặt openstack ha trên multinode server

## Nếu Server không có kết nối internet

- Move all file repo list, create local.repo
```
cd /etc/yum.repos.d
mkdir bk
mv * bk/
```

```
cat <<EOF >local.repo
[cent7_repo]
name=Cent7 Repo
baseurl=http://10.240.201.50:8081/repository/centos-7
enabled=1
gpgcheck=0
priority=10

[fedora_repo]
name=Fedora Repo
baseurl=http://10.240.201.50:8081/repository/fedora-epel/
enabled=1
gpgcheck=0
priority=20
EOF
```
- Config pip repository 
```
sudo mkdir -p ~/.config/pip
cat << EOF > ~/.config/pip/pip.conf
[global]
trusted-host = 10.240.201.50
index = http://10.240.201.50:8081/repository/pypi
index-url = http://10.240.201.50:8081/repository/pypi/simple
extra-index-url = http://10.240.201.50:8081/repository/pypi/simple
EOF
```

## Edit playbook

- Disable docker repo in 'kolla-ansible/ansible/roles/baremetal/defaults/main.yml'

```
enable_docker_repo: false
```

- Change roles/baremetal/defaults/main.yml:
    
bash
    docker_legacy_packages: false       # disable để cài đặt docker-ce thay cho docker-engine (local repo chưa có)
    
-   Cài đặt pip bằng yum  thay vì easy_install bằng cách thay đổi các cấu hình trong file kolla-ansible/ansible/roles/baremetal/tasks/install.yml:
    
```
# - name: Install pip
#   easy_install:
#     name: pip
#     virtualenv: "{{ virtualenv is none | ternary(omit, virtualenv) }}"
#     virtualenv_site_packages: "{{ virtualenv is none | ternary(omit, virtualenv_site_packages) }}"
#   become: True
#   when: easy_install_available

- name: Install pip
  package:
    name: "{{ item }}"
    state: present
    update_cache: yes
  become: True
  with_items:
    - python-pip
```
## Tạo VLAN giữa các server để tạo VIP 
- Playbook tạo VLAN

```
- name: Add the 802.1q module
  modprobe:
    name: 8021q
    state: present

- name: Create VLAN 103 ifcfg file
  copy:
    dest: /etc/sysconfig/network-scripts/ifcfg-bond0.103
    content: |
      NAME=bond0.103
      DEVICE=bond0.103
      ONPARENT=yes
      VLAN=yes
      BOOTPROTO=none
      NM_CONTROLLED=no
      IPADDR="{{ ADDR }}"
      PREFIX=24

- name: Restart network service
  systemd:
    name: network
    state: restarted
```


## Sau cài đặt 

- Sử dụng iptables để truy cập horizon qua interface khác 

```
iptables -A PREROUTING -t nat -i bond0.100 -p tcp --dport 80 -j DNAT --to 11.1.1.224:80
iptables -A FORWARD -p tcp -d 10.60.17.224 --dport 80 -j ACCEPT

# 11.1.1.224: địa chỉ của local machine của card network-interface trong /etc/kolla/globals.yml
# 10.60.17.224: địa chỉ card interface muốn truy cập horizon   
```

## Các lỗi đã gặp

- Lỗi do VIP

  + Kiểm tra dải mạng đang sử dụng kolla_internal_vip_address và kolla_external_vip_address có cho phép cấp phát địa chỉ không.
  + Kiểm tra network_interface cần truy cập được kolla_internal_vip_address

- Lỗi không phân giải được hostname 

  + Thực hiện bootstrap-servers lại để thay đổi /etc/hosts

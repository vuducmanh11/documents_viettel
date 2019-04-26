Tài liệu cài đặt KVM, tạo các VM, set network để thực hiện deploy multinode với các VM trên server.

## Install KVM
- Dowload package

```
yum install qemu-kvm libvirt libvirt-python libguestfs-tools virt-install
```
- Enable libvirtd

```
systemctl enable libvirtd
```

- Start libvirtd

```
systemctl start libvirtd
```
- Verify kvm

```
lsmod | grep -i kvm
```

### Fix error fail start libvirtd

- Failed to start Virtualization daemon
```
cd /var/run/
rm -f libvirtd.pid
systemctl start libvirtd
```
- Failed start VM 'virsh start vm1': 'error start vm on kvm : guest CPU doesn't match specification'
```
virsh edit vm1
```
  - change section cpu 

```
<cpu mode='host-model' check='partial'>
    <model fallback='allow'/>
</cpu>
```
## Create VM
### Create image,iso

- Download image centos or ubuntu on local computer

```
wget https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img
wget https://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud-1809.qcow2
```
- Scp image to server
```
scp -P 8922  CentOS-7-x86_64-GenericCloud-1809.qcow2 root@10.60.17.224:/root
```
- Resize image
```
qemu-img resize CentOS-7-x86_64-GenericCloud-1809.qcow2 20G
```
- Check infor image
```
qemu-img info CentOS-7-x86_64-GenericCloud-1809.qcow2
```
- Copy image to libvirt storage
```
qemu-img convert -f CentOS-7-x86_64-GenericCloud-1809.qcow2 /var/lib/libvirt/images/vm1.img
```
- Create cloud config file cloud-config.txt
```
#cloud-config
password: centos
chpasswd: { expire: False }
ssh_pwauth: True
hostname: vm1
```
- Install cloud tool 
```
sudo apt-get install cloud-init-tools
yum install cloud-utils
```
- Create iso file 
```
cloud-localds /var/lib/libvirt/images/vm1.iso cloud-config.txt
```
### Create network

On server

- Check network visible

```
virsh net-list
```
- Create network file
```
touch vm.xml
cat << EOF > ./vm.xml 
<network>
  <name>vm</name>
  <forward mode='nat'/>
  <bridge name='br0' stp='on' delay='0'/>
  <ip address='10.0.0.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='10.0.0.2' end='10.0.0.254'/>
    </dhcp>
  </ip>
</network>
EOF
```
- Create, start, autostart network

```
virsh net-define vm
virsh net-start vm
virsh net-autostart vm
```
- Check network
```
ifconfig br0
virsh net-list
```

### Build VM

- Install VM
```
virt-install --virt-type=kvm --name vm1 --ram 20000 --vcpus=4 --os-variant centos7.0 --virt-type=kvm  --network network=default,model=virtio --graphics vnc,listen=0.0.0.0 --noautoconsole --disk /var/lib/libvirt/images/centos7.img,device=disk,bus=virtio --disk /var/lib/libvirt/images/centos.iso,device=cdrom --cpu host
virt-install \
--name vm1 \
--cpu host \
--virt-type=kvm --ram 8000 --vcpus=4 --os-variant centos7.0 \
--network network=default,model=virtio --graphics vnc,listen=0.0.0.0 --noautoconsole \
--disk /var/lib/libvirt/images/vm1.img,device=disk,bus=virtio \
--disk /var/lib/libvirt/images/vm1.iso,device=cdrom 
```
- Move all file repo list, create local.repo on VM
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
- Config pip repository on VM
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
- Config network on VM
- Restart network
```
systemctl restart network
```


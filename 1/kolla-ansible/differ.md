# Loi khi cai dat VM tren server


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

# Loi deploy
- Lỗi do VIP

  + Kiểm tra dải mạng đang sử dụng kolla_internal_vip_address và kolla_external_vip_address có cho phép cấp phát địa chỉ không.
  + Kiểm tra network_interface cần truy cập được kolla_internal_vip_address

- Lỗi không phân giải được hostname 

  + Thực hiện bootstrap-servers lại để thay đổi /etc/hosts
- Loi Cinder
  + Khi enable cinder cần enable một backend cho cinder
  + Khi set **enable_cinder_backend_lvm: "yes",  xảy ra lỗi "the role 'module-load' was not found", nguyên nhân khi import role 'module-load' trong file kolla-ansible/ansible/roles/iscsi/tasks/config.yml **chưa fix được**


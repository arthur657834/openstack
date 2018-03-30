```yum -y install libguestfs-tools

解决读取镜像配置无权限的问题
sed -i 's/#group = "root"/group = "root"/g' /etc/libvirt/qemu.conf
sed -i 's/#user = "root"/user = "root"/g' /etc/libvirt/qemu.conf
systemctl restart libvirtd

guestfish  --rw -a openstack-img/rhel-guest-image-7.4-263.x86_64.qcow2 

run
list-filesystems
mount /dev/sda1 /
vi /etc/cloud/cloud.cfg
修改如下内容，也可修改其他文件

disable_root: 0
ssh_pwauth:   1
passwd: 'redhat123'

system_info:
  default_user:
    name: cloud-user
    lock_passwd: false
    plain_text_passwd: 'redhat123'
    
直接mont修改    
guestmount -a centos63_desktop.qcow2 -m /dev/vg_centosbase/lv_root --rw /mnt
```

其他方式:
 * virsh 
 * losetup
 * kpartx
 * qemu-nbd
    
 
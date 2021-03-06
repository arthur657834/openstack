```
配置:150G 4C16G

https://docs.openstack.org/developer/devstack/

local - 在stackrc被加载前，从local.conf中提取localrc 
post-config - 在二层服务配置之后，服务启动之前post-config将会执行 
extra - 在各project中主服务启动之后然后在extra.d中的任何文件被执行之前 
post-extra - 在extra.d文件被执行之后运行 
local.conf文件是严格按照这个顺序来处理的，元数据可能将会被多次定义，如果任何值被重复设定了那么在文件中最后出现的将会被使用。

ex1:
[[post-config|/etc/nova/nova.conf]]
[neutron]
allow_duplicate_networks = True
[[post-config|/etc/heat/heat.conf]]
[DEFAULT]
plugin_dirs=/opt/stack/gbpautomation/gbpautomation/heat

ex2:
可以使用/opt/stack/devstack/stackrc  定义的变量
[post-config|$NOVA_CONF]]
[scheduler]
discover_hosts_in_cells_interval = 2


cp samples/local.conf ./

FORCE=True ./stack.sh

./unstack.sh
FORCE=True ./stack.sh

https://docs.openstack.org/developer/ceilometer/install/development.html
openstack插件安装
edit local.conf: add the following line to the [[local|localrc]] section.

[[local|localrc]]
# Enable the Ceilometer devstack plugin
enable_plugin ceilometer https://git.openstack.org/openstack/ceilometer.git

openstack插件列表:
https://docs.openstack.org/horizon/latest/install/plugin-registry.html

openstack镜像列表
https://docs.openstack.org/image-guide/obtain-images.html

devstack安装指定版本
git branch -a   #显示本地分支
git checkout -b icehouse origin/stable/icehouse  #切换到指定icehouse分支或标签
git clone https://github.com/openstack-dev/devstack.git  #克隆devstack的Git代码仓库
cd ./devstack

devstack添加日志:
log_dir = /var/log/nova
chown -R stack:stack /var/log/nova
```

### 查看命令
```
source openrc admin admin
source openrc demo demo
```

### devstack 重启机器后重启服务
```shell
systemctl enable httpd
systemctl start httpd
ifconfig br-ex 172.24.4.1/24
iptables -t nat -I POSTROUTING -s 172.24.4.0/24 -j MASQUERADE
iptables -I FORWARD -s 172.24.4.0/24 -j ACCEPT
iptables -I FORWARD -d 172.24.4.0/24 -j ACCEPT
```

### 修复不能新建卷的问题:
```
pvcreate /dev/sdb /dev/sdc
pvdisplay <==> pvs

vgcreate stack-volumes-lvmdriver-1 /dev/sdb /dev/sdc 
vgdisplay <==> vgs

新建卷组的名字可以通过以下命令获得
[root@200-openstack ~]# grep -nir volume_ /etc/cinder/cinder.conf 
20:osapi_volume_workers = 2
26:default_volume_type = lvmdriver-1
31:osapi_volume_listen = 0.0.0.0
32:osapi_volume_extension = cinder.api.contrib.standard_extensions
51:image_volume_cache_enabled = True
52:volume_clear = zero
55:volume_group = stack-volumes-lvmdriver-1
56:volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
57:volume_backend_name = lvmdriver-1
```
### 添加cinder-backup服务
```
mkdir /opt/stack/backup_mount
chown -R stack:stack /opt/stack/backup_mount

vi /etc/cinder/cinder.conf 
backup_driver = cinder.backup.drivers.nfs
#backup_mount_point_base = /opt/stack/backup_mount
backup_mount_point_base = $state_path/backup_mount <==> /opt/stack/data/cinder/backup_mount/1c7ba13a73edd1dc2d678fd64e440f45
backup_share = 10.1.50.199:/data

/usr/bin/python /usr/bin/cinder-backup --config-file /etc/cinder/cinder.conf
systemctl restart devstack@c-*

开启界面备份按钮
vi /opt/stack/horizon/openstack_dashboard/local/local_settings.py 
OPENSTACK_CINDER_FEATURES = {
    'enable_backup': True,
}
systemctl restart httpd
```
### 添加ceph
```
http://docs.ceph.com/docs/master/install/get-packages/

vi /etc/yum.repos.d/ceph.repo 

[ceph]
name=Ceph packages for $basearch
baseurl=https://download.ceph.com/rpm-jewel/rhel7/$basearch
enabled=1
priority=2
gpgcheck=1
gpgkey=https://download.ceph.com/keys/release.asc

[ceph-noarch]
name=Ceph noarch packages
baseurl=https://download.ceph.com/rpm-jewel/rhel7/noarch
enabled=1
priority=2
gpgcheck=1
gpgkey=https://download.ceph.com/keys/release.asc

[ceph-source]
name=Ceph source packages
baseurl=https://download.ceph.com/rpm-jewel/rhel7/SRPMS
enabled=0
priority=2
gpgcheck=1
gpgkey=https://download.ceph.com/keys/release.asc

yum -y install python2-lz4 ceph
```

### 日志查看
journalctl -f --unit devstack@n-cpu.service --unit devstack@n-cond.service

### RDO安装
```
RDO是红帽Red Hat Enterprise Linux OpenStack Platform的社区版
yum update -y
yum install -y https://rdoproject.org/repos/rdo-release.rpm
yum install -y openstack-packstack
packstack --allinone 安装失败之后重新尝试
```

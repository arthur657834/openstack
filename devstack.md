```
配置:150G 4C16G

https://docs.openstack.org/developer/devstack/

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
https://docs.openstack.org/developer/devstack/plugin-registry.html

devstack安装指定版本
git branch -a   #显示本地分支
git checkout -b icehouse origin/stable/icehouse  #切换到指定icehouse分支或标签
git clone https://github.com/openstack-dev/devstack.git  #克隆devstack的Git代码仓库
cd ./devstack
```

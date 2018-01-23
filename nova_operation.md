![nova_operation](nova_operation.[png])

### Resize 分两种情况：
1. nova-scheduler 选择的目标节点与源节点是不同节点。操作过程跟上一节 Migrate 几乎完全一样，只是在目标节点启动 instance 的时候按新的 flavor 分配资源。 同时，因为要跨节点复制文件，也必须要保证 nova-compute 进程的启动用户（通常是 nova，也可能是 root，可以通过 ps 命令确认）能够在计算节点之间无密码访问。 

2. 目标节点与源节点是同一个节点。则不需要 migrate。

### Lock/Unlock
为了避免误操作，比如意外重启或删除 instance，可以将 instance  加锁。对被加锁（Lock）的 instance 执行重启等改变状态的操作会提示操作不允许。执行解锁（Unlock）操作后恢复正常。<br>
Lock/Unlock 操作都是在 nova-api 中进行的。

提示：
1.	admin 角色的用户不受 lock 的影响，及无论加锁与否都可以正常执行操作。
2.	根据默认 policy 的配置，任何用户都可以 unlock。也就是说如果发现 instance 被加锁了，可以通过 unlock 解锁，然后在执行操作。


### Pause/Suspend/Resume不同点
1.	Suspend 将 instance 的状态保存在磁盘上；Pause 是保存在内存中，所以 Resume 被 Pause 的 instance 要比 Suspend 快。
2.	Suspend 之后的 instance，其状态是 Shut Down；而被 Pause 的 instance 状态是Paused。
3.	虽然都是通过 Resume 操作恢复，Pause 对应的 Resume 在 OpenStack 内部被叫作 “Unpause”；Suspend 对应的 Resume 才是真正的 “Resume”。这个在日志中能体现出来。

### Snapshot	
备份 instance 到 Glance。产生的 image 可用于故障恢复，或者以此为模板部署新的 instance。

## 故障处理
故障处理有两种场景：计划内和计划外。

## 计划内
### Migrate 
支持共享存储和非共享存储<br>
Migrate 前必须满足一个条件：计算节点间需要配置 nova 用户无密码访问。 

### Live Migrate
与 Migrate 不同，Live Migrate 能不停机在线地迁移 instance，保证了业务的连续性。也支持共享存储和非共享存储（Block Migration）
```
tips: 条件
1. 源和目标节点的 CPU 类型要一致。 
2. 源和目标节点能相互识别对方的主机名称，比如可以在 /etc/hosts 中加入对方的条目。
3. 在源和目标节点的 /etc/nova/nova.conf 中指明在线迁移时使用 TCP 协议。
4. Instance 使用 config driver 保存其 metadata。在 Block Migration 过程中，该 config driver 也需要迁移到目标节点。由于目前 libvirt 只支持迁移 vfat 类型的 config driver，所以必须在 /etc/nova/nova.conf 中明确指明 launch instance 时创建 vfat 类型的 config driver。
3. 源和目标节点的 Libvirt TCP 远程监听服务得打开，需要在下面两个配置文件中做一点配置。
/etc/default/libvirt-bin
/etc/libvirt/libvirtd.conf
service libvirt-bin restart
```

### Shelve/Unshelve	
Shelve 将 instance 保存到 Glance 上，释放Hypervisor 在宿主机上为其预留了资源。之后可通过 Unshelve 重新部署。<br>
Shelve 操作成功后，instance 会从原来的计算节点上删除。<br>
Unshelve 会重新选择节点部署，可能不是原节点。<br>

## 计划外
### Rescue/Unrescue
1. nova rescue c2
  - 关闭 instance 
  - 通过 image 创建新的引导盘，命名为 disk.rescue
  - 启动 instance

2. nova unrescue c2
  - 切换原始启动盘启动instance

### Rebuild
snapshot恢复

### revert
反转操作

### Evacuate 
可在 nova-compute 无法工作的情况下将节点上的 instance 迁移到其他计算节点上。但有个前提：Instance 的镜像文件必须放在共享存储上。 <br>
nova evacuate c2 --on-shared-storage





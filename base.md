### 虚拟化
虚拟化使得在一台物理的服务器上可以跑多台虚拟机，虚拟机共享物理机的 CPU、内存、IO 硬件资源，但逻辑上虚拟机之间是相互隔离的。通过Hypervisor 的程序实现的。

### Hypervisor 的实现方式和所处的位置，虚拟化又分为两种：
1. Hypervisor 直接安装在物理机上，多个虚拟机在 Hypervisor 上运行。Hypervisor 实现方式一般是一个特殊定制的 Linux 系统。Xen 和 VMWare 的 ESXi 都属于这个类型。

2. 物理机上首先安装常规的操作系统，比如 Redhat、Ubuntu 和 Windows。Hypervisor 作为 OS 上的一个程序模块运行，并对管理虚拟机进行管理。KVM、VirtualBox 和 VMWare Workstation 都属于这个类型。
> KVM 全称是 Kernel-Based Virtual Machine。也就是说 KVM 是基于 Linux 内核实现的。
KVM有一个内核模块叫 kvm.ko，只用于管理虚拟 CPU 和内存。<br>
IO 的虚拟化由Linux 内核和Qemu来实现
>> Libvirt
Libvirt 包含 3 个东西：后台 daemon 程序 libvirtd、API 库和命令行工具 virsh
>> 1. libvirtd是服务程序，接收和处理 API 请求；
>> 2. API 库使得其他人可以开发基于 Libvirt 的高级工具，比如 virt-manager，这是个图形化的 KVM 管理工具，后面我们也会介绍；
>> 3. virsh 是我们经常要用的 KVM 命令行工具，后面会有使用的示例。
  
### CPU overcommit
虚机的 vCPU 总数可以超过物理 CPU 数量

### 内存虚拟化
KVM 需要实现 VA（虚拟内存） -> PA（物理内存） -> MA（机器内存）直接的地址转换。虚机 OS 控制虚拟地址到客户内存物理地址的映射 （VA -> PA），但是虚机 OS 不能直接访问实际机器内存，因此 KVM 需要负责映射客户物理内存到实际机器内存 （PA -> MA）。具体的实现就不做过多介绍了，大家有兴趣可以查查资料。<br>
内存也是可以 overcommit 的，即所有虚机的内存之和可以超过宿主机的物理内存。

### 存储虚拟化
KVM 的存储虚拟化是通过存储池（Storage Pool）和卷（Volume）来管理的。<br>
Storage Pool 是宿主机上可以看到的一片存储空间，可以是多种类型，后面会详细讨论。Volume 是在 Storage Pool 中划分出的一块空间，宿主机将 Volume 分配给虚拟机，Volume 在虚拟机中看到的就是一块硬盘。

#### Storage Pool:
1. 目录类型
```
配置文件: /etc/libvirt/storage 
默认目录: /var/lib/libvirt/images 
优点: 存储方便、移植性好、可复制、可远程访问(网络共享)。
```
>Volume 文件格式:
>* raw 是默认格式，即原始磁盘镜像格式，移植性好，性能好，但大小固定，不能节省磁盘空间。
>* qcow2 是推荐使用的格式，cow 表示 copy on write，能够节省磁盘空间，支持 AES 加密，支持 zlib 压缩，支持多快照，功能很多。
>* vmdk 是 VMWare 的虚拟磁盘格式，也就是说 VMWare 虚机可以直接在 KVM上 运行。

2. LVM 类型
```
不仅一个文件可以分配给客户机作为虚拟磁盘，宿主机上 VG 中的 LV 也可以作为虚拟磁盘分配给虚拟机使用。
不过，LV 由于没有磁盘的 MBR 引导记录，不能作为虚拟机的启动盘，只能作为数据盘使用。
这种配置下，宿主机上的 VG 就是一个 Storage Pool，VG 中的 LV 就是 Volume。
优点: 较好的性能；
缺点: 管理和移动性方面不如镜像文件，而且不能通过网络远程使用。
```

### 网络虚拟化
1. Linux Bridge
可以通过配置文件实现，也可以通过brctl实现

2. vlan
可以通过配置文件实现

### 云平台
```
IaaS（Infrastructure as a Service）提供的服务是虚拟机。
IaaS 负责管理虚机的生命周期，包括创建、修改、备份、启停、销毁等。
使用者从云平台得到的是一个已经安装好镜像（操作系统+其他预装软件）的虚拟机。
使用者需要关心虚机的类型（OS）和配置（CPU、内存、磁盘），并且自己负责部署上层的中间件和应用。
IaaS 的使用者通常是数据中心的系统管理员。
典型的 IaaS 例子有 AWS、Rackspace、阿里云等

PaaS（Platform as a Service）提供的服务是应用的运行环境和一系列中间件服务（比如数据库、消息队列等）。
使用者只需专注应用的开发，并将自己的应用和数据部署到PaaS环境中。
PaaS负责保证这些服务的可用性和性能。
PaaS的使用者通常是应用的开发人员。
典型的 PaaS 有 Google App Engine、IBM BlueMix 等

SaaS（Software as a Service）提供的是应用服务。
使用者只需要登录并使用应用，无需关心应用使用什么技术实现，也不需要关系应用部署在哪里。
SaaS的使用者通常是应用的最终用户。
典型的 SaaS 有 Google Gmail、Salesforce 等
```

![虚拟网络逻辑图] (虚拟网络逻辑图.jpg)

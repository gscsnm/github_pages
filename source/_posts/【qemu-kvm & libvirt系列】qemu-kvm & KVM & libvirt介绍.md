
----

title: 【qemu-kvm & libvirt系列】qemu-kvm & KVM & libvirt介绍
date: 2016-09-20  10:00:27
tags: libvirt, kvm, qemu-kvm

----

感谢朋友支持本博客，欢迎共同探讨交流。
由于能力和时间有限，错误之处在所难免，欢迎指正！
原创作品，允许转载，转载时请务必以超链接形式标明文章原始出处 、作者信息和本声明。如果转载，请保留作者信息。
博客地址：https://gscsnm.github.io 
邮箱地址：gscsnm@gmail.com

----



> 目录：
- [虚拟化模型介绍](#虚拟化模型介绍)
- [KVM & QEMU & Libvirt介绍](#kvm-qemu-libvirt介绍)
	- [KVM](#kvm)
	- [QEMU](#qemu)
	- [Libvirt](#libvirt)
	- [区别](#区别)
	- [三者关系图](#三者关系图)

----

# 虚拟化模型介绍
如图所示：
底层是物理系统，就是一些硬件，cpu、内存、io等等。
在物理系统之上，运行着监控器，Hypervisor，管理物理硬件平台。
上层才是虚拟机，虚拟机内有虚拟的硬件，这样保证虚拟机间的隔离。虚拟机可以运行自己的操作系统。
![虚拟化模型图](/pictures/虚拟化模型.png)
图1：虚拟化模型图

----

# KVM & QEMU & Libvirt介绍

## KVM
(Kernel-based Virtual Machine的简称)，
是一个开源的系统虚拟化模块，自Linux2.6.20之后集成在Linux的各个主要发行版本中。
它使用Linux自身的调度器进行管理，所以相对于Xen，其核心源码很少。KVM目前已成为学术界的主流VMM之一。
KVM 仅仅是 Linux 内核的一个模块。管理和创建完整的 KVM 虚拟机，需要更多的辅助工具。所以现在的Linux中KVM不用安装。
KVM架构图如下所示，KVM是Linux内核空间的一个模块，对硬件进行管理，KVM本身不执行任何设备虚拟。
Linux用户空间中有个QEMU（下文介绍），负责虚拟机的创建和运行等。
![KVM架构图](/pictures/KVM架构图.png)
图2：KVM架构图

## QEMU
（Quick Emulator），
是一个独立的虚拟软件，能独立运行虚拟机（根本不需要 kvm ）。
kqemu 是该软件的加速软件。kvm 并不需要 qemu 进行虚拟处理，
只是需要它的上层管理界面进行虚拟机控制。虚拟机依旧是由 kvm 驱动。
所以，大家不要把概念弄错了，盲目的安装 qemu 和 kqemu 。

## Libvirt
是管理虚拟机和其他虚拟化功能，比如存储管理，网络管理的软件集合。
它包括一个API库，一个守护程序（libvirtd）和一个命令行工具（virsh）.
libvirt本身构建于一种抽象的概念之上。它为受支持的虚拟机监控程序实现的常用功能提供通用的API。
libvirt的主要目标是为各种虚拟化工具提供一套方便、可靠的编程接口，用一种单一的方式管理多种不同的虚拟化提供方式。

## 区别
kvm是最底层的hypervisor，它是用来模拟CPU和内存，但是kvm不进行虚拟机的生命周期操作（创建、启动等等）。QEMU-KVM就是一个完整的模拟器，它是建基于KVM上面的，它提供了完整的网络和I/O支持。一般不直接控制qemu-kvm，会用一个叫libvit的库去间接控制qemu-lvm， libvirt提供了夸VM平台的功能，还提供了一些高级的功能，例如pool/vol管理。

## 三者关系图
![三者关系图](/pictures/kvm-qemu_kvm-libvirt关系.png)


----
个人粗劣的理解，欢迎交流。gscsnm@gmail.com

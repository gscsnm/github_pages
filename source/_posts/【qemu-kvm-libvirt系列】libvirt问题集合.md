---
title: 【qemu-kvm & libvirt系列】libvirt问题集合
date: 2016-11-20 11:06:27
tags: libvirt; kvm; qemu-kvm 
---

感谢朋友支持本博客，欢迎共同探讨交流。
由于能力和时间有限，错误之处在所难免，欢迎指正！
原创作品，允许转载，转载时请务必以超链接形式标明文章原始出处 、作者信息和本声明。如果转载，请保留作者信息。
博客地址：https://gscsnm.github.io 
邮箱地址：gscsnm@gmail.com

----

由于近期使用libvirt，遇到了很多问题，记录一下。

----

> 目录（直达火箭）：
>
> - [Could not open 'xxxxx':Permission denied](#permission-denied)
> - [uid:107, gid:107 permission denied](#uid:107)
>



----
## Permission denied

<span id ="permission-denied"></span>

权限问题我遇到几次，对kvm不是很熟悉，所以你懂得....
先描述我的情况：  

第一种情况：

``` bash

[root@localhost kkk]# virsh define test1.xml   
Domain demo1 defined from test1.xml

[root@localhost kkk]# virsh start demo1    
error: Failed to start domain demo1
error: internal error: process exited while  connecting to monitor: 2022-08-18T10:14:48.384715Z qemu-system-x86_64: -drive file=/home/kkk/test2.img,if=none,id=drive-ide0-0-0,format=qcow2: Could not open ‘/home/kkk/test2.img’: Permission denied
```

第二种情况：

``` bash

[root@localhost kvm-imge]# virsh define test1.xml  
Domain demo1 defined from test1.xml

[root@localhost kvm-imge]# virsh start demo1  
error: Failed to start domain demo1
error: internal error: process exited while connecting to monitor: 2022-08-18T11:19:02.791629Z qemu-system-x86_64: -drive file=/home/kkk/xp_sp3_x86.iso,if=none,id=drive-ide0-0-1,readonly=on,format=raw: Could not open '/home/kkk/xp_sp3_x86.iso': Permission denied

```

### 具体分析


我的test1.xml里面定义了device：

```xml

<domain type="kvm"> 
    <name>demo1</name>  
    <devices> 
        <emulator>/usr/local/bin/qemu-system-x86_64</emulator>  
        <disk type="file" device="disk"> 
          <driver name="qemu" type="qcow2"/>  
          <source file="/home/kkk/test2.img" />    
          <target dev="hda" bus="ide"/> 
        </disk>  
        <disk type="file" device="cdrom"> 
          <source file="/home/kkk/xp_sp3_x86.iso"/>  
          <target dev="hdb" bus="ide"/> 
        </disk>  
    </devices> 
</domain>

```

定义了两个存储设备，一个disk，一个是cdrom。都是两个file，然后分别给了两个source，文件储存在/home/kkk/，kkk是我
登录的用户名。  
目录下文件权限如下：

```

[root@localhost home]# ll
total 4
drwx------. 7 kkk kkk 4096 Aug 18 19:16 kkk

[root@localhost kkk]# ll  
total 12241912
-rw-r--r--. 1 root    root        197120 Aug 17 23:06 test1.img
-rwxrwxrwx. 1 root    root        197120 Aug 17 23:07 test2.img
-rwxrwxrwx. 1 root    root      23724032 Aug 17 18:58 waf-1.qcow2
-rw-r--r--. 1 root    root    5940707328 Aug 17 18:10 waf_bak.qcow2
-rw-r--r--. 1 root    root    5940707328 Aug 13 00:46 waf.qcow2
-rw-r--r--. 1 root    root     630237184 May  3  2008 xp_sp3_x86.iso

```

主要是因为virsh用libvirt调用qemu-kvm进行虚拟机的创建操作，首先要根据xml文件定义一个虚拟机，然后启动start虚拟机。在启动虚拟机的时候qemu是用qemu这个用户去找文件的，由于qemu这个用户不属于kkk组，所以对test2.img和xp_sp3_x86.iso所在文件夹kkk没有访问权限，所以提示Permission denied。

### 解决方案


更改文件所在目录的拥有人和组为qemu。
由于我之前将文件放到了kkk用户的目录里了，不能修改拥有人为qemu，所以对文件进行移动。
将这两个文件移动到/kvm-image ，目录名可以自定义。
然后更改此目录的用户和用户组为qemu.   
命令如下:

``` bash

[root@localhost kkk]# mv test1.img /kvm-image/
[root@localhost kkk]# mv xp_sp3_x86.iso /kvm-image/
[root@localhost kkk]# mv test1.xml /kvm-image/
[root@localhost kkk]# chown qemu:qemu /kvm-image/ -R
[root@localhost /]# ll
................显示内容省略................
drwxr-xr-x. 2 qemu qemu  80 Aug 18 19:19 kvm-image
...............显示内容省略................

[root@localhost kkk]# cd /kvm-image/  
[root@localhost kvm-image]# ll  
total 615860  
-rw-r--r--. 1 root root    393216 Aug 18 20:43 test1.img  
-rw-r--r--. 1 root root      1497 Aug 18 20:43 test1.xml  
-rw-r--r--. 1 root root 630237184 Aug 18 19:19 xp_sp3_x86.iso  

```

可以看到，我将kvm-image这个目录的拥有人和组都换为了qemu，而里面的内容的拥有人和组保持没变。

### 结果


再次创建虚拟机：

``` bash

[root@localhost kvm-image]# virsh undefine demo1
Domain demo1 has been undefined

[root@localhost kvm-image]# virsh define test1.xml 
Domain demo1 defined from test1.xml

[root@localhost kvm-image]# virsh start demo1
Domain demo1 started

```

可以看到demo1正常运行了。

##  uid:107, gid:107 permission denied

<span id ="uid:107"></span>

### 描述

今天自己编译了spice-protocol spice-gtk spice qemu,然后想用virsh去创建一个虚机：

``` bash
[root@kkk]# virsh undefine demo1   
Domain demo1 has been undefined

[root@kkk]# virsh define /home/kkk/test.xml 
Domain demo1 defined from /home/kkk/test.xml

[root@kkk]# virsh start demo1
error: Failed to start domain demo1
error: Cannot access storage file '/home/kkk/test2.img' (as uid:107, gid:107): Permission denied

```

出现了uid107，gid107的错误。

### 解决方法：

Changing /etc/libvirt/qemu.conf make working things.
Uncomment user/group to work as root.  
把qemu.conf里面user和group前面的#去掉。

``` xml
# The user for QEMU processes run by the system instance. It can be
# specified as a user name or as a user id. The qemu driver will try to
# parse this value first as a name and then, if the name doesn't exist,
# as a user id.
#
# Since a sequence of digits is a valid user name, a leading plus sign
# can be used to ensure that a user id will not be interpreted as a user
# name.
#
# Some examples of valid values are:
#
#       user = "qemu"   # A user named "qemu"
#       user = "+0"     # Super user (uid=0)
#       user = "100"    # A user named "100" or a user with uid=100
#
user = "root"

# The group for QEMU processes run by the system instance. It can be
# specified in a similar way to user.
group = "root"

# Whether libvirt should dynamically change file ownership
# to match the configured user/group above. Defaults to 1.
# Set to 0 to disable file ownership changes.
#dynamic_ownership = 1
```

### 结果

  成功了。^_^



----

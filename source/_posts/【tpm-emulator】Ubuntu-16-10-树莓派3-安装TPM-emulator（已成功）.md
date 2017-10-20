---
title: 【tpm-emulator】Ubuntu-16-10-树莓派3-安装TPM-emulator（已成功）
date: 2017-05-05 14:23:52
tags: tpm_emulator 可信计算 
---

注：2017-7-14更新

---

# 0. 序  

最近搞可信计算方面，需要使用tpm模拟器，查阅不少资料，也看到了网上的教程。现将自己的安装步骤写个备注，方便自己查看。也希望对看官有所帮助。开干。

---*原创水印：https://gscsnm.github.io/*---

----

# 1. 环境
vmware 12  
Ubuntu 16.10 桌面版 （服务器版也行）

----

# 2. 安装cmake  

	sudo apt-get install cmake

<!--more-->
# 3. 安装GNU MP library  
两种方式安装：apt-get 、源码

### 3.1 apt-get 安装 （推荐）
3.1.1 切换用户到root（原谅我……）：
​
	su

3.1.2 搜索libgmp：   

	apt-cache search libgmp
![这里写图片描述](http://img.blog.csdn.net/20170505105013081?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3Njc25t/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

3.1.3 上图中很多gmp的库，我安装的是libgmp3-dev：

	apt-get -y install libgmp3-dev

### 3.2 源码安装
官网：https://gmplib.org/
下载 gmp-5.0.5.tar.bz2

	tar -jxf gmp-5.0.5.tar.bz2
	./configure
	make
	make check
	make install
具体可以看INSTALL和README。


# 4. 安装TPM_emulator
官网应该是： https://developer.berlios.de ，现在已经打不开了。
我GitHub上有源码,tpm_emulator-0.7.4：
https://github.com/gscsnm/tpm-emulator   
这个是我从PeterHuewe那儿fork的，用这个就行，这个是修改后的，如果用官方的话，会报错。  
原因：tpm_emulator还是2011年那会儿开发的，那会linux内核还是3.xxx  ,现在linux内核更新后，net.h里面的一些函数也进行了更新，所以报错。
---*原创水印：https://gscsnm.github.io/*---

### 4.1 下载
	 https://github.com/gscsnm/tpm-emulator/archive/master.zip

	wget https://github.com/gscsnm/tpm-emulator/archive/master.zip

### 4.2 解压

	unzip master.zip
如果没有unzip，请安装。apt-get install unzip

### 4.3 安装

	cd tpm-emulator-master/
	mkdir build;cd build
	cmake ../
	make
	sudo make install

	----
	make install的时候需要root权限复制文件。
	参考README文件
	** 如果遇到错误，看文章结尾的错误记录。 **

### 4.4 初始化、启动TPM_emulator

初始化tpm：

	tpmd deactivated
	killall tpmd
	tpmd clear    

启动tpm：  

	depmod -a
	modprobe tpmd_dev
	tpmd -f -d

tpmd -h：可以查看启动参数
如果启动出错，可以用  `tpmd –f –d clear  `

错误：`failed: address already in use`
解决：运行命令`rm /var/run/tpm/tpmd_socket:0`

如果成功，会不断打印：

Debug: waiting for connections...

### tpm_emulator 安装成功。

----


# 5. 安装TSS协议栈  
这里指的是Trousers。Trousers是一个开源可信计算软件栈，它从下到上依次为TDDL（驱动库）、TCS（核心服务）、TSP（服务提供者），tcsd是编译完毕之后的一个用户态可执行文件，作为守护进程运行。tcsd需在系统启动初始化阶段运行，这样以后所有要操作tpm的程序都必须经过TSS栈，这也是TCG规范所规定的。TDDL负责唯一打开TPM设备，实现打开、关闭、收发指令数据、获取版本信息等，TCS提供核心服务，调用TDDL。TSP是给上层程序的接口，用户可以编写应用程序利用这些接口，实现操作TPM。
---*原创水印：https://gscsnm.github.io/*---

### 5.1 安装trousers
下载地址：
https://sourceforge.net/projects/trousers/files/

下载3.14版本：  
 https://sourceforge.net/projects/trousers/files/trousers/0.3.14/trousers-0.3.14.tar.gz/download

5.1.1 解压：

	mkdir  trousers
	cd trousers
	tar -xzvf trousers-0.3.14.tar.gz  


5.1.2 安装依赖：

	看README：

	  Packages needed to build:
	  automake > 1.4
	  autoconf > 1.4
	  pkgconfig
	  libtool
	  gtk2-devel
	  openssl-devel >= 0.9.7
	  pthreads library (glibc-devel)

	挨个儿安装：  
	用apt-get install 安装，可以先用apt-cahce search 搜索定位一下。  
	能装多少就装多少吧，不装感觉也没事，没出错。
	我安装了：
	apt-get install -y automake1.9 autoconf2.64 pkgconf pkg-config libtool gtk2-engines openssl libssl-dev glibc-doc


5.1.3 编译安装前，需要把它的tddl库改成tpm_emulator提供的库，修改几个文件：

5.1.3.1 修改`./src/tcsd/Makefile.am`：

 第4行：

	tcsd_LDADD=${top_builddir}/src/tcs/libtcs.a ${top_builddir}/src/tcs/libtddl.so -lpthread @CRYPTOLIB@  
修改为：

	tcsd_LDADD=${top_builddir}/src/tcs/libtcs.a /usr/local/lib/libtddl.so -lpthread @CRYPTOLIB@


5.1.3.2 修改`./src/tcsd/Makefile.in`:
第55,56行：

	tcsd_DEPENDENCIES = ${top_builddir}/src/tcs/libtcs.a \
		${top_builddir}/src/tddl/libtddl.a
 修改为：

	tcsd_DEPENDENCIES = ${top_builddir}/src/tcs/libtcs.a \
		/usr/local/lib/libtddl.so

注：上面修改中的`/usr/local/lib/`路径，需要根据libtddl.so文件位置决定。

 另：该版本文件夹中没有bootstrap，因此省略`sh bootstrap.sh `步骤

5.1.4 编译安装

	./configure
	make
	make install

问题：
1） `./configure`后报错：

	configure: error: openssl is currently the only supported crypto library for trousers.   
	Please install openssl from http://www.openssl.org or the -devel package from your distro  

​	 

解决：
安装openssl-devel：
​
	sudo apt-get install openssl
	sudo apt-get install libssl-dev

2）本来还有的，结果没记录。。(⊙﹏⊙)b   不过大家应该不会遇到。。

5.1.5 启动tcsd

先开一个终端，启动TPM_emulator。参考上文。
再开一个终端，启动tcsd。

	#tcsd -e -f

在启动TCSD之前，必须先启动tpm-emulator，否则会提示找不到设备.
如果提示如下，运行tpm的时候加上clear：`tpmd -f -d clear`：
```
TCSD TDDL ioctl: (25) Inappropriate ioctl for device
TCSD TDDL Falling back to Read/Write device support.
TCSD TCS ERROR: TCS GetCapability failed with result = 0x1c
```


### 5.2 安装tpm-tools

	apt-get install tpm-tools

### 5.3 检测安装是否成功

先开一个终端，启动TPM_emulator。参考上文。
再开一个终端，启动tcsd。参考上文。
再开一个终端，运行下面命令。

	在/usr/sbin目录下有3个关于tpm的命令运行如下
	cd /usr/sbin
	./tpm_version      #查看版本号
	./tpm_getpubek   #查看ek公钥
	./tpm_takeownership   #获取owner


注：如果都成功表明TPM模拟环境已经完全构建成功了。

## 6 与TPM_emulator交互


下载read.c 和 read_change.c文件，linux下编译运行。
下载地址：http://download.csdn.net/download/gscsnm/9898726

步骤：
	1.一个终端运行TPM_emulator
	2.一个终端运行trousers
	3.在新开一个终端：

编译 read.c   
		`gcc read.c -o read -ltspi` 	
运行read  
	    `./read`  
如果报错：  
	`./read: error while loading shared libraries: libtspi.so.1: cannot open shared object file: No such file or directory`
	则需要添加	/usr/local/lib	到	/etc/ld.so.conf file :
	`echo "/usr/local/lib" >>/etc/ld.so.conf`
	重新运行ok！
	read_change.c 一样~~




## 7 安装过程中，错误记录（后补）
### 7.1 安装TPM_emulator
（树莓派3 Linux raspberrypi 4.9.35-v7+ 安装的使用遇到的问题 ）

1）make，到98%出错。      
		---*原创水印：https://gscsnm.github.io/*---
	提示：
```
    make[4]: *** /lib/modules/4.9.35-v7+/build: No such file or directory.  Stop.
```

	不一定是 4.9.35-v7+ ，中间这个是你自己内核的版本号。可以用`uname -r `查看。
	原因及解决办法：
```
查看该目录下是否有build：
    cd /lib/modules/4.9.35-v7+/
如果没有的话，再查看/usr/src下是否有linux-headers-*：
    cd /usr/src
如果没有linux-headers，那么安装linux-headers:
安装这个请不要用国内的各大镜像，
因为可能没有最新的linux-headers。
我后来用树莓派安装tpm，
版本是Linux raspberrypi 4.9.35-v7+，
如果用国内的apt-get源安装的话，
就没有对应的linux-headers。
用`#deb-src http://archive.raspberrypi.org/debian/ jessie main ui`
这个源进行安装linux-headers.
	sudo apt-get install raspberrypi-kernel-headers    
```
重新 cmake  ，make install。
OK！

2）  make，到98%出错。
    ---*原创水印：https://gscsnm.github.io/*---
    提示：
```
    Makefile:628: arch/armv7l/Makefile: No such file or directory
make[4]: *** No rule to make target 'arch/armv7l/Makefile'.  Stop.
```
原因及解决办法：

```
因为我是用树莓派安装的，可能tpm没有针对 树莓派做特殊
的makefile内容，至少是没有armv7l的，
所以在make 和make install 的时候指定一
下架构就行了。如下：

   make ARCH=arm
   make install ARCH=arm
```
OK！

3）  make，到98%出错。
    ---*原创水印：https://gscsnm.github.io/*---
    提示：
```
/home/pi/Desktop/tpm-emulator-master/build/tpmd_dev/linux/tpmd_dev.c: In function ‘tpmd_handle_command’:
/home/pi/Desktop/tpm-emulator-master/build/tpmd_dev/linux/tpmd_dev.c:143:9: error: too many arguments to function ‘sock_recvmsg’
   res = sock_recvmsg(tpmd_sock, &msg, tpm_response.size, 0);
         ^
In file included from /home/pi/Desktop/tpm-emulator-master/build/tpmd_dev/linux/tpmd_dev.c:25:0:
./include/linux/net.h:228:5: note: declared here
 int sock_recvmsg(struct socket *sock, struct msghdr *msg, int flags);
```
原因及解决办法：

```
提示说
error: too many arguments to function ‘sock_recvmsg’，
说明源码里sock_recvmsg函数不适配这个linux 内核版本。
也就是说这个linux内核提供的sock_recvmsg函数与
源码里用的不一致~~
 都是树莓派的锅哈哈哈。。
 我下了个原版的tmp_emulator源码，然后make就过了。
 地址： [http://download.csdn.net/download/gscsnm/9898586](csdn下载)~~
```

---
title: 【翻译系列】libvirt开发指南-Python版-第三章-连接
date: 2016-09-20  11:04:27
tags: libvirt;开发指南; 教程; libvirt_Application_Development_Guides

---
感谢朋友支持本博客，欢迎共同探讨交流。
由于能力和时间有限，错误之处在所难免，欢迎指正！
原创作品，允许转载，转载时请务必以超链接形式标明文章原始出处 、作者信息和本声明。如果转载，请保留作者信息。
博客地址：https://gscsnm.github.io 
邮箱地址：gscsnm@gmail.com

------

由于近期使用libvirt，但是发现中文学习材料很少，故来大概翻译一下官方的材料，恶心我一个人就行了，方面大家。本人水平有限，凑合着看吧。部分句子或段落的翻译加上了个人理解，不喜勿喷。 
原文链接： 
[http://libvirt.org/docs/libvirt-appdev-guide-python/en-US/html/](http://libvirt.org/docs/libvirt-appdev-guide-python/en-US/html/)

----

# 第三章 连接

## 简介
在libvirt中，连接是每一个指令（动作action）和对象的基础。每个想要与libvirt交互的实体（entity），像virsh、virt-manager或一个使用libvirt库的程序，都需要首先获得一个与主机中libvirt守护进程的连接。这个连接不仅描述了虚拟化技术的类型(qemu,xen,uml等)，而且还描述了任何一种必要的身份认证方法。

## 3.1 概述

libvirt代理（libvirt agent）必须做的第一件事就是调用 **virInitialize** 函数，或调用一个Python libvirt的连接函数获得的一个实例**virConnect** 类。这个实例将用于后续的操作。Python libvirt模块连接到一个资源提供了三种不同的方法:
```python
conn = libvirt.open(name)
conn = libvirt.openAuth(uri, auth, flags)
conn = libvirt.openReadOnly(name)
```
其中参数***name*** 实际上是指连接Hypervisor的URI。前面几节中，2.2节“驱动程序模型” 和3.2.2节“远程uri” 提供详细的各种URI格式。如果URI是***None***，libvirt将应用启发式的探测一个合适的Hypervisor驱动程序。虽然这可能方便开发人员进行特定的测试，但是强烈建议不应该依赖于探测驱动的方式，因为它随时有可能改变。应用程序应该显式地给出URI进行请求连接。  

上述三种方法的区别是他们请求的方式，是否进行身份认证、授权的级别不同。  
如果创建连接成功，则返回
下面将分别介绍。


### 3.1.1 open

**open**函数会建立一个***读-写***连接，进行全部的读写访问。 它没有任何进行身份认证的回调函数，所以它需要应用程序本身提供的证书才会成功连接。
例3.1 使用**open**
```python
# Example-1.py 
from __future__ import print_function
import sys
import libvirt

conn = libvirt.open('qemu:///system')
if conn == None:
    print('Failed to open connection to qemu:///system', file=sys.stderr)
    exit(1)
conn.close()
exit(0)
```

上面的例子主要功能是，建立了一个可以操作系统qemu驱动程序的***读-写***连接，检查此连接以确保它是成功的，如果是的话将连接关闭。 
有关libvirt uri的更多信息，请参考[3.2节“URI格式” ]( )。

### 3.1.2. openReadOnly  

**openReadOnly**函数将建立一个 ***只读*** 连接。这样的连接可以调用的函数是有限制的，通常用于监视应用程序，不能进行更改操作。与**open**方法一样,这种方法不提供认证机制。
例3.2 openReadOnly 使用
```python
# Example-2.py
from __future__ import print_function
import sys
import libvirt

conn = libvirt.openReadOnly('qemu:///system')
if conn == None:
    print('Failed to open connection to qemu:///system', file=sys.stderr)
    exit(1)
conn.close()
exit(0)
```

上面的示例建立一个只读连接到系统qemu类型的Hypervisor，进行检查以确保它是成功的，如果是关闭连接。  
有关libvirt uri的更多信息，请参考[3.2节“URI格式” 。]()

### 3.1.3. openAuth
**openAuth** 函数是最灵活的,有效地取代之前的两个函数。它需要一个附加的参数提供一个Python **list**列表，其中包含了客户机应用程序的身份认证凭证。**flag** 参数允许应用程序请求一个只读连接通过***VIR_CONNECT_RO***。  
下面有一个Python程序的简单例子，使用**openAuth**方法，使用用户名和密码进行连接。与**open**方法一样，这种方法没有认证回调，依赖于凭证。

例3.3 openAuth使用
```python
# Example-3.py
from __future__ import print_function
import sys
import libvirt

SASL_USER = "my-super-user"
SASL_PASS = "my-super-pass"

def request_cred(credentials, user_data):
    for credential in credentials:
        if credential[0] == libvirt.VIR_CRED_AUTHNAME:
            credential[4] = SASL_USER
        elif credential[0] == libvirt.VIR_CRED_PASSPHRASE:
            credential[4] = SASL_PASS
    return 0

auth = [[libvirt.VIR_CRED_AUTHNAME, libvirt.VIR_CRED_PASSPHRASE], request_cred, None]

conn = libvirt.openAuth('qemu+tcp://localhost/system', auth, 0)
if conn == None:
    print('Failed to open connection to qemu+tcp://localhost/system', file=sys.stderr)
    exit(1)
conn.close()
exit(0)
```

测试上面的项目,必须进行以下配置:  
1. /etc/libvirt/libvirtd.conf
```
listen_tls = 0
listen_tcp = 1
auth_tcp = "sasl"
```

2. /etc/sasl2/libvirt.conf
```
mech_list: digest-md5
```

3. virt 用户已经添加到SASL数据库:

4. libvirtd已经开始 ——听

配置完以上内容，例3.3 “使用openAuth” 可以使用用户名和密码对libvirtd的进行读写访问。

与libvirt C接口不同，Python不提供自定义回调函数收集凭证。

### 3.1.4 close

连接必须通过调用**virConnection**类的**close**函数进行关闭。连接是采用引用计数的对象，所以应该有一个相应的函数来关闭每个创建的连接。  

连接采用引用计数，当创建(open，openAuth 等)时，计数器会增加；也会因为其他函数而临时的增加，主要是根据连接是否保持存活。**open** 函数调用应该有一个相应的**close**函数，所有其他引用将相应的操作完成后释放。  

Python中的引用计数，当一个类实例超出范围或当程序执行结束时，会自动减少。

>这部分根据理解的不好，放上原文各位自己品品，帮忙理解一下。(⊙﹏⊙)b
>
>A connection must be released by calling the close method of the virConnection class when no longer required. Connections are reference counted objects, so there should be a corresponding call to the close method for each open function call.  
>
>Connections are reference counted; the count is explicitly increased by the initial (open, openAuth, and the like); it is also temporarily increased by other methods that depend on the connection remaining alive. The open function call should have a matching close, and all other references will be released after the corresponding operation completes.  
>
>In Python reference counts can be automatically decreased when an class instance goes out of scope or when the program ends.  

例3.4 使用**close**
```python
# Example-5.py
from __future__ import print_function
import sys
import libvirt

conn1 = libvirt.open('qemu:///system')
if conn1 == None:
    print('Failed to open connection to qemu:///system', file=sys.stderr)
    exit(1)
conn2 = libvirt.open('qemu:///system')
if conn2 == None:
    print('Failed to open connection to qemu:///system', file=sys.stderr)
    exit(1)
conn1.close()
conn2.close()
exit(0)
```

还需要注意，每一个类的实例与一个连接(virDomain、virNetwork等)交互，也将获取一个引用连接。

----

## 3.2 URI格式

Libvirt使用统一资源标识符(URI)来标识Hypervisor的连接。本地和远程的Hypervisor都是用URI进行定位的。URI scheme和平path定义了要连接的Hypervisor，URI中host部分确定了Hypervisor的定位（located）。

### 3.2.1 本地URI（Local URIs）

>Local URI还是不翻译了吧，交本地url怪怪的。\(^o^)/~

Libvirt的local URIs 有下列形式之一:

```python
driver:///system
driver:///session
driver+unix:///system
driver+unix:///session
```

所有其他使用libvirt URIs的方法形式都是远程连接的，即使是连接到localhost（本地主机）。关于远程URIs详看 3.2.2节-“远程uri” 。

> **system 和 session **的解释 （以kvm为例子）  
> libvirt中kvm使用QEMU的驱动。QEMU是多实例驱动，提供了两个驱动，包括一个特权驱动（system）和非特权驱动（session）。system是系统范围内的；session是用户相关的。  
> 使用system特权驱动，需要root账号权限，这样在建立system连接后，权限最大，可以管理整个节点内的所有域。使用session驱动建立的连接，只能管理当前用户权限范围内的资源或域。



目前支持以下驱动（drivers）:
![python-3.2-1.png](/pictures/翻译-libvirt开发指南-python-3.2-1.png)

下面的例子展示了使用local URI连接到本地QEMU hypervisor。

例3.5 连接到一个本地QEMU Hypervisor

```python
# Example-6.py
from __future__ import print_function
import sys
import libvirt

conn = libvirt.open('qemu:///system')
if conn == None:
    print('Failed to open connection to qemu:///system', file=sys.stderr)
    exit(1)
conn.close()
exit(0)
```

### 3.2.2 远程URI（Remote URIs）

Remote uri的一般形式是(“\[… \]”表示可选的部分):  

```

driver[+transport]://
[username@][hostname][:port]/[path][?extraparameters]

```

每个组件的URI描述如下。
![python-3.2-2.png-URI描述](/pictures/翻译-libvirt开发指南-python-3.2-2.png)

>个人理解：
>* **driver**：这个就是Hypervisor的类型名称，跟local RUI一样，一般是 ***xen , qemu , lxc , openvz , test**  。有一个 **特例**，***remote*** 这个假的driver的名字可以使用，使用***remote***让远程Hypervisor在自己的电脑上查询有什么Hypervisor，选择一个使用。一般来说，如果应用程序知道hypervisor是什么类型，应该显式的制定驱动程序名称而不是依靠自动探测。
>* **transport**：数据传输协议，一般是 ***tls , tcp , unix , ssh*** 和 ***ext*** 。 如果省略不写的话，如果提供了hostname（主机名），将使用tls；没有提供hostname的话，将使用unix协议。
>* **username**：用户名。当使用***SSH***数据传输协议时，允许选择的一个不同于客户的当前的登录名的用户名。
>* **hostname**：主机名。限定远程主机的主机名。如果使用***TLS x509***证书，或带有GSSAPI / Keberos插件的***SASL***时，这是比较关键的，如果不能匹配主机名，将返回认证失败。
>* **port**：端口。很少需要这个参数，除非***SSH***或***libvirtd***配置为运行在一个非标准的TCP端口时。SSH默认端口为 22，TCP端口为16509，TLS端口是16514。
>* **path**：路径。path使用方法与local URI一样，定位Hypervisor的位置。对于***Xen***来说，默认总是  **/** ，而对于***QEMU***默认是 ***/system*** 。
>* **extraparameters**：附加参数。URI查询参数为了对远程连接的微调某些方面，下面详细讨论。


基于本文描述的信息和参照本文前面提到的Hypervisor特定URIs，现在展示一些示例remote URI。

```xml
连接到一个远程主机的Xen，
host: node.example.com ，
使用ssh数据传输协议和ssh用户名*root* :
xen + ssh://root@node.example.com/  

连接到一个远程主机QEMU ，
host: node.example.com ，
使用TLS x509证书:
qemu://node.example.com/system

连接到一个远程主机Xen,
host：node.example.com ，
使用TLS，跳过验证服务器的x509证书(注:这是不安全的): 
xen://node.example.com/?no_verify=1

连接到本地QEMU实例，
使用不标准的Unix socket(Unix套接字的完整路径提供显式):
qemu+unix:///system?socket=/opt/libvirt/run/libvirt/libvirt-sock

连接到一个libvirtd守护进程，
使用未加密的TCP/IP连接，
TCP端口5000 ，
使用自行探测的驱动程序默认配置:
test+tcp://node.example.com:5000/default

```

#### extraparameters(附加参数)

extraparameters可以被添加到远程URI，作为查询字符串的一部分，使用问号开始（`?no_verify=1`）。remote URI可以解析如下所示的参数，其他的参数不做修改，直接传给后台处理。注意，参数必须是URI-escaped。  
参考： 
[http://xmlsoft.org/html/libxml-uri.html](http://xmlsoft.org/html/libxml-uri.html#xmlURIEscapeStr)  

Extra parameters：
![python-3.2-3.png-附加参数描述](/pictures/翻译-libvirt开发指南-python-3.2-3.png)

>个人理解
>
>* **name**:任何协议可以通用。local hypervisor URI传递给remote virConnectOpen函数。这个URI通常是由传输协议、主机名、端口号、用户名和附件参数构成，但在某些非常复杂的情况下可能需要提供明确的名称。 例子:`name=qemu:///system`
>
>* **command**:ssh,ext协议使用。外部命令。 ext协议是必需参数。ssh协议默认值是ssh。PATH会对提供的命令进行搜索。例子:`command=/opt/openssh/bin/ssh`
>
>* **socket**: unix,ssh使用。外部命令。ext协议是必需的。ssh协议默认值是ssh。PATH会对提供的命令进行搜索 例子:
>  `socket=/opt/libvirt/run/libvirt/libvirt-sock`  
>
>  * **netcat**:ssh协议使用。这个就是linux中netcat命令。缺省值是nc。对于ssh协议来说，libvirt构造一个ssh命令的样式：
>    `command -p port [-l username] hostname netcat -U socket`  
>    可以指定端口、用户名、主机名作为远程URI的一部分,command、netcat和socket来自附加参数(或默认值)。 例子:   
>    `netcat=/opt/netcat/bin/nc`
>  * **no_verify**:tls协议使用。客户端知道服务器的证书是禁用，如果设置一个非零值。注意，禁用服务器检查客户机的证书或IP地址必须改变libvirtd配置。 例子: `no_verify = 1`
>  * **no_tty**:ssh协议使用。如果设置为非零值，会阻止ssh请求密码，如果不能自动登录到远程计算机(例如,当使用ssh-agent)。当你没有访问终端权限是使用这个。例如，使用libvirt的图形程序。例子: `no_tty = 1`

下面的例子展示了如何使用remote RUI连接到QEMU。

连接到一个远程QEMU
```python
# Example-7.py
from __future__ import print_function
import sys
import libvirt

conn = libvirt.open('qemu+tls://host2/system')
if conn == None:
    print('Failed to open connection to qemu+tls://host2/system', file=sys.stderr)
    exit(1)
conn.close()
exit(0)
```

----

## 3.3 功能信息的方法

### 概述

getCapabilities 方法可以用来获取虚拟化主机的性能信息。如果获取成功，将返回一个Python字符串，内容是包含性能信息的XML(在下面描述)。如果失败，返回None。下面的代码演示如何使用 getCapabilities 方法:  

例3.7。 使用getCapabilities
``` python

# Example-8.py
from __future__ import print_function
import sys
import libvirt

conn = libvirt.open('qemu:///system')
if conn == None:
    print('Failed to open connection to qemu:///system', file=sys.stderr)
    exit(1)

caps = conn.getCapabilities() # caps will be a string of XML
print('Capabilities:\n'+caps)

conn.close()
exit(0)
```

XML格式的性能信息提供了主机虚拟化技术的信息。特别是，它描述了虚拟主机的性能，虚拟化技术（driver），可以启动的Geust域的类型。注意性能XML会随着libvirt driver变化。一个例子:

例3.8. 以QEMU为例子

```xml
<capabilities>
 <host>
   <cpu>
     <arch>x86_64</arch>
   </cpu>
   <migration_features>
     <live/>
     <uri_transports>
       <uri_transport>tcp</uri_transport>
     </uri_transports>
   </migration_features>
   <topology>
     <cells num='1'>
       <cell id='0'>
         <cpus num='2'>
           <cpu id='0'/>
           <cpu id='1'/>
         </cpus>
       </cell>
     </cells>
   </topology>
 </host>

 <guest>
   <os_type>hvm</os_type>
   <arch name='i686'>
     <wordsize>32</wordsize>
     <emulator>/usr/bin/qemu</emulator>
     <machine>pc</machine>
     <machine>isapc</machine>
     <domain type='qemu'>
     </domain>
     <domain type='kvm'>
       <emulator>/usr/bin/qemu-kvm</emulator>
     </domain>
   </arch>
   <features>
     <pae/>
     <nonpae/>
     <acpi default='on' toggle='yes'/>
     <apic default='on' toggle='no'/>
   </features>
 </guest>

 <guest>
   <os_type>hvm</os_type>
   <arch name='x86_64'>
     <wordsize>64</wordsize>
     <emulator>/usr/bin/qemu-system-x86_64</emulator>
     <machine>pc</machine>
     <machine>isapc</machine>
     <domain type='qemu'>
     </domain>
     <domain type='kvm'>
       <emulator>/usr/bin/qemu-kvm</emulator>
     </domain>
   </arch>
   <features>
     <acpi default='on' toggle='yes'/>
     <apic default='on' toggle='no'/>
   </features>
 </guest>

</capabilities>

```


在性能XML里，总有 `/host` 子节点、0个或多个 `/`guest/子节点(没有`/guest`子节点，意味着不能在这个主机上启动Guest 域)。

`/host`子节点描述了主机的性能。  

`/host/uuid`显示了host的UUID。数据来自SMBIOS UUID，如果SMBIOS UUID是可用的、有效的，否则可以在`libvirtd.conf`配置自定义值。 如果上面的两种设置都无效，则会产生一个临时UUID，libvirtd每次重新启动都会从新生成。

`/host/cpu`子节点描述了主机的cpu的性能。当决定guest域是否可以正常创建时，会使用这个值。当动态迁移期间也用来确定目的机器可以有能力继续运行guest域。

`/host/cpu/arch` 描述主机CPU架构。截止到撰写本文时，所有libvirt驱动程序初始化这个节点时，使用uname(2)的输出值。  

`/host/cpu/features` 是一个可选的子节点，描述了cpu附加的功能特性。截止到撰写本文时，只有xen driver会使用它，to report on the presence or lack of the svm or vmx flag, and to report on the presence or lack of the pae flag.  

`/host/cpu/model` 描述主机CPU 模式的元素。CPU模型列表在`cpu_map.xml`文件查询。

`/host/cpu/feature` 0个或多个节点描述CPU附加功能不在`/host/cpu/model`中包含。  

`/host/migration_features`是一个可选的子节点，描述了迁移特性，如果driver支持的话。如果不存在，那么不支持迁移。到撰写本文时，xen、qemu、esx 驱动程序支持迁移。  

`/host/migration_features/live` 此元素存在的话，说明驱动程序支持动态迁移。

`/host/migration_feature/uri_transports` 是一个可选的元素，描述交替连接迁移机制。这些连接机制在多宿主的虚拟化系统是非常有用的。例如，virsh迁移命令可通过10.0.0.1连接到迁移源，10.0.0.2连接到迁移地。 然而，因为安全政策，迁移的来源可能只被允许通过192.168.0.0/24连接迁移地。在这种情况下，使用连接迁移机制将允许这种迁移成功。到撰写本文时，xen驱动程序支持替代移植机制“xenmigr”，而qemu驱动程序支持“tcp”迁移机制。请参阅文档了解的更多信息。  

`/host/topology` 子节点描述了主机的NUMA拓扑。每个NUMA节点由一个`/host/topology/cells/cell`表示，描述了哪些cpu在NUMA节点里。如果主机是乌玛(non-NUMA)机,然后会有只有一个细胞和所有cpu将细胞。 这是非常特定于硬件,所以在不同的机器之间必然不同。

`/host/secmodel` 是一个可选的子节点，描述的主机上使用的安全模型。`/host/secmodel/model` 显示安全模型的名称， `/host/secmodel/doi` 显示域的解释(the Domain Of Interpretation)。关于安全的更多信息，请参见安全部分。  

每一个 `/guest` 子节点描述一种主机上的driver可以启动的域。这个描述包括guest的架构及给guest提供的ABI(hvm、xen或uml)。(原文：This description includes the architecture of the guest (i.e. i686) along with the ABI provided to the guest (i.e. hvm, xen, or uml))  

`/guest/os_type` 是一个必需元素，描述的guest的类型。  

guest类型表：
![Guest Types](/pictures/翻译-libvirt开发指南-python-3.3-1.png)

`/guest/arch ` 是一个子节点的根，描述guest的各种虚拟硬件信息。有个单一的属性名为“name”，可以用来定位这个子节点。  

`/guest/arch/wordsize` 是一个必需的元素，描述一个word有多少位，通常是32或64。  

`/guest/arch/emulator` 是一个可选元素，描述这个guest类型的emulator默认路径。注意，该模拟器可以被 `/guest/arch/domain/emulator element`(在下面描述)覆盖(be overredden)。（注：emulator是模拟器。例如：qemu就是一个emulator。）

`/guest/arch/loader` 是一个可选元素，描述firmware（固件） loader的默认路径。注意，可以被`/guest/arch/domain/loader`(在下面描述)覆盖（overridden）。目前，这只是被HVM的guest的xen驱动使用。  

可以有0个或多个 `/guest/arch/machine` 元素，描述主机可以创建的域类型。这些“machine”通常代表了guest可以运行的ABI或硬件接口。注意，这些machine可以被`/guest/arch/domain/machine`元素(在下面描述)覆盖。典型值是“pc”、“isapc”，代表着PCI based PC, and an older, ISA based PC。  

可以有0个或多个 `/guest/arch/domain`子树，(尽管没有这个子树时，不能创建任何域实例)。每一个`/guest/arch/domain`子树有可选的< emulator >、< loader >、< machine >元素覆盖上述指定各自的默认值。 对于没有的元素，使用默认值。  

`/guest/features` 是一个可选你的子节点，描述了各种附加的特性，可以启用或禁用，及它们的默认状态，是否可以进行切换或关闭。 

----

## 3.4 主机信息
待翻译
### 概述

### 3.4.1. 

----

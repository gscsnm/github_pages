---
title: 【翻译系列】libvirt开发指南-Python版-目录
date: 2016-09-20  11:00:00
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

## 目录

- 前言
  - 文档约定
    - 排版约定
    - 引文 约定
    - 提醒和警告
  - 我们需要反馈!


1. 简介  
  1.1. 概述   
  1.2. 术语表
<!--more-->
2. 结构    
  2.1. 对象模型    
  2.1.1. 连接Hypervisor    
  2.1.2. 客户域Guest Domains  
  2.1.3. 虚拟网络  
  2.1.4. 存储池  
  2.1.5. 存储卷  
  2.1.6. 主机设备  
  2.2. 驱动程序模型  
  2.3. 远程管理  
  2.3.1. 基本用法  
  2.3.2. 数据传输  
  2.3.3. 身份验证方案  
  2.4. 生成TLS证书  
  2.4.1. 公钥的设置  

3. 连接  
  3.1. 概述  
  3.1.1. open  
  3.1.2. openReadOnly  
  3.1.3. openAuth  
  3.1.4. close  
  3.2. URI格式  
  3.2.1. 本地URI  
  3.2.2. 远程URI  
  3.3. 性能信息的方法（Capability Information Methods）  
  3.4. 主机信息  
  3.4.1. getHostname  
  3.4.2. getMaxVcpus  
  3.4.3. getInfo  
  3.4.4. getCellsFreeMemory  
  3.4.5. getType  
  3.4.6. getVersion  
  3.4.7. getLibVersion  
  3.4.8. getURI  
  3.4.9. isEncrypted  
  3.4.10. isSecure  
  3.4.11. isAlive  
  3.4.12. compareCPU  
  3.4.13. getFreeMemory  
  3.4.14. getFreePages  
  3.4.15. getMemoryParameters  
  3.4.16. getMemoryStats  
  3.4.17. getSecurityModel  
  3.4.18. getSysinfo  
  3.4.19. getCPUMap  
  3.4.20. getCPUStats  
  3.4.21. getCPUModelNames  

4. 客户域（Guest Domains）  
  4.1. 域概述  
  4.2. 列出域（Listing Domains）  
  4.3. 获得域信息  
  4.3.1. 获取一个域的ID  
  4.3.2. 获取一个域的UUID  
  4.3.3. 获取域的操作系统类型  
  4.3.4. 确定当前域是否有一个快照  
  4.3.5. 确定域是否已经保存镜像  
  4.3.6. 获取域的主机名  
  4.3.7. 获取域的硬件信息  
  4.3.8. 确定域是否正在运行  
  4.3.9. 确定域是否是持久的  
  4.3.10. 确定域是否被更新过  
  4.3.11. 确定域的最大内存  
  4.3.12. 确定域的最大vcpu数量  
  4.3.13. 获取域的名字  
  4.3.14. 获取域的状态  
  4.3.15. 获取域的时间信息  
  4.4. 生命周期控制  
  4.4.1. 配置和启动  
  4.4.2. 停止  
  4.4.3. 暂停/恢复和保存/恢复  
  4.4.4. 迁移  
  4.4.5. 自动启动  
  4.5. 域配置  
  4.5.1. 引导模式  
  4.5.2. 内存/ CPU资源  
  4.6. 监控性能  
  4.6.1. 域的块设备性能  
  4.6.2. vCPU性能  
  4.6.3. 内存统计信息  
  4.6.4. I/O统计信息  
  4.7. 设备配置  
  4.7.1. 模拟器  
  4.7.2. 磁盘  
  4.7.3. 网络  
  4.7.4. 鼠标、键盘和平板电脑  
  4.7.5. USB设备透传  
  4.7.6. PCI设备透传  
  4.8. 活配置更改（Live Configuration Change）  
  4.8.1. 块设备工作（Block Device Jobs）  

5. 存储池  
  5.1. 概述  
  5.2. 列出池内容（Listing pools）  
  5.3. 池的使用  
  5.4. 生命周期控制  
  5.5. 获得池的来源（Discovering pool soutces）  
  5.6. 池配置  
  5.7. 卷的概述  
  5.8. 卷清单（Listing volumes）  
  5.9. 卷信息  
  5.10. 创建和删除卷  
  5.11. 克隆卷  
  5.12. 配置卷  

6. 虚拟网络  
  6.1. 概述  
  6.2. 列出网络（Listing networks）  
  6.3. 生命周期控制  
  6.4. 网络配置 

7. 网络接口  
  7.1. 概述  
  7.2. XML接口描述格式  
  7.3. 检索信息的接口  
  7.3.1. 列举接口  
  7.3.2. 获得一个接口实例（Obtaining a virInterface instance for an Interface）    
  7.3.3. 获取详细的接口信息  
  7.3.4. 获取接口的网络地址  
  7.4. 管理接口配置文件  
  7.4.1. 定义一个接口配置  
  7.4.2. 取消（undefining）一个接口配置  
  7.4.3. changeRollback  
  7.4.4. changeBegin  
  7.4.5. changeCommit  
  7.5. 生命周期管理界面  
  7.5.1. 激活一个接口  
  7.5.2. 停用一个接口  

8. 错误处理  
  8.1. virGetLastError  
  8.2. SublibvirtError  
  8.3. 注册一个错误处理函数  

9. 事件和定时器处理  
  9.1. 事件处理  
  9.2. 定时器处理  

10. 安全模型  

11. 调试/日志记录  
   11.1. 环境变量  

A. 修订历史  

-----

## 原文：

### Preface
- Document Conventions  
  - Typographic Conventions  
  - Pull-quote Conventions  
  - Notes and Warnings 
- We Need Feedback!  


1. Introduction  
  1.1. Overview  
  1.2. Glossary of terms  
2. Architecture  
  2.1. Object model  
  2.1.1. Hypervisor connections  
  2.1.2. Guest domains  
  2.1.3. Virtual networks  
  2.1.4. Storage pools  
  2.1.5. Storage volumes  
  2.1.6. Host devices  
  2.2. Driver model  
  2.3. Remote management  
  2.3.1. Basic usage  
  2.3.2. Data Transports  
  2.3.3. Authentication schemes  
  2.4. Generating TLS certificates  
  2.4.1. Public Key Infrastructure setup  

3. Connections  
  3.1. Overview  
  3.1.1. open  
  3.1.2. openReadOnly  
  3.1.3. openAuth  
  3.1.4. close  
  3.2. URI formats  
  3.2.1. Local URIs  
  3.2.2. Remote URIs  
  3.3. Capability Information Methods  
  3.4. Host information  
  3.4.1. getHostname  
  3.4.2. getMaxVcpus  
  3.4.3. getInfo  
  3.4.4. getCellsFreeMemory  
  3.4.5. getType  
  3.4.6. getVersion   
  3.4.7. getLibVersion  
  3.4.8. getURI      
  3.4.9. isEncrypted  
  3.4.10. isSecure  
  3.4.11. isAlive  
  3.4.12. compareCPU  
  3.4.13. getFreeMemory  
  3.4.14. getFreePages  
  3.4.15. getMemoryParameters  
  3.4.16. getMemoryStats  
  3.4.17. getSecurityModel  
  3.4.18. getSysinfo  
  3.4.19. getCPUMap  
  3.4.20. getCPUStats   
  3.4.21. getCPUModelNames   

4. Guest Domains     
  4.1. Domain Overview  
  4.2. Listing Domains  
  4.3. Obtaining State Information About a Domain  
  4.3.1. Fetching the ID of a domain  
  4.3.2. Fetching the UUID of a domain  
  4.3.3. Fetching the OS type of a domain  
  4.3.4. Determining if the domain has a current snapshot  
  4.3.5. Determining if the domain has managed save images   
  4.3.6. Fetch the hostname of the domain  
  4.3.7. Get the Domain hardware info  
  4.3.8. Determine if the Domain is running  
  4.3.9. Determine if the Domain is persistent  
  4.3.10. Determine if the Domain is updated  
  4.3.11. Determine the max memory of the Domain  
  4.3.12. Determine the max Vcpus of the Domain  
  4.3.13. Fetch the name of the Domain  
  4.3.14. Fetch the state of the Domain  
  4.3.15. Extract the time information from Domain  

4.4. Lifecycle Control  
4.4.1. Provisioning and Starting  
4.4.2. Stopping  
4.4.3. Suspend / Resume and Save / Restore  
4.4.4. Migration  
4.4.5. Autostart  
4.5. Domain Configuration   
4.5.1. Boot Modes   
4.5.2. Memory / CPU Resources   
4.6. Monitoring Performance  
4.6.1. Domain Block Device Performance  
4.6.2. vCPU Performance  
4.6.3. Memory Statistics  
4.6.4. I/O Statistics  
4.7. Device configuration  
4.7.1. Emulator  
4.7.2. Disks  
4.7.3. Networking  
4.7.4. Mice, Keyboard & Tablets  
4.7.5. USB Device Passthrough  
4.7.6. PCI device passthrough  
4.8. Live Configuration Change  
4.8.1. Block Device Jobs  

5. Storage Pools  
  5.1. Overview  
  5.2. Listing pools  
  5.3. Pool usage  
  5.4. Lifecycle control  
  5.5. Discovering pool sources   
  5.6. Pool configuration  
  5.7. Volume overview  
  5.8. Listing volumes  
  5.9. Volume information  
  5.10. Creating and deleting volumes  
  5.11. Cloning volumes   
  5.12. Configuring volumes  

6. Virtual Networks  
  6.1. Overview  
  6.2. Listing networks  
  6.3. Lifecycle control  
  6.4. Network configuration  

7. Network Interfaces  
  7.1. Overview  
  7.2. XML Interface Description Format  
  7.3. Retrieving Information About Interfaces  
  7.3.1. Enumerating Interfaces  
  7.3.2. Obtaining a virInterface instance for an Interface  
  7.3.3. Retrieving Detailed Interface Information  
  7.3.4. Retrieving Interface Network Addresses  
  7.4. Managing interface configuration files  
  7.4.1. Defining an interface configuration  
  7.4.2. Undefining an interface configuration  
  7.4.3. changeRollback  
  7.4.4. changeBegin  
  7.4.5. changeCommit  
  7.5. Interface lifecycle management  
  7.5.1. Activating an interface  
  7.5.2. Deactivating an interface  

8. Error Handling  
  8.1. virGetLastError  
  8.2. Subclassing libvirtError  
  8.3. Registering an Error Handler Function  

9. Event and Timer Handling  
  9.1. Event Handling  
  9.2. Timer Handling  

10. Security Model  

11. Debugging / Logging  
   11.1. Environment Variables  

A. Revision History  

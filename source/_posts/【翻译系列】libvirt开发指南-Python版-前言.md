title: 【翻译系列】libvirt开发指南-Python版-前言
date: 2016-09-20  11:01:27
tags: libvirt;开发指南; 教程; libvirt_Application_Development_Guides

----

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

# 前言

## 1 文档约定
本手册使用几个约定来强调某些词和短语和关注特定的信息。

在PDF和纸质版本，本手册使用的字体 Liberation Fonts字体集。Liberation Fonts还用于HTML版本。如果没有安装，字体显示。 注意:Red Hat Enterprise Linux 5以后默认包括Liberation Fonts。

### 1.1 排版约定
<!--more-->
四个排版约定是用来提醒特定的词汇和短语。 约定如下。

##### Mono-spaced 粗体

用来强调系统输入，包括shell命令、文件名称和路径。 也用来强调键帽和组合键。

##### Proportional 粗体

这表示系统上遇到的单词或短语，包括应用程序名称;对话框文本；标记按钮，复选框和单选按钮标签;菜单标题和子标题。 例如:  
选择 系统 → 首选项 → 鼠标 从主菜单栏 鼠标的偏好 。 在 按钮 选项卡上，单击 左手鼠标 复选框，然后单击 关闭 切换主鼠标按钮从左到右(使鼠标适用于左手)。

##### Mono-spaced粗斜体 或 Proportional粗斜体  

斜体表示可更换或变量的添加文本。 斜体表示文本，不要逐字输入或显示文本，根据情况变化。

### 1.2 Pull-quote约定

终端输出和源代码表示如下：  
输出发送到终端使用 mono-spaced roman字体，如下:  
![0.2-1](/pictures/翻译-libvirt开发指南-python-0.2-1.png)
源代码使用 mono-spaced roman字体，但添加语法高亮显示，如下:  
![0.2-2](/pictures/翻译-libvirt开发指南-python-0.2-2.png)  

### 1.3 注意和警告

最后，我们使用三个视觉风格对重要信息进行提醒，否则可能会被忽视。  
![0.2-3](/pictures/翻译-libvirt开发指南-python-0.2-3.png)  

## 2 我们需要反馈!

如果你发现本手册的印刷错误，或者如果你有想到一个方法，使本手册更好，我们很乐意听到你的反馈！ 请提交错误报告[http://libvirt.org/bugs.html](http://libvirt.org/bugs.html)

---
layout: post
title: Windows10 下安装 Nexus OSS 3.xx
category: 2017年5月
tags: Maven Nexus
keywords: Windows10, Nexus, Maven
description: 主要介绍如何在 Windows10 下安装 Nexus OSS 3.xx
---

## 说明

　　本文主要介绍如何在`Windows10_64bit`环境下安装`Sonatype`的`Nexus OSS 3.0.1-01`。 

　　安装须知：`JDK`必须确保在`Java8`及以上，不支持`OpenJDK`；`Maven`版本建议不低于`3.3.3`。

　　**官方文档上的配置**

![安装须知.jpg](https://ooo.0o0.ooo/2016/09/10/57d37d091c01e.jpg)

---

## 下载

　　从[官网](https://www.sonatype.com/download-oss-sonatype)上下载`Nexus`的最新版本，这里选择`nexus-3.0.1-01-win64.zip`压缩版本。

![下载.jpg](https://ooo.0o0.ooo/2016/09/10/57d37d0907f3e.jpg)

---

## 解压

　　解压缩该`zip`包，建议`Nexus`的存储路径不使用中文（为方便后文描述，规定**%NEXUS_HOME%**作为`Nexus`的存储路径）。

---

## 基本配置

　　主要配置`Nexus`的端口与`Nexus`的上下文路径。打开`%NEXUS_HOME%/etc/org.sonatype.nexus.cfg`文件，可以看到如下的默认配置： 
![默认配置.jpg](https://ooo.0o0.ooo/2016/09/10/57d37eab8ebc6.jpg)
　　其中`application-host`代表`Nexus`服务监听的主机，`application-port`代表`Nexus`服务监听的端口，`nexus-context-path`代表`Nexus`服务的上下文。通常可以不做任何修改，但个人习惯于修改`application-host`为`127.0.0.1`（关于`0.0.0.0`与`127.0.0.1`的区别自行检索），修改`nexus-context-path`为`/nexus`。（这些修改都是个人口味，即使你不做修改也是没问题的）

![端口上下文配置.jpg](https://ooo.0o0.ooo/2016/09/10/57d37d0914dc2.jpg)

---

## 运行环境配置

　　可以在`%NEXUS_HOME%/bin/nexus.vmoptions`中配置运行时的最大堆、最小堆，根据个人的电脑以及需要可以自行修改，默认配置如下： 
![默认运行环境配置.jpg](https://ooo.0o0.ooo/2016/09/10/57d37d0915fe6.jpg)

---

## 运行并安装Nexus的Windows服务

　　在`%NEXUS_HOME%/bin/`目录下，以管理员身份运行`CMD`，键入`nexus.exe /run`命令可以启动`Nexus`（没试过，官网上是这样写的），但是通常不建议这么做，更好的方式是作为一个系统服务来启动。`Windows`下安装`Nexus`服务更是简单：还是在`%NEXUS_HOME%/bin/`目录，直接以管理员身份运行`CMD`并键入`nexus.exe /install`可以安装`Nexus`服务，相应地：`nexus.exe /uninstall`可以卸载服务。 

　　服务安装完毕之后，可以在`控制面板/所有控制面板项/管理工具/计算机管理`或者右键`我的电脑` –> `管理`中找到`服务和应用程序/服务`，接着将我们安装的`Nexus`服务`启动类型`修改为`手动`。

![Nexus服务启动类型.jpg](https://ooo.0o0.ooo/2016/09/11/57d4b51ee8784.jpg)

　　有三种方式启动`Nexus`： 

<span class="blue">第一种： </span>

　　在`%NEXUS_HOME%/bin/`目录下，以管理员身份运行`CMD`，键入`nexus.exe /start`命令启动服务，对应的`nexus.exe /stop`命令关闭当前开启的`Nexus`服务；【可以将`%NEXUS_HOME%/bin/`添加到系统变量`Path`中，这样有什么作用你应该懂得】 

<span class="blue">第二种： </span>

　　在任意目录下，直接以管理员身份运行`CMD`，键入`net start nexus`命令可以启动服务，对应的`net stop nexus`可以关闭服务。~~（配置并使用过`MySQL`或者`MongoDB`等`Windows`服务的人基本都知道这两条命令）~~ 

<span class="blue">第三种： </span>

　　在`控制面板/所有控制面板项/管理工具/计算机管理`或者右键`我的电脑` –> `管理`中找到`服务和应用程序/服务`选项，在右侧的服务列表中找到`nexus服务`，接着可以选择`启动`、`停止`、`重启动`该服务。

　　当然，如果你把`nexus服务`的启动类型设置为`自动`的话，那么则不必再手动启动了。

---

## 访问

　　服务启动之后，在浏览器中键入`localhost:8081/nexus/`（如果没有修改过默认配置的话则访问`localhost:8081/`）即可进入`Nexus`的欢迎页，点击右上角的`Sign in`按钮并输入默认用户`admin`以及默认密码`admin123`便可以管理配置`Nexus`私服了。

![欢迎页.jpg](https://ooo.0o0.ooo/2016/09/10/57d37d091c5fd.jpg)

------

　　本文并没有讲到`Nexus`的第二个基础：

- 如何配置`Nexus`的宿主/代理仓库以及仓库组；

- 如何在项目中使用`Nexus`；

- 如何通过`Maven`配置`Nexus`；

- 如何在`Nexus`中检索依赖。

  　　这部分请自己参考[官方文档](http://books.sonatype.com/nexus-book/3.0/reference/index.html)或者等以后我再写一篇介绍。

|                 |
| --------------- |
| 编写日期：2016-09-10 |
| 发布日期：2017-05-23 |
---
title: NSA逆向工具--Ghidra安装及漏洞修复
date: 2019-03-07 10:35:51
tags: 逆向
categories: 工具使用
---

# 0x01 简介
2019年RSA安全大会上，美国国家安全局（NSA）公布了一款名叫Ghidra的逆向工程工具。Ghidra使用Java编写，具有图形界面，可在Windows，Mac和Linux上运行。

**Ghidra官网地址：**[https://ghidra-sre.org/](https://ghidra-sre.org/)
注：屏蔽了部分国家IP地址，方法大家都懂。。。
<!--more-->

# 0x02 安装环境

## 平台支持
* Microsoft Windows 7 or 10 (64位)
* Linux （64位，推荐使用CentOS 7）
* macOS (OS X) 10.8.3+（Mountain Lion或更高版本）
**注意：** 不推荐使用32位操作系统
（本人使用64位的CentOS 7）

## 最低要求

### 硬件
* 4 GB RAM
* 1 GB storage

### 软件
* JDK 11 或更高版本
  （推荐使用OpenJDK，地址：[jdk.java.net](http://jdk.java.net/)，本人使用JDK 11.0.2版本。）

# 0x03 安装步骤

解压jdk到本地
```sh
tar -zxvf openjdk-11.0.2_linux-x64_bin.tar.gz
```
![](2019-03-07-NSA逆向工具-Ghidra安装及漏洞修复\OpenJDK.png)

将JDK bin目录添加到PATH环境变量。当然我们可以指定Ghidra使用指定的Java启动（在support / launch.properties文件中设置JAVA_HOME_OVERRIDE属性），当Java版本不兼容时，Ghidra会自动查找兼容版本。

解压Ghidra到本地，进入文件夹，运行ghidraRun，即可打开Ghidra

![](2019-03-07-NSA逆向工具-Ghidra安装及漏洞修复\Ghidra.png)

# 0x04 bugdoor修复

## 漏洞说明

工具发布后，Matthew Hickey报告了一个安全问题，在调试模式下运行套件时，会向网络打开18001端口，接受任何可以连接的机器的远程命令。

![](2019-03-07-NSA逆向工具-Ghidra安装及漏洞修复\漏洞.PNG)

## 修复方法

打开support目录下的launch.sh文件，找到150行代码，将*修改为127.0.0.1，仅监听本机的调试连接，而不是通过网络监听任何主机。

![](2019-03-07-NSA逆向工具-Ghidra安装及漏洞修复\修复.png)

# 0x05 参考
[Ghidra Installation Guide](https://ghidra-sre.org/InstallationGuide.html)

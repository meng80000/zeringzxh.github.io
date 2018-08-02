---
title: Cuckoo-Sandbox学习-安装篇
date: 2018-06-06 14:11:45
tags:
  - Cickoo Sandbox
  - 工具安装
categories: Cuckoo Sandbox
---

Cuckoo Sandbox是一个开源的恶意文件自动化分析系统，通过Cuckoo Sandbox可以方便地帮助安全研究人员验证恶意程序的特征信息。

## 安装环境
本人在vmware中建立两个虚拟机，一个host，一个guest。
VMware 12 Pro
ubuntu-16.04.1-desktop-amd64
window7-64bit
<!--more-->
## 安装准备
使用vmware安装ubuntu-16.04.1-desktop-amd64，添加2个网卡，一个NAT模式，一个内网网卡。
![](2018-06-06-Cuckoo-Sandbox学习-安装篇\网卡-ubuntu.PNG)

依赖库
```bash
$ sudo apt-get install python python-pip python-dev libffi-dev libssl-dev
$ sudo apt-get install python-virtualenv python-setuptools
$ sudo apt-get install libjpeg-dev zlib1g-dev swig
```
为了使用基于Django的Web界面,使用PostgreSQL作为数据库，需要安装MongoDB和PostgreSQL。
```bash
$ sudo apt-get install mongodb
$ sudo apt-get install postgresql libpq-dev
```
建议首先升级pip和setuptools库，因为它们经常过时，导致试图安装Cuckoo时出现问题
```bash
$ sudo pip install -U pip setuptools
```
---
若使用virtual box则创建一个cuckoo用户，将其添加到vboxusers组里<font color="red">（VirtualBox安装后创建）</font>。确保运行Cuckoo的用户与用于创建和运行虚拟机的用户相同（至少在VirtualBox的情况下），否则Cuckoo将无法识别和启动这些虚拟机。
```bash
$ sudo adduser cuckoo
$ sudo usermod -a -G vboxusers cuckoo
```
安装VirtualBox
```bash
$ sudo apt-get install virtualbox
```
---

## 安装Cuckoo
使用一条命令安装
```bash
$ sudo pip install -U cuckoo
```
部分依赖需要翻墙，安装成功如下：
![](2018-06-06-Cuckoo-Sandbox学习-安装篇\install.PNG)

如果在本地使用virtual box则切换cuckoo用户并运行，
![](2018-06-06-Cuckoo-Sandbox学习-安装篇\run.PNG)

从中可以看出配置文件路径为*/home/cuckoo/.cuckoo/conf*，工作目录路径可以修改，.cuckoo为隐藏目录，使用`ctrl+h`查看隐藏文件。
## 配置文件
几个主要的配置文件：
* cuckoo.conf：用于配置常规行为和分析选项。
* auxiliary.conf：用于启用和配置辅助模块。
* \<machinery>.conf：用于定义虚拟化软件的选项（该文件与您在cuckoo.conf中选择的机器模块名称相同,virtualbox.conf和vmware.conf）。
* memory.conf：波动配置。
* processing.conf：用于启用和配置处理模块。
* reporting.conf：用于启用或禁用报告格式。

## 客户机安装及配置
使用vmware安装windows7 64bit，配置一个内网网卡。
查看ubuntu16.04.1的ip地址，配置window7客户机的ipv4地址的默认网关和DNS地址为ubuntu的ip地址，如下：
![](2018-06-06-Cuckoo-Sandbox学习-安装篇\ip-windows7.PNG)

安装python2.7.10，并关闭windows防火墙和自动更新，原因为他们可能影响恶意软件的行为。
![](2018-06-06-Cuckoo-Sandbox学习-安装篇\win7配置.PNG)

为了正常运行，需要配置如下一些选项。
* 启用自动登录
* 启用远程RPC
* 关闭分页（可选）
* 禁用屏幕保护程序（可选）

在Windows 7中，以管理员权限打开管理命令提示符，输入以下命令以启用自动登录和远程RPC。
```cmd
reg add "hklm\software\Microsoft\Windows NT\CurrentVersion\WinLogon" /v DefaultUserName /d <USERNAME> /t REG_SZ /f
reg add "hklm\software\Microsoft\Windows NT\CurrentVersion\WinLogon" /v DefaultPassword /d <PASSWORD> /t REG_SZ /f
reg add "hklm\software\Microsoft\Windows NT\CurrentVersion\WinLogon" /v AutoAdminLogon /d 1 /t REG_SZ /f
reg add "hklm\system\CurrentControlSet\Control\TerminalServer" /v AllowRemoteRPC /d 0x01 /t REG_DWORD /f
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v LocalAccountTokenFilterPolicy /d 0x01 /t REG_DWORD /f
```
前两条命令里的 \<USERNAME> 和 \<PASSWORD> 为自己的账户和密码。

找到之前安装的cuckoo路径下的agent/agent.py,并将其放到win7的启动目录下，`C:\Users\用户名\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`，把agent.py后缀改为agent.pyw，这样程序启动就不会有GUI窗口。

## 网络配置
很多恶意软件都需要访问网络才可以正常运行，因此需要配置windows主机上网，使用host主机开启流量转发、用 iptables 做地址转换（NAT）规则。

临时开启ip转发功能方式，系统重启之后便失效：
* sudo sysctl -w net.ipv4.ip_forward=1
* sudo echo 1 > /proc/sys/net/ipv4/ip_forward

若要重启后仍然有效，则需修改配置文件/etc/sysctl.conf，去掉net.ipv4.ip_forward=1 前的注释，之后执行如下命令：
```bash
sudo sysctl -p /etc/sysctl.conf
```
然后配置Iptables的规则:
```
sudo iptables -A FORWARD -o eth0 -i vboxnet0 -s 192.168.56.0/24 -m conntrack --ctstate NEW -j ACCEPT
sudo iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A POSTROUTING -t nat -j MASQUERADE

sudo vim /etc/network/interfaces
编辑 Ubuntu 的网络配置文件 /etc/network/interfaces ，在文件末尾添加以下两行：
pre-up iptables-restore < /etc/iptables.rules #开机时启用 iptables 规则
post-down iptables-save > /etc/iptables.rules #关机前保存当前所有的 iptables 规则
```
效果如下：
![](2018-06-06-Cuckoo-Sandbox学习-安装篇\ping.PNG)

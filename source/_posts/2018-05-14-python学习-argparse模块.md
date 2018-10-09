---
title: python学习-argparse模块
date: 2018-05-14 15:28:37
tags:
  - argparse
categories: python
---

  argparse模块是python中用于解析命令行参数和选项的标准模块，方便用户在命令行中输入参数。

## argparse使用
<!--more-->
### 示例
先看一个简单的例子
```python
import argparse
parser = argparse.ArgumentParser()
parser.parse_args()
```
* 创建 ArgumentParser() 对象
* 使用 parse_args() 解析

将上述代码保存并命名为test.py,运行：
```bash
$ python test.py
$ python test.py -h
usage: t.py [-h]

optional arguments:
  -h, --help  show this help message and exit
```
* 不添加任何参数，无回显
* 添加-h参数，获取帮助，-h为默认预设参数

### ArgumentParser函数解析
  使用help(argparse.ArgumentParser())查看函数帮助，如下：
```bash
class ArgumentParser(_AttributeHolder, _ActionsContainer)
 |  Object for parsing command line strings into Python objects.
 |
 |  Keyword Arguments:
 |      - prog -- The name of the program (default: sys.argv[0])
 |      - usage -- A usage message (default: auto-generated from arguments)
 |      - description -- A description of what the program does
 |      - epilog -- Text following the argument descriptions
 |      - parents -- Parsers whose arguments should be copied into this one
 |      - formatter_class -- HelpFormatter class for printing help messages
 |      - prefix_chars -- Characters that prefix optional arguments
 |      - fromfile_prefix_chars -- Characters that prefix files containing
 |          additional arguments
 |      - argument_default -- The default value for all arguments
 |      - conflict_handler -- String indicating how to handle conflicts
 |      - add_help -- Add a -h/-help option
 |
......
 |
 |  Methods defined here:
 |
 |  __init__(self, prog=None, usage=None, description=None, epilog=None, version=None, parents=[], formatter_class=<class 'argparse.HelpFormatter'>, prefix_chars='-', fromfile_prefix_chars=None, argument_default=None, conflict_handler='error', add_help=True)
 ......
```
常用参数解析：
 * prog：程序名（默认为sys.argv[0]）
 * usage：程序的使用用例，默认情况下会自动生成
 * description：简短的描述这个程序的用途，help参数之前显示的信息。
 * parents：ArgumentParser对象组成列表，这些对象中的参数也要包含进来。
 * formatter_class：一个自定义帮助信息格式化输出的类。
 * prefix_chars：可选参数之前的前缀（默认为-)。
 * fromfile_prefix_chars：如果是从文件中读取参数，这个文件名参数的前缀（默认为None）。
 * conflict_handler：通常不需要，定义了处理冲突选项的策略。

### add_argument() 方法
使用add_argument()方法，用来定义程序可接受的命令行参数。
```
ArgumentParser.add_argument(name or flags...[, action][, nargs][, const][, default][, type][, choices][, required][, help][, metavar][, dest])
```
* name or flags - 选项字符串的名字或者列表，例如 foo 或者 -f, --foo。
* action - 命令行中该参数存在时的操作，默认值是 store
  * store_ture，表示赋值为true；
  * append，将遇到的值存储成列表，也就是如果参数重复则会保存多个值;
  * append_const，将参数规范中定义的一个值保存到一个列表；
  * count，存储遇到的次数；
* default - 不指定参数时的默认值；
* nargs - 应该读取的命令行参数个数，可以是具体的数字，或者是?号，当不指定值时对于 Positional argument 使用 default，对于 Optional argument 使用 const；或者是 * 号，表示 0 或多个参数；或者是 + 号表示 1 或多个参数。
* required - 可选参数是否可以省略 (仅针对可选参数),默认为False；
* const - action 和 nargs 所需要的常量值。
* metavar - 对于必选参数默认就是参数名称，对于可选参数默认是必须存在参数后；
* dest - 解析后的参数名称，默认情况下，对于可选参数选取最长的名称，中划线转换为下划线。

### 添加位置参数
先看如下例子：
```python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument("square", help="display a square of a given number",
                    type=int)
args = parser.parse_args()
print args
print args.square**2
```
运行结果：
```bash
$ python test.py
usage: test.py [-h] square
t.py: error: too few arguments

$ python test.py -h
usage: test.py [-h] square

positional arguments:
  square      display a square of a given number
optional arguments:
  -h, --help  show this help message and exit
$ python test.py 3
Namespace(square=3)
9
$ python test.py three
usage: test.py [-h] square
t.py: error: argument square: invalid int value: 'three'
```
* 运行程序，就必须设置一个参数。
* parse_args()方法获取Namespace。

### 可选参数
通过两种方式指定
1. 通过<span style="background:#BDD3F7"> - </span>来指定短参数，如：<span style="background:#BDD3F7">-h</span>
2. 通过<span style="background:#BDD3F7"> -- </span>来指定长参数，如：<span style="background:#BDD3F7">--help</span>
修改test.py文件：
```python
import argparse

parser = argparse.ArgumentParser(formatter_class=argparse.RawTextHelpFormatter, prog='python test.py <OPTIONS>',description='test')
parser.add_argument('-a', '--all', help="Perform all operations",metavar='xx')
parser.add_argument('-d', '--dns', help="Perform",required=False, action='store_true')
par=parser.parse_args()
print vars(par)
print vars(par)['all']
for v in vars(par):
    print v
    print vars(par)[v]
```
运行结果：
```bash
$ python test.py
{'all': None, 'dns': False}
None
all
None
dns
False
$ python test.py -h
usage: python Stealth.py <OPTIONS> [-h] [-a xx] [-d]

collect

optional arguments:
  -h, --help       show this help message and exit
  -a xx, --all xx  Perform all operations
  -d, --dns        Perform

$ python test.py -a
usage: python Stealth.py <OPTIONS> [-h] [-a xx] [-d]
python Stealth.py <OPTIONS>: error: argument -a/--all: expected one argument
$ python test.py -a test1 -d
{'all': 'test1', 'dns': True}
test1
all
test1
dns
True
```
* 存在metavar参数时，可选参数后必须接值
* 存在action='store_true'，默认值为False
* metavar和action='store_true'不能共存

### 混合参数使用
计算一个整数数列的最大值或求和：
```python
import argparse

parser = argparse.ArgumentParser(description='Process some integers.')
parser.add_argument('integers', metavar='N', type=int, nargs='+',
                   help='an integer for the accumulator')
parser.add_argument('--sum', dest='accumulate', action='store_const',
                   const=sum, default=max,
                   help='sum the integers (default: find the max)')

args = parser.parse_args()
print(args.accumulate(args.integers))
```
运行结果：
```bash
$ python test.py 1 2 3 4
4
$ python test.py 1 2 3 4 --sum
10
```
上述示例使用accumulate来重命名sum参数，与下面代码实现功能一样：
```python
import argparse

parser = argparse.ArgumentParser(description='Process some integers.')
parser.add_argument('integers', metavar='N', type=int, nargs='+',
                   help='an integer for the accumulator')
parser.add_argument('--sum', action='store_const',
                   const=sum, default=max,
                   help='sum the integers (default: find the max)')

args = parser.parse_args()
print(args.sum(args.integers))
```

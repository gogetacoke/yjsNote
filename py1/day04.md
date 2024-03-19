- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [模块基础](#模块基础)
    - [导入模块](#导入模块)
    - [导入自定义模块](#导入自定义模块)
    - [导入模块的方法](#导入模块的方法)
    - [模块加载](#模块加载)
    - [模块特性](#模块特性)
    - [练习1：生成随机密码](#练习1生成随机密码)
  - [时间模块](#时间模块)
    - [时间表示方式](#时间表示方式)
    - [结构化时间](#结构化时间)
    - [格式化时间字符次](#格式化时间字符次)
    - [主要方法](#主要方法)
      - [time.sleep(t)](#timesleept)
      - [time.time()](#timetime)
      - [time.gmtime()](#timegmtime)
      - [time.localtime()](#timelocaltime)
      - [time.mktime(t)](#timemktimet)
      - [time.strftime()](#timestrftime)
      - [time.strptime()](#timestrptime)
    - [时间格式之间的转换](#时间格式之间的转换)
    - [练习2：打印日志文件](#练习2打印日志文件)
  - [Pyhon语法风格](#pyhon语法风格)
    - [变量赋值](#变量赋值)
    - [合法标识符](#合法标识符)
    - [获取关键字](#获取关键字)
    - [内建函数](#内建函数)
  - [字符串](#字符串)
    - [格式化详解](#格式化详解)
    - [字符串常用函数](#字符串常用函数)
      - [startswith](#startswith)
      - [endswith](#endswith)
      - [islower](#islower)
      - [upper](#upper)
      - [strip](#strip)
      - [split](#split)
      - [join](#join)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)


# 学习目标

模块基础

时间模块

语法风格及布局

字符串详解

# 课堂笔记（命令）

# 课堂笔记（文本）

## 模块基础

> - 模块是从逻辑上组织python代码的形式
> - 当代码量变得相当大的时候，最好把代码分成一些有组织的代码段，前提是保证它们的 **彼此交互**
> - 这些代码片段相互间有一定的联系，可能是一个包含数据成员和方法的类，也可能是一组相关但彼此独立的操作函数
> - 人话：一个   **.py文件**   就是一个python模块

### 导入模块

> - 使用 **import** 导入模块
> - 模块属性通过 **“模块名.属性”** 的方式调用
> - 模块函数通过 **“模块名.函数名”** 的方式调用
> - 如果仅需要模块中的某些属性，也可以单独导入

### 导入自定义模块

> 为什么需要导入模块？
>
> + 可以提升开发效率，简化代码

```python
# 导入上篇写的copy方法
from day03 import copyFunc

src_file = "/etc/passwd"
dst_file = "/tmp/passwd"
# 从day03模块中导入copyFunc模块，再调用模块中的copy方法
# copyFunc.copy(src_file, dst_file)
```

### 导入模块的方法

> - 一行指导入一个模块，可以导入多行， 例如：**import random**
> - 也可一行导入多个模块，例如：**import random, sys**
> - 只导入模块中的某些方法，例如：**from random import choice, randint**

### 模块加载

> - 一个模块只被 **加载一次**，无论它被导入多少次
> - 只加载一次可以 **阻止多重导入时，代码被多次执行**
> - 如果两个文件相互导入，防止了无限的相互加载
> - 模块加载时，顶层代码会自动执行，所以只将函数放入模块的顶层是最好的编程习惯

### 模块特性

> 模块在被导入时，会先完整的执行一次模块中的所有程序

```python
# 在copyFnuc.py文件中添加一个print(__name__)，再在当前文件进行调用

# copFunc.py
print(__name__)  # __main__

# test.py
import day03.copyFuc # day03.copyFunc

# 所以在后续使用python模块执行代码的标准格式
def test():
    pass
# 程序入口;表示在当前模块下执行
if __name__ == "__main__"
	test()
```

### 练习1：生成随机密码

```python
"""
练习：
编写一个能生成 8 位随机密码的程序
使用 random 的 choice 函数随机取出字符(大小写字母数字)
改进程序，用户可以自己决定生成多少位的密码
"""

import random

passwd_str = "abcdefghijklmnopkrstuvwxyz123456789ABCDEFGHIJKLMNOPKRSTUVWXYZ"
passwd = '' # 定义存储密码的变量
for i in range(8):
    passwd += random.choice(passwd_str) # 循环八次随机从passwd_str获取一个值进行相加
print(passwd) # 循环结束后输出密码

"""
优化一：函数化程序，并可以指定密码长度,新建一个test2.py文件测试
"""
import random

def gen_passwd(pwd_length=8):
    passwd_str = "abcdefghijklmnopkrstuvwxyz123456789ABCDEFGHIJKLMNOPKRSTUVWXYZ"
    passwds = ''
    for i in range(pwd_length):
        passwds += random.choice(passwd_str)
    return passwds
if __name__ == '__main__':
	print(gen_passwd()) # lvKFAgdK
    print(gen_passwd(9)) # w3XoxPAJy
"""
优化二：随机密码的字符选择可以调用模块，在test3.py文件中测试
"""
import random
import string

def gen_passwds(pwd_length=8):
    passwd_str = string.ascii_letters + string.digits
    passwds = ''
    for i in range(pwd_length):
        passwds += random.choice(passwd_str)
    return passwds
if __name__ == '__main__':
    print(gen_passwds())
    print(gen_passwds(9))
```

## 时间模块

> time模块

### 时间表示方式

> - **时间戳 timestamp：**表示的是从 1970 年1月1日 00:00:00 开始按秒计算的偏移量
> - **UTC（Coordinated Universal Time, 世界协调时）**亦即格林威治天文时间，世界标准时间。在中国为 UTC+8  DST(Daylight Saving Time) 即夏令时；
> - **结构化时间（struct_time）**： 由 9 个元素组成

### 结构化时间

> 使用 `time.localtime()` 等方法可以获得一个结构化时间

```python
"""
调用时间模块的localtime方法获得机构化时间
"""
import time

print(time.localtime())
# time.struct_time(tm_year=2024, tm_mon=3, tm_mday=13, tm_hour=10, tm_min=35, tm_sec=46, tm_wday=2, tm_yday=73, tm_isdst=0)
```

结构化时间共有9个元素，按顺序排列如下表：

| 索引 | 属性                      | 取值范围                      |
| ---- | ------------------------- | ----------------------------- |
| 0    | tm_year（年）             | 比如 2021                     |
| 1    | tm_mon（月）              | 1 - 12                        |
| 2    | tm_mday（日）             | 1 - 31                        |
| 3    | tm_hour（时）             | 0 - 23                        |
| 4    | tm_min（分）              | 0 - 59                        |
| 5    | tm_sec（秒）              | 0 - 59                        |
| 6    | tm_wday（weekday）        | 0 - 6（0表示周一，6表示周日） |
| 7    | tm_yday（一年中的第几天） | 1 - 366                       |
| 8    | tm_isdst（是否是夏令时）  | 默认为-1                      |

```python
"""
然结构化时间是一个序列，那么就可以通过索引进行取值，也可以进行分片，或者通过属性名获取对应的值。
"""
import time

date1 = time.localtime()
print(date1[3]) # 10
print(date1[0:5]) # (2024, 3, 13, 10, 38)
```

> [注]：time类型时不可变类型，所有时间值都只读不能改

### 格式化时间字符次

```python
"""
利用 time.strftime('%Y-%m-%d %H:%M:%S') 等方法可以获得一个格式化时间字符串
"""
date2 = time.strftime("%Y-%m-%d %H-%M-%S")
print(date2) # 2024-03-13 10-42-36
```

> 对于格式化控制字符串 `"%Y-%m-%d %H:%M:%S`，其中每一个字母所代表的意思如下表所示，注意大小写的区别：

| 格式 | 含义                                    | 格式 | 含义                                              |
| ---- | --------------------------------------- | ---- | ------------------------------------------------- |
| %a   | 本地简化星期名称                        | %m   | 月份（01 - 12）                                   |
| %A   | 本地完整星期名称                        | %M   | 分钟数（00 - 59）                                 |
| %b   | 本地简化月份名称                        | %p   | 本地am或者pm的相应符                              |
| %B   | 本地完整月份名称                        | %S   | 秒（01 - 59）                                     |
| %c   | 本地相应的日期和时间                    | %U   | 一年中的星期数（00 – 53，星期日是一个星期的开始） |
| %d   | 一个月中的第几天（01 - 31）             | %w   | 一个星期中的第几天（0 - 6，0是星期天）            |
| %H   | 一天中的第几个小时（24小时制，00 - 23） | %x   | 本地相应日期                                      |
| %I   | 第几个小时（12小时制，01 - 12）         | %X   | 本地相应时间                                      |
| %j   | 一年中的第几天（001 - 366）             | %y   | 去掉世纪的年份（00 - 99）                         |
| %Z   | 时区的名字                              | %Y   | 完整的年份                                        |

### 主要方法

#### time.sleep(t)

> time 模块最常用的方法之一，用来睡眠或者暂停程序t秒，t可以是浮点数或整数。

```python
"""
睡眠方法
tim.sleep(t)
"""
def make_money():
    print("板砖中。。。")
    time.sleep(2)
    print("到帐1元。。")
make_money()

板砖中。。。 # 两秒后再执行下一句
到帐1元。。
```

#### time.time()

> 返回当前系统时间戳。时间戳可以做算术运算。

```python
"""
time.time()
返回当前系统时间戳。时间戳可以做算术运算。
返回程序执行所需时间
"""
def make_moneyy():
    print("板砖中。。。")
    time.sleep(4)
    print("到帐1元。。")
start_time = time.time()
make_moneyy()
end_time = time.time()
print("程序所用时间为：%.2f" %(end_time-start_time))

板砖中。。。
到帐1元。。
程序所用时间为：4.00
```

#### time.gmtime()

```python
"""
time.gmtime([secs])
将时间戳转换为 UTC时区的结构化时间。
可选参数secs的默认值为 time.time()。
1705114957 ：网上生成的一个时间戳
"""
date3 = time.gmtime(1705114957)

print(date3[1])
print(date3[0], date3[1], date3[2])
print(date3[:3])
print(date3.tm_hour)
print(date3.tm_min)

1
2024 1 13
(2024, 1, 13)
3
2
```

#### time.localtime()

```python
"""
time.localtime([secs])
将一个时间戳转换为 当前时区(中国东八区，与UTC相差8个时区)的结构化时间。
如果secs参数未提供，则以当前时间为准，即time.time()。
"""
date4 = time.localtime(1705114957)

print(date4[1])
print(date4[0], date4[1], date4[2])
print(date4[:3])
print(date4.tm_hour)
print(date4.tm_min)

1
2024 1 13
(2024, 1, 13)
11
2
```

#### time.mktime(t)

```python
"""
time.mktime(t)
结构化时间转换为时间戳，t(结构化时间)
"""
date5 = time.mktime(time.localtime())
print(date5)

1710299587.0
```

#### time.strftime()

```python
"""
time.strftime(format [, t])

返回格式化字符串表示的当地时间。
把一个struct_time（如time.localtime()和time.gmtime()的返回值）转化为格式化的时间字符串，显示的格式由参数format决定。
如果未指定t，默认传入time.localtime()。
"""

# date6 = time.strftime("%Y-%m-%d  %H-%M-%S")
date6 = time.strftime("%Y年%m月%d日  %H:%M:%S")
print(date6)

2024-03-13  11-20-39
2024年03月13日  15:33:26
```

#### time.strptime()

```python
"""
time.strptime(string[,format])
将格式化时间字符串转化成结构化时间
该方法是 time.strftime()方法的逆操作。
time.strptime() 方法根据指定的格式把一个时间字符串解析为结构化时间。
提供的字符串要和 format参数 的格式一一对应
如果string中日期间使用 “-” 分隔，format中也必须使用“-”分隔
时间中使用冒号 “:” 分隔，后面也必须使用冒号分隔
并且值也要在合法的区间范围内
"""
date7 = time.strptime("2024-03-02", "%Y-%m-%d")
print(date7)

time.struct_time(tm_year=2024, tm_mon=3, tm_mday=2, tm_hour=0, tm_min=0, tm_sec=0, tm_wday=5, tm_yday=62, tm_isdst=-1)
```

### 时间格式之间的转换

> Python的三种类型时间格式，可以互相进行转换

| 从             | 到             | 方法        |
| -------------- | -------------- | ----------- |
| 时间戳         | UTC结构化时间  | gmtime()    |
| 时间戳         | 本地结构化时间 | localtime() |
| 本地结构化时间 | 时间戳         | mktime()    |
| 结构化时间     | 格式化字符串   | strftime()  |
| 格式化字符串   | 结构化时间     | strptime()  |

### 练习2：打印日志文件

> 编写日志文件：web.log放在当前目录
>
> 2030-01-02 08:01:43 aaaaaaaaaaaaaaaaa
> 2030-01-02 08:34:23 bbbbbbbbbbbbbbbbbbbb
> 2030-01-02 09:23:12 ccccccccccccccccccccc
> 2030-01-02 10:56:13 ddddddddddddddddddddddddddd
> 2030-01-02 11:38:19 eeeeeeeeeeeeeeee
> 2030-01-02 12:02:28 fffffffffffffff

```python
"""
练习：
有一日志文件，按时间先后顺序记录日志
给定时间范围，取出该范围内的日志
"""
import time
# 方案一：取出指定时间段的日志 9-12点
start_ = time.strptime("2030-01-02 09:00:00", "%Y-%m-%d %H:%M:%S")
end_ = time.strptime("2030-01-02 12:00:00", "%Y-%m-%d %H:%M:%S")
with open("web.log", mode="r") as f:
    for item in f.readlines():
        t = time.strptime(item[:19], "%Y-%m-%d %H:%M:%S")
        if start_ < t < end_:
            print(item, end=' ')

# 方案二 日志文件中，时间点是从前往后不断增加的，所以只要遍历到有一行时间超过预计，则后面的所有行均是不满足条件的     
with open("web.log", mode="r") as f:
    for item in f.readlines():
        t = time.strptime(item[:19], "%Y-%m-%d %H:%M:%S")
        if t > end_: # 当时间超过12点就结束
            break
        print(item, end=' ')
```

## Pyhon语法风格

### 变量赋值

```python
"""
python支持链式多重赋值
"""
x = y = 10
print(x) # 10
print(y) # 10

"""
列表赋值
"""
blist = alist = [1, 2]
print(blist) #  [1, 2]
print(alist) #  [1, 2]
# 给列表重新赋值
blist[-1] = 5 
print(blist)#  [1, 5]

"""
字符串赋值
只能一一对值，字符串的值不能超过给定的变量
"""
str1, str2 = "ab"
print(str1) # a
print(str2) # b

"""
变量值互换
"""
a, b = 100, 200
print(a) # 100
a, b = b, a
print(a) # 200
```

### 合法标识符

> - Python 标识符，字符串规则和其他大部分用 C 编写的高级语言相似
> - 第一个字符必须是 **字母或下划线 _**
> - 剩下的字符可以是字母和数字或下划线
> - **大小写敏感**

### 获取关键字

> 定义变量时，不能定义与系统关键字相同的关键字

```python
import keyword
print(keyword.kwlist) # 打印系统下所有的关键字
print("pass" in keyword.kwlist) # 判断字符串是否是关键字
```

### 内建函数

> - Python 为什么可以直接使用一些内建函数，而不用显式的导入它们？
> - 比如 **str()、int()、id()、type()，len()** 等，许多许多非常好用，快捷方便的函数。
> - 这些函数都是一个叫做 `builtins` 模块中定义的函数，而 `builtins` 模块默认 **在Python环境启动的时候就自动导入**，所以可以直接使用这些函数

## 字符串

### 格式化详解

> - 百分号：可以使用格式化符号来表示特定含义

| **格式化字符** | **转换方式**                  |
| -------------- | ----------------------------- |
| %s             | 优先用str()函数进行字符串转换 |

```python
"""
字符串的使用
"""
name, age, hobby = "Bob", 18, ["唱", "跳", "Rap"]
# 字符串拼接,添加列表时 需要转为字符串才能进行拼接
str1 = "姓名：" + name + "年龄：" + "爱好：" + str(hobby)
print(str1)

# %s的使用
str2 = "姓名：%s 年龄：%s  爱好：%s" % (name, age, hobby)
print(str2)

# f的使用
str3 = f"姓名：{name} 年龄：{age}  爱好：{hobby}"
print(str3)
```

> - 可以传入任意类型的数据，如 **整数、浮点数、列表、元组甚至字典**，都会自动转成字符串类型

### 字符串常用函数

#### startswith

```python
# str1.startswith(s1),判断字符串str4是否是以s1开头

str4 = "hello world"
print(str4.startswith("h")) # True
print(str4.startswith("e")) # False
```

#### endswith

```python
# str1.endswith(s1),判断字符串str4是否是以s1结尾
"""
string.endswith(obj,beg=0,end=len(string))检查字符串是否以obj结束，如果beg或在end指定则检查指定的范围内是否以obj结束，如果是，返回Ttue，否则返回False
"""
print(str4.endswith("d")) # True
print(str4.endswith("l"))  # False
```

#### islower

```python
# islower()判断字符串是否全是小写，isrupper()判断字符串是否全是大写
str5 = "abcDe"
str6 = "ABCEDF"
print(str5.islower()) # False 既包含大写也包含小写
print(str5.isupper()) # False
print(str6.isupper()) # True
```

#### upper

```python
# upper()将字符串中的所有小写字母转为大写,islower()相反
str6 = "abcdEfgH"
print(str6.upper()) # ABCDEFGH
print(str6.lower()) # abcdefgh
```

#### strip

```python
# strip() 去除字符串两边的空白符号
str7 = " hello,world"
str8 = " hello,world "
print(str7.strip()) # hello,world
print(str8.strip()) # hello,world
# lstrip()与rstrip()，一个是去除左边空白一个是去除右边空白
```

#### split

```python
# split(s1) 将字符串按照s1进行分割,默认以空格进行分割，分割后将存入在列表中
"""
string.split(str="",num=string.count(str))以str为分割符切片string，如果num有指定值，则仅分割num个字符串
"""
str9 = "a,b,c,de,fg"
str99 = "a b c d"
print(str9.split(',')) # ['a', 'b', 'c', 'de', 'fg']
print(str9.split()) # ['a', 'b', 'c', 'd']
```

#### join

```python
# '='join(s1),将可迭代对象s1按照=进行拼接在一起
str10 = ['a', 'b', 'c', 'd']
str11 = "abcdef"
print('_'.join(str10)) # a_b_c_d
print('_'.join(str11)) # a_b_c_d_e_f
```





# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习
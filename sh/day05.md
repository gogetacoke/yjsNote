- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [awk](#awk)
    - [选项](#选项)
      - [-F](#-f)
    - [指令](#指令)
      - [print](#print)
    - [条件](#条件)
      - [/字符串/](#字符串)
    - [内置变量](#内置变量)
    - [awk处理时机](#awk处理时机)
    - [处理条件](#处理条件)
      - [设置条件](#设置条件)
      - [使用数值](#使用数值)
      - [逻辑测试](#逻辑测试)
      - [数学运算](#数学运算)
    - [awk数组](#awk数组)
      - [awk for循环](#awk-for循环)
      - [awk属组排序](#awk属组排序)
    - [案例](#案例)
- [快捷键](#快捷键)
- [问题](#问题)
    - [1 简述awk工具的基本语法格式。](#1-简述awk工具的基本语法格式)
    - [2 简述awk工具常用的内置变量、各自的作用。](#2-简述awk工具常用的内置变量各自的作用)
    - [3 awk处理文本时，读文件前、读取文件内容中、读文件后后这三个环节是如何表示的？](#3-awk处理文本时读文件前读取文件内容中读文件后后这三个环节是如何表示的)
    - [4 提取当前eth0网卡的IPv4地址及掩码信息。](#4-提取当前eth0网卡的ipv4地址及掩码信息)
    - [5 找出UID位于10~20之间的用户，输出用户名及对应的UID。](#5-找出uid位于1020之间的用户输出用户名及对应的uid)
    - [6 利用awk工具统计使用bash作为解释器的用户数量。](#6-利用awk工具统计使用bash作为解释器的用户数量)
    - [7 在awk中是否可以使用数组，分别以什么构成？](#7-在awk中是否可以使用数组分别以什么构成)
    - [8 在linux中对文本的排序如何实现？](#8-在linux中对文本的排序如何实现)
    - [9 通过安全日志/var/log/secure，分析是谁登录账户tom且密码输入错误，如果错误次数达到3次以上的话，就输出登陆者的IP](#9-通过安全日志varlogsecure分析是谁登录账户tom且密码输入错误如果错误次数达到3次以上的话就输出登陆者的ip)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)




# 学习目标

awk的使用

监控脚本

# 课堂笔记（命令）

# 课堂笔记（文本）

## awk

**准备素材**

```
]#vim abc
hello the world
welcom to chengdu
]#head -5 /etc/passwd > user
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/root/lpd:/sbin/nologin
```

### 选项

#### -F

定义分隔符,默认以空格分割

```
]#awk '{print $1}' abc # 输出abc中的第一列
hello
welcom

]#awk -F: '/root/{print $1}' user # 输出包含root行中的第一列，按照:分割
```

### 指令

#### print

输出内容

```
]#awk '{print}' user # 输出user中所有内容
```

### 条件

#### /字符串/

```
]#awk /to/{print} abc #输出所有包括to的行
```

### 内置变量

```
# 语法
$1 第一列 
$2 第二列
....
$1,$3 第1列和第3列
$0 所有列
NR 行号
NF 列号
```

```
]#awk {print $1} abc # 输出第1列所有内容
]#awk {print $1,$3} abc #输出第1列和第三列
]#awk {print $0} abc # 输出所有列
]#awk {print $0,$1} abc # 列出所有列和第一列
]#awk {print NR,$0} abc #列出所有列的行号
]#awk {print NF,$0} abc # 列出所有行的行号
]#awk {print NR,NF} abc # 里出有多少行和列
```

### awk处理时机

```
awk [选项] '[条件]{指令}' 文件
awk  [选项]  'BEGIN{指令} {指令} END{指令}'  文件

BEGIN{ } 行前处理，读取文件内容前执行，指令执行1次
{ } 逐行处理，读取文件过程中执行，指令执行n次
END{ } 行后处理，读取文件结束后执行，指令执行1次
```

```
]#awk 'BEGIN{print "abc"}{print}' user
abc
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/root/lpd:/sbin/nologin

]# awk 'BEGIN{print "abc"}{print NR}END{print "over"}' user
abc
1
2
3
4
5
over
```

案例：格式化输出passwd中的User UID Home

```
]#awk -F: 'BEGIN{print"User\tUid\tHome"}{print $1"\t"$3"\t"$6}END{print "总共"NR"行"}' user
User    Uid     Home
root    0       /root
bin     1       /bin
daemon  2       /sbin
adm     3       /var/adm
lp      4       /var/root/lpd
总共5行
```

###  处理条件

***当定义了条件且指令就是print时可以省略指令不写***

> awk -F: '$6~/root/' user # 输出所有内容
>
> awk -F: '$6~/root/{print $6}' user # 只输出第6行

#### 设置条件

```
~：包含
!~：不包含
```

```
]#awk -F: '$6~/root/' user # 输出第6列包含root的行
root:x:0:0:root:/root:/bin/bash
lp:x:4:7:lp:/var/root/lpd:/sbin/nologin
]#awk -F: '$6!~/root/' user  #输出第6列不包含root的行
```

#### 使用数值

```
==(等于) !=（不等于） >（大于）
>=（大于等于） <（小于） <=（小于等于）
```

```
]#awk -F: '$3>=3' user # 输出第3列大于3的行
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/root/lpd:/sbin/nologin
]#awk -F: '$3<=3' user # 输出第3列小于3的行
```

#### 逻辑测试

```||
&&：两则都必须满足
||：满足一个显示一个
```

```
]# awk -F: 'NR<=3&&$3<=2' user 
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin

]# awk -F: 'NR<=3||$3<=1' user  # NR行号
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
```

#### 数学运算

```
+ - * / %
```

```
]# awk -F: 'NR%2==0{print NR,$0}' user #输出所有行号除2余0的所有行
2 bin:x:1:1:bin:/bin:/sbin/nologin
4 adm:x:3:4:adm:/var/adm:/sbin/nologin
```

### awk数组

```
定义数组的格式：数组名[下标]=元素值
调用数组的格式：数组名[下标]
```

```
]#awk 'BEGIN{a[1]=10;a[2]=20;print a[2],a[1]}'
20 10
```

#### awk for循环

```
for(变量 in 属组名){print 变量}
变量：属组的下标
作用：循环显示某数组的下标
vim abc
abc
xyz
abc
der
abc
xyz
```

```
]#awk '{a[$1]++}END{for(i in a){print i,a[i]}}' abc
der 1
xyz 2
abc 3
```

#### awk属组排序

```
针对文本排序输出可以采用sort命令，相关的常见选项为-r、-n、-k。其中
-n表示按数字顺序升序排列
-r表示反序
-k可以指定按第几个字段来排序
默认为升序
```

```
]#awk '{a[$1]++}END{for(i in a){print i,a[i]}}' abc | sort -nr k2
abc 3
xyz 2
der 1
```

### 案例

1. 变量与常量配合使用

   ```
   # 输出user文件中：用户的id是什么
   ]#awk -F: '/root/{print $1"的用户ID是"$3}' user #  操作的是root行
   root的用户ID是0
   
   # 查询磁盘容量大小
   ]#df-h | awk '/\/$/{print "根分区的剩余容量还剩于："$4}'
   根分区的剩余容量还剩于：12G
   ```

2. 使用awk统计访问网站的ip以及访问数量

   ```
   ]#awk '{a[$1]++}END{for(i in a){print i,a[i]}}' /var/log/httpd/access_log
   192.168.88.2 1
   192.168.88.254 22
   ```

   

# 快捷键



# 问题

### 1 简述awk工具的基本语法格式。

> awk [选项] '条件'{处理动作} 文件列表
>
> 命令 | awk [选项]'条件'{处理动作}

### 2 简述awk工具常用的内置变量、各自的作用。

> $

### 3 awk处理文本时，读文件前、读取文件内容中、读文件后后这三个环节是如何表示的？

### 4 提取当前eth0网卡的IPv4地址及掩码信息。

### 5 找出UID位于10~20之间的用户，输出用户名及对应的UID。

### 6 利用awk工具统计使用bash作为解释器的用户数量。

### 7 在awk中是否可以使用数组，分别以什么构成？

### 8 在linux中对文本的排序如何实现？

### 9 通过安全日志/var/log/secure，分析是谁登录账户tom且密码输入错误，如果错误次数达到3次以上的话，就输出登陆者的IP

# 补充



# 今日总结



# 昨日复习
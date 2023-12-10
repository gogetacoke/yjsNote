-- [学习目标](#学习目标)

- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [Sehll的使用方式](#sehll的使用方式)
  - [Shell脚本执行方法](#shell脚本执行方法)
    - [添加x权限](#添加x权限)
    - [使用解释器](#使用解释器)
    - [使用source或.](#使用source或)
  - [Shell脚本](#shell脚本)
    - [脚本文件](#脚本文件)
    - [简单demo](#简单demo)
    - [重定向](#重定向)
  - [Shell变量](#shell变量)
    - [调用变量](#调用变量)
    - [自定义变量](#自定义变量)
    - [取消变量](#取消变量)
    - [环境变量](#环境变量)
    - [位置变量](#位置变量)
    - [预定义变量](#预定义变量)
    - [变量的扩展](#变量的扩展)
      - [三种引号](#三种引号)
      - [使用read](#使用read)
      - [stty回显](#stty回显)
      - [使用export](#使用export)
  - [Shell数值运算](#shell数值运算)
  - [bc计算器](#bc计算器)
      - [交互式](#交互式)
      - [非交互式](#非交互式)
- [快捷键](#快捷键)
- [问题](#问题)
  - [如何执行一个未设置x权限的shell脚本](#如何执行一个未设置x权限的shell脚本)
  - [自定义变量时，变量有什么规则](#自定义变量时变量有什么规则)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)




# 学习目标

编写及执行脚本

Shell变量

数值运算

# 课堂笔记（命令）

# 课堂笔记（文本）

## 的使用方式

**交互式**

> 人工干预、智能化程度高
>
> 逐条解释执行、系哦啊率低

**非交互式**

> 需要提前设计、智能化难度大
>
> 批量执行、效率高
>
> 方便在后台静默执行

### 系统所有解释器

```
]#cat /etc/shells
```

## Shell脚本执行方法

前提：编写一个简单sh脚本

```
]#vim test.sh
echo 123
```

### 添加x权限

```
]./test.sh
-bash: ./test01.sh: 权限不够
]#chmod +x test.sh
]#./test.sh
123
```

### 使用解释器

无需x权限，但会开启解释器子进程

```
]#chmod -x test.sh  # 去掉x权限
]#bash test.sh
# 使用-x可进行排错检查，那一项执行报错
]#bash -x test.sh
+ echo 123
123
```

### 使用source或.

无需x权限，不会开启解释器子进程

不开启解释器：使用当前系统正在使用的解释器

```
]#source test.sh
123
]#. ./test.sh
123
```

## Shell脚本

### 脚本文件

.sh结尾的文件

### 简单demo

```
]#vim test1.sh
#!/bin/bash    # 指定解释器
# this is demo   # 注释
echo "123"  # 编写的脚本
]#source test1.sh  # 执行脚本
123

--------------------
vim test02.sh
#!/bin/bash
rm -rf /etc/yum.repos.d/*.repo
echo "
[name]
name=app
baseurl=xxx
gpgchechk=0
" > /etc/yum.repos.d/x.repo
```

### 重定向输出

> `>` 重定向标准输出
>
> `2>`重定向错误输出
>
> `&>`重定向所有输出

## Shell变量

变量：以固定名称存放，可能会变化的值

作用：提高脚本对任务需求、运行环境变化的适应能力;方便在脚本中重复使用

### 调用变量

```
]#vim test03.sh
a=1
echo $a
]#. ./test03.sh
1
```

### 自定义变量

变量可以用数字、字母、下划线，不能以数字开头，不能使用特殊符号，等号两边不能有空格,严格区分大下写

防止变量与常量混淆，加入一个{}

```
]#a=1
]#ab12=3
]#a_2=jj

]#b=100
]#echo {$b}RMB
100RMB
```

### 取消变量

```
1. unset 变量名
2. 变量名=
例：
]#a=10
]#echo $a
10
]#unset a

#或则
]#a=  #将a的值赋空
```

### 环境变量

> $USER #显示当前用户名
> $UID # 显示当前用户uid
> $HOME # 显示到当前用户主目录
> $SHELL # 解释器
> $PWD
> $HOSTNAME
> $PS1 #一级提示符  [\u@\h \W]\$
> $PS2
> $PATH # 存储了命令的路径

```
]#echo $USER
root
]#USER=XXX #临时修改用户名
]#vim /etc/profile # 永久修改（所有用户）
]# vim ~/.bash_profile  # 针对单个用户
```

### 位置变量

脚本执行时，后面跟的参数;

类似函数中的位置参数;

```
$n # 当前脚本的第n个参数
]#vim /test04.sh
echo $1
echo $2
echo $4
]#. test04.sh abc 5 6 9 #位置参数跟在脚本后，空格隔开
abc
5
9
```

### 预定义变量

```
]#vim test05.sh
$0 #bash运行输出脚本名，source运行输出解释器名
$* # 输出所有的参数
$# # 输出参数的个数
$$ # 输出当前脚本的PID
$? # 输出上一个命令运行是否成功，0成功 0其他表示不成功
]#bash test05.sh 1 2 3 4 # 使用bash运行脚本，添加了4个参数
test05.sh
1 2 3 4
4
11410
0
]#source test05.sh 1 2 3 4 # 使用source允许脚本
-bash
1 2 3 4
4
36076
0
```

### 变量的扩展

#### 三种引号

**双引号**

可以界定一个完整的字符串

```
]#x=a b
bash: b: 未找到命令...
]#x="a b"
]#echo $x
a b
]#touch a b
]#ls
5  a  b  test01.sh
]#touch "a b"
 5  a b 'a b'
```

**单引号**

界定一个完整的字符串，并且可以实现屏蔽特殊符号的功能

```
]# test=12
]# echo "$test"
12
]# echo '$test'
$test
```

**反撇号**

使用**反撇号**或**$()**,可以将命令执行标准输出作为字符串存储，称为命令替换

```
]# a=`date`
]# echo $a
2023年 11月 29日 星期三 17:18:31 CST
]# a=$(date)
]# echo $a
2023年 11月 29日 星期三 17:18:44 CST
```

#### 使用read

执行后会等待并接受用户输入，并赋值给变量

read -p 给定用户提示信息 a存储用户输入字符的变量

```
# 基本使用
]# echo $str
what's your name?
]# read str
what's your name?

# 交互式
]# read -p "what's your name?" n
what's your name?yyh
[root@pc2 opt]# echo $n
yyh
```

```
]#vim test06.sh
#!/bin/bash
read -p please.enter.username: n
useradd $n
read -p please.enter.password: p
echo $p | passwd --stdin $n
echo user{$n} created successful password is {$p}

]#. test06.sh
please.enter.username:tom
please.enter.password:123456
更改用户 tom 的密码 。
passwd：所有的身份验证令牌已经成功更新。
user{tom} created successful password is {123456}
```

#### stty回显

开启命令回显功能 stty echo

开启后，控制台输入的东西，将不会会显，敲击回车后，输入stty echo 敲击回车将会恢复

```
]#stty -echo # 关闭回显
]#    #输入pwd
/
]#  #输入stty echo 开启回显
]#pwd
/

# 应用与脚本
]#vim test07.sh
#!/bin/bash
read -p please.enter.username: n
useradd $n
stty -echo
read -p please.enter.password: p
stty echo
echo $p | passwd --stdin $n
echo user{$n} created successful password is {$p}
]#. test07.sh
please.enter.username:tom
please.enter.password:
更改用户 tom 的密码 。
passwd：所有的身份验证令牌已经成功更新。
user{tom} created successful password is {123}
```

#### export全局变量

默认情况下，自定义的变量为局部变量，只在当前Shell环境中有效，而在子Shell环境中无法直接使用。

```
]# yy="abc"
]# echo $yy                        #查看yy变量的值，有值
abc
]# bash                              #开启bash子进程
]# echo $yy                          #查看yy变量的值，无值

]# exit                              #返回原有Shell环境
]# echo $yy                        #再次查看，有值
abc
]# export yy                              #发布已定义的变量
]# bash                                  #进入bash子Shell环境
]# echo $yy                              #查看全局变量的值
abc
```

## Shell数值运算

1. expr     乘法操作应采用 `\*`转义，避免被作为Shell通配符；参与运算的整数值与运算操作符之间需要以空格分开，引用变量时必须加$符号

   ```
   ]# X=1234                              #定义变量X
   ]# expr  $X  +  78                      #加法
   1312
   ]# expr  $X  -  78                       #减法
   1156
   ]# expr  $X  \*  78                      #乘法，操作符应添加\转义
   96252
   ]# expr  $X  /  78                      #除法，仅保留整除结果
   15
   ]# expr  $X  %  78                     #求模
   64
   ```

2. $[] 或 $(())     乘法操作*无需转义，运算符两侧可以无空格；引用变量可省略 $ 符号；计算结果替换表达式本身，可结合echo命令输出。

   ```
   # X=1234   
   ]# echo $[X+78]
   1312
   ]# echo $[X-78]
   1156
   ]# echo $[X*78]
   96252
   ]# echo $[X/78]
   15
   ]# echo $[X%78]
   64
   ```

3. let    expr或$[]、$(())方式只进行运算，并不会改变变量的值；而let命令可以直接对变量值做运算再保存新的值。

   ```
   常规写法         主流写法
   let a=a+1       let a++         #变量a加1
   let a=a-1       let a--          #变量a减1
   let a=a+10      let a+=10       #变量a加10
   let a=a-10      let a-=10        #变量a减10
   let a=a*2       let a*=2         #变量a乘以2
   let a=a/2       let a/=2        #变量a除以2
   let a=a%3       let a%=3        #变量a除以3取余数
   ```

## bc计算器

#### 交互式

先执行bc命令进入交互环境，然后再输入需要计算的表达式。

```
]# bc
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'
2*3
6
quit # 退出
```

#### 非交互式

将需要运算的表达式通过管道操作交给bc运算。注意，小数位的长度可采用scale=N限制。

```
]# echo "1+3" | bc
4
]# echo "scale=3;10/3"| bc
3.333
```





# 快捷键



# 问题

## 如何执行一个未设置x权限的shell脚本

> bash 脚本名
>
> source 脚本名
>
> . 脚本名

## 自定义变量时，变量有什么规则

> 以数字、字母、下划线组成，不能以数字开头，不能定义系统默认变量

## 简述预定义变量$$、$?、$0、$#、$*、$!作用

> $$ 输出当前脚本PID
> $? 判断上个命令执行是否成功
> $0 bash执行输出当前脚本名字，source执行输出当前脚本PID
> $# 输出所有参数
> $* 输出参数个数
> $! 

## 简述三种定界符在赋值操作中的特点

> 单引号：屏蔽特殊字符，界定一个完整字符
>
> 双引号：界定一个完整的字符，以$引用其他变量
>
> 反撇号、$()：命令替换，可将命令赋值给变量



# 补充

bash -x  可以进行排错

# 今日总结



# 昨日复习
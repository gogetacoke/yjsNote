- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
  - [ss](#ss)
  - [RANDOM](#random)
- [课堂笔记（文本）](#课堂笔记文本)
  - [While循环](#while循环)
  - [循环中断](#循环中断)
    - [exit](#exit)
    - [break](#break)
    - [continue](#continue)
  - [case分支](#case分支)
  - [echo -e](#echo--e)
  - [函数](#函数)
  - [字符串](#字符串)
    - [变量](#变量)
      - [字符串截取](#字符串截取)
      - [字符串替换](#字符串替换)
      - [字符串删除](#字符串删除)
      - [获取长度](#获取长度)
      - [初始值](#初始值)
- [快捷键](#快捷键)
- [问题](#问题)
  - [简述定义一个Shell函数的两种方法](#简述定义一个shell函数的两种方法)
  - [简述Shell环境常见的中断及退出控制指令](#简述shell环境常见的中断及退出控制指令)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)




# 学习目标

while循环

中断控制

函数

字符串处理

# 课堂笔记（命令）

## ss

查看系统中启动的端口信息

ss命令可以查看系统中启动的端口信息，该命令常用选项如下：

-n以数字格式显示端口号

-t显示TCP连接的端口

-u显示UDP连接的端口

-l显示服务正在监听的端口信息，如httpd启动后，会一直监听80端口

-p显示监听端口的服务名称是什么（也就是程序名称）

```sh
]#ss -ntulp | grep nginx
tcp   LISTEN 0      128          0.0.0.0:80         0.0.0.0:*    users:(("nginx",pid=12336,fd=8),("nginx",pid=12335,fd=8))
```

## RANDOM

```sh
]#echo $RANDOM # 随机生成0-32767之间的一个整数
326
```

# 课堂笔记（文本）

## While循环

```sh
# 语法格式

while 条件测试/命令
do
	命令
	....
done
```

```sh
#!/bin/bash
i=1
while [ $i -le 5 ]
do
        echo "$i"
        let i++
done
```

## 循环中断

### exit

退出脚本

```sh
#!/bin/bash
for i in {1..5}
do
	[ $i -eq 3 ] && exit
	echo $i
done
echo over
]#bash test.sh
ok
ok
```

### break

跳出当前循环体，执行循环体后的内容

```sh
#!/bin/bash
for i in {1..5}
do
        [ $i -eq 3 ] && break
        echo ok
done
echo over
]#bash test.sh
ok
ok
over
```

**案例：**用户输入值，将累加，输入0后退出脚本输出总和

```sh
#!/bin/bash
sum=0
while :
do
        read -p "please enter num sum(input 0 exit): " num
        [ $num -eq 0 ] && break
        let sum+=num
done
echo "sum: "$sum
```

### continue

跳过循环体内余下的语句，重新判断条件以决定是否需要执行下一次循环

```sh
#!/bin/bash
for i in {1..5}
do
        [ $i -eq 3 ] && continue
        echo ok$i
done
echo over
]#bash test03.sh 
ok1
ok2
ok4
ok5
over
```

**案例：**完善break案例，用户输入空时，让用户重新输入

```sh
#!/bin/bash
sum=0
while :
do
    read -p "please enter num sum(input 0 exit): " num
    [ -z $num ] && continue 
    [ $num -eq 0 ] && break
    let sum+=num
done
echo "sum: "$sum
```

## case分支

```sh
# 语法格式

case 变量 in
模式1)
	命令序列1;;
模式2)
	命令序列2;;
*)
	默认命令序列
esac
```

```sh
case $1 in
abc)
        echo 123;;
cbd)
        echo 321;;
e|d) # e或d 都进行输出
		echo ok;;
*)  # *代表任意
        echo haha
esac
]#bash test.sh abc
123
]#bash test.sh cbd
321
]#bash test.sh e
ok
]#bash test.sh d
ok
]#bash test.sh abcss
haha
]#bash test.sh abc1556156
haha
```

**案例：**

使用脚本1安装nginx

脚本2启停以及状态查询nginx

```sh
#!/bin/bash  # 安装所需依赖以及编译工具
yum -y install gcc make pcre-devel openssl-devel
tar -xf nginx-1.22.1.tar.gz
cd nginx-1.22.1
./configure
make
make install

# 手动启动
/usr/local/nginx/sbin/nginx
# 手动关闭
/usr/locl/nginx/sbin/nginx -s stop

#!/bin/bash
case $1 in 
start)
        /usr/local/nginx/sbin/nginx;;
stop)
        /usr/local/nginx/sbin/nginx -s stop ;;
restart)
        /usr/local/nginx/sbin/nginx -s stop
        /usr/local/nginx/sbin/nginx;;
status)
        ss -ntulp | grep -q  nginx  # 使用grep -q 查询到内容静默，不输出
        [ $? -eq 0 ] && echo Be running || echo start failed;; # 查询到则使用$?判断
*)
        echo input error
esac
```

## echo -e

```sh
# 字体颜色变化

]#echo -e "\033[32m\abcd\033[0m"
# \033[32m 转义序列，用于设置终端文本颜色，32m为绿色，32是颜色代码
# \abcd 需要设置颜色并输出的文本
# \033[0m 转义序列，用于恢复终端文本默认颜色	，[0m表示文本样式重置为默认值
abcd
```

## 函数

```sh
# 语法格式
function 函数名 {
	命令
}
或
函数名(){
	命令
}
```

**案例：**编写一个设置颜色的函数

```sh
#!/bin/bash
color(){
	echo -e "\033[$1m$2\033[0m"
}
color 32 abc  # 函数体内的参数，需要在调用函数时进行使用
color 33 cbd
color $1 $2  # 可以使用此方法，在脚本外添加参数
#]. test.sh 33 cbd # 设置颜色代码33 使其cbd变色
```

## 字符串

### 变量

#### 字符串截取

```sh
# 格式
echo ${变量名:截取位置:截取长度} # 位置长度：以0开头
```

```sh
]#a=abcdefg
	0123456# 下标-正序

]#echo ${a:3:2} # 正序截取，是截取位置后的长度
ef
     g f e d c b a   	
	-7-6-5-4-3-2-1  # 下标-倒序
]#echo ${a:-3:1} # 倒序进行截取时，从-1开始，截取当前位置到后的长度
e
```

**案例：**随机生成密码

```sh
# 首先实现1个字符的随机产生
#!/bin/bash
x=abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789
n=$[RANDOM%62] #得到0～61随机数存在变量n中，通过取余
p=${x:n:1} # 通过截取，将1个随机字符赋值给变量p

# 完善
#!/bin/bash
num=abcdefghjklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789
pwd= 
for i in {1..8}
do
    n=$[RANDOM%62]
    p=${num:n:1}
    pwd+=$p
done
echo $a      
]#. test.sh
zIFJLlDd
```

#### 字符串替换

```sh
# 语法格式
${变量名/old/new} # 单独替换一个字符
${变量名//old/new}# 换多个
```

```sh
]#a=3211315564113111118564
]# echo ${a/1/2}
3221315564113111118564
]# echo ${a/11/A}
32A315564113111118564
]# echo ${a//1/B}
32BB3B5564BB3BBBBB8564
```

#### 字符串删除

```sh
#语法格式
${变量名#*关键词} # 从左向右删除-最短匹配，匹配及停止
${变量名%关键词*} # 从右向左删除-最短匹配，匹配及停止

单个（#/%）：删除以什么开头/结尾，匹配单个字符，且开头或结尾必须是你写的值
例如
a=as356
echo ${a#s}  # 为什么返回原数据？因为未找到以s开头的字符串
as356
echo ${a#a}
s356

a=as356
echo ${a%s}
as356
echo ${a%6}
as35

${变量名##*关键词} # 从左向右删除-最长匹配，直到匹配到没有关键词为止
${变量名%%关键词*} # 从右向左删除-最长匹配，直到匹配到没有关键词为止

# 多个（#/%*）：删除内容可以不是以它开头，删除的内容需在给定的字符里面，配合*匹配所有使用，注意*号在匹配字符的前后位置，在后是删除以整个字符开头，反之删除字符后面内容
```

```sh
]# a=abcdef
]# echo ${a%bc}
abcdef
]# echo ${a%fe}
abcdef
]# echo ${a%abc}
abcdef
]# echo ${a#abc}
def

]#a=abcbcaadek
]#echo ${a##*c} # `##`代表匹配当前字符串最后一个字符c，删除c前面所有内容，留取c后内容，不包括c
aadek
]#a=abcbcaadek
]#echo ${a%%c*} # `%%`从右边开始匹配到最后一个c，删除c后所有内容，留取c前面内容，不包括c
ab
```

#### 获取长度

```sh
# 格 式
echo ${#变量名}
```

```sh
]#a=456321
]#echo ${#a}
6
```

#### 初始值

当变量为空时，自动运用变量的初始值，不为空则，还是运用本身值

```sh
]#c=${c:-123}
]#echo $c
123
]#c=456
]$echo $c
456
```



# 快捷键



# 问题

## 简述定义一个Shell函数的两种方法

> function 函数名{
>
> ​	command
>
> }
>
> 函数名(){
>
> ​	command
>
> }

## 简述Shell环境常见的中断及退出控制指令

> exit：退出当前脚本，若使用source将退出整个解释器
>
> break：结束当前所有循环，接着执行循环下面的所有内容
>
> continue：跳过本次循环，直到循环结束，接着执行循环后的内容s

# 补充

grep -p  静默查询内容 类似丢进/dev/null

# 今日总结



# 昨日复习
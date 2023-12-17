- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
  - [rsync](#rsync)
    - [本地同步](#本地同步)
    - [远程同步](#远程同步)
    - [实时数据同步](#实时数据同步)
    - [使用Shell编写一个简单实时脚本](#使用shell编写一个简单实时脚本)
- [课堂笔记（文本）](#课堂笔记文本)
  - [源码编译安装](#源码编译安装)
    - [解压源码至指定位置](#解压源码至指定位置)
    - [运行configure脚本](#运行configure脚本)
    - [进行make编译](#进行make编译)
    - [make install安装](#make-install安装)
    - [卸载](#卸载)
  - [MariaDB数据库系统](#mariadb数据库系统)
    - [安装数据库](#安装数据库)
    - [基本使用](#基本使用)
    - [恢复数据到数据库](#恢复数据到数据库)
    - [查询](#查询)
    - [增加](#增加)
    - [修改](#修改)
    - [删除](#删除)
    - [修改数据库密码](#修改数据库密码)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)



# 学习目标

源码编译安装

rsync本地同步

rsync远程同步

inotify实时同步

数据库服务基础

# 课堂笔记（命令）

## rsync

格式：rsync [选项] 源目录	目标目录

-n：测试同步过程，不做实际修改

`--`delete：删除目标文件夹内多余的文档

-a：递归模式

-v：显示详细操作信息

-X：保持acl策略不变

### 装包

```
yum -y install rsync
```

### 本地同步

```
]#mkdir /mydir /todir
]#echo 123 > /mydir/1.txt
]#echo 321 > mydir/2.txt
]#ls -l /mydir
1.txt 2.txt
]#ls -l /todir
]#rsync -avX --delete /mydir/ /todir/
sending incremental file list
./
1.txt
2.txt

sent 284 bytes  received 31 bytes  630.00 bytes/sec
total size is 12  speedup is 0.04

]#ls -l /todir
1.txt 2.txt
```

代码解释：

数据同步，只同步源数据里最新内容到目标目录

`--`delete：若目标目录内容增加一条，源目录没有，敲击代码后，目标目录内容将将被覆盖成源目录内容

### 远程同步

> 下行：rsync [...] user@host:远程目录 本地目录
>
> 上行：rsync [...] 本地目录 user@host:远程目录

**虚拟机A**

```
~]# rsync -avX  --delete /mydir/   root@192.168.88.2:/nsd10
 ……**..**connecting **(**yes**/**no)? yes
root@192.168.88.2's password:         #输入密码
```

**虚拟机B**

```
]#ls /nsd10
```

### 实时数据同步

前提：需安装 inotifywait-tools程序

**A机器配置**

```
]#ssh-keygen # 一路回车
]#ls /root/.ssh # 查看生成公私钥
id_rsa(私钥)   id_rsa.pub(公钥)    known_hosts(记录曾经远程管理过的机器)
]#ssh-copy-id root@192.168.88.2 # 将公钥传递给B机器
```

**监控目录内容变化工具**

inotifywait [选项] 目标文件夹

-m：持续监控（捕获一个事件后不退出）

-r：递归监控

-q：减少屏幕输出信息; -qq：屏幕不输出

-e：指定见识的信息modify、move、create、delete、attrib等事件类别

例：监控源目录，发生变化便进行通知

```
]#/opt/myrpms/bin/inotifywait -rq /mydir：
/mydir/ CREATE 5.txt
]#ls /mydir菜单
  1.txt  2.txt  5.txt  hh  test
```

### 使用Shell编写一个简单实时脚本

```
[root@server /]# vim    /root/hello.sh  
echo  hello  world
hostname
id  root
ifconfig   |   head  -2
[root@server /]# chmod   +x   /root/hello.sh  #所有人赋予执行权限
[root@server /]# /root/hello.sh   #绝对路径执行脚本
重复性：循环解决  
格式：  
     while   条件
     do
          重复执行的事情
     done
[root@server /]# vim   /etc/rsync.sh   
while   /opt/myrpm/bin/inotifywait  -rqq   /mydir/
do
rsync -aX  --delete   /mydir/   root@192.168.88.2:/nsd10
done
[root@server /]# chmod  +x  /etc/rsync.sh  #赋予执行权限
[root@server /]# /etc/rsync.sh   & #放入后台运行脚本程序
[root@server /]# jobs  -l    #-l选项  显示进程的pid
[1]    + 17707 运行中               /etc/rsync.sh &
[root@server /]# kill  17707      #停止脚本，可以杀死进程
```



# 课堂笔记（文本）

## 源码编译安装

RPM软件包：gcc、make

前提：拥有源码包;

### 解压源码至指定位置

```
]#tar -xf /usr/local/inotify-tools-3.13 /usr/local/
```

### 运行configure脚本

作用：检测是否安装gcc、指定安装位置、生成Makefile文件

```
]#cd /usr/local/inotify-tools-3.13/
]#./configure --prefix=/opt/myrpms # 指定位置安装，此操作不产生相应的目录
```

### 进行make编译

作用：变成可执行的程序，放入内存中

```
]#cd /usr/local/inotify-tools-3.13/
]#make
```

### make install安装

```
]#cd /usr/local/inotify-tools-3.13/
]#make install
]#ls /opt
myrpms

]#ls /opt/myrpms
bin  include  lib  share
```

### 卸载

```
]#rm -rf /opt/mrpms
```

## MariaDB数据库系统

### 安装数据库

```
]#yum -y install mariadb-server
]#systemctl restart mariadb
```

### 基本使用

```
]#mysql #进入数据系统
MariaDB [(none)]> show databases; #列出当前数据库
>create datases test; # 创建数据库
>create datases	nsd10;
>show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
| nsd10              |
+--------------------+
>drop database nsd10; #删除指定数据库
>use mysql; #切换到msql数据库 
>show tables; # 查看库中所有表格
```

### 恢复数据到数据库

格式：库 < sql文件

```
]#mysql test < /root/user.sql  # 将user.sql数据恢复到test库中
]#mysql
>use test;
>show tables;
+----------------+
| Tables_in_test |
+----------------+
| base           |
| location       |
+----------------+
2 rows in set (0.000 sec)
```

### 查询

```
>select * from base; #查询base表中的所有内容
+------+---------+------------+
| id   | name    | password   |
+------+---------+------------+
|    1 | Tom     | 123        |
|    2 | Barbara | 456        |
|    3 | James   | solicitous |
|    4 | Smith   | tarena     |
|    5 | Barbara | pwd123     |
+------+---------+------------+
5 rows in set (0.000 sec)
> select name from base; #只查询base表中的name字段
+---------+
| name    |
+---------+
| Tom     |
| Barbara |
| James   |
| Smith   |
| Barbara |
+---------+
5 rows in set (0.001 sec)
> select * from base where password='456';
+------+---------+----------+
| id   | name    | password |
+------+---------+----------+
|    2 | Barbara | 456      |
+------+---------+----------+

> select * from base where password='456' or id='1';
+------+---------+----------+
| id   | name    | password |
+------+---------+----------+
|    1 | Tom     | 123      |
|    2 | Barbara | 456      |
+------+---------+----------+

> select * from base where password='456' and id='1';
Empty set (0.000 sec)
```

### 增加

```
>insert base values('4','xixi','123');
>select * from base;
+------+---------+------------+
| id   | name    | password   |
+------+---------+------------+
|    1 | Tom     | 123        |
|    2 | Barbara | 456        |
|    3 | James   | solicitous |
|    4 | Smith   | tarena     |
|    5 | Barbara | pwd123     |
|    9 | xixi    | 123        |
+------+---------+------------+
```

### 修改

```
>update base set password='456' where id='9';
>select * from base;
+------+---------+------------+
| id   | name    | password   |
+------+---------+------------+
|    1 | Tom     | 123        |
|    2 | Barbara | 456        |
|    3 | James   | solicitous |
|    4 | Smith   | tarena     |
|    5 | Barbara | pwd123     |
|    9 | xixi    | 456        |
+------+---------+------------+
6 rows in set (0.000 sec)
```

### 删除

```
> delete from base where id='9';
> select * from base;
+------+---------+------------+
| id   | name    | password   |
+------+---------+------------+
|    1 | Tom     | 123        |
|    2 | Barbara | 456        |
|    3 | James   | solicitous |
|    4 | Smith   | tarena     |
|    5 | Barbara | pwd123     |
+------+---------+------------+
5 rows in set (0.000 sec)
```

### 修改数据库密码

```
]#mysqladmin -u root password='132'; #初始修改密码
]#mysql -u root -p # 登录

]#mysqladmin -u root -p'132' password='456'; # 修改密码
```



# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习
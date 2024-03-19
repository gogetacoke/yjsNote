- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
	- [LAMP构建](#lamp构建)
		- [装包](#装包)
		- [编写页面](#编写页面)
		- [部署数据库](#部署数据库)
		- [授权用户](#授权用户)
		- [部署协同工作软件](#部署协同工作软件)
	- [Ubuntu](#ubuntu)
		- [查看系统版本](#查看系统版本)
		- [编写仓库](#编写仓库)
		- [更新仓库配置](#更新仓库配置)
		- [软件安装卸载](#软件安装卸载)
		- [软件基本操作](#软件基本操作)
	- [时间同步](#时间同步)
		- [装包](#装包-1)
		- [修改配置](#修改配置)
		- [重启服务](#重启服务)
		- [调用同步](#调用同步)
	- [邮件收发](#邮件收发)
		- [装包配置](#装包配置)
		- [发送邮件](#发送邮件)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)




# 学习目标

构建LAMP平台

# 课堂笔记（命令）



# 课堂笔记（文本）

## LAMP构建

### 装包

```sh
]#yum -y install hhtpd
]#yum -y install php php-xml php-json
```

### 编写页面

```sh
]vim /var/www/html/test.php #编写一个输出php配置信息的动态网页
<?php
	phpinfo();
?>
]#systemctl restart httpd # 重启服务
]#curl 192.168.88.3/test.php # 访问php网页
```

[注]：php配置文件在httpd的访问配置文件的地发  /etc/httpd/conf.d/php.conf

### 部署数据库

```sh
]#yum -y install mariadb-server
]#systemctl restart mariadb # 启动数据库服务
]#mysql # 进入mysql
]>create database nsd; # 创建数据库
]>show databases; #查看是否创建成功
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| nsd                |
| performance_schema |
+--------------------+
```

### 授权用户

```sh
]>grant all on nsd.* to lisi@localhost identified by '123'; # 授权用户lisi所有权限访问nsd数据库
Query OK, 0 rows affected (0.001 sec)
]>exit
]#mysql -u lisi -p123
]>show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| nsd                |
+--------------------+
```

代码解释：

grant all on nsd.* to lisi@localhost identified by ‘123’;

授权所有权限（crud）在数据库nsd的所有表给到本机用户lisi，其设置用户密码为123，用户不存在将自动创建

### 部署协同工作软件

```sh
]#yun -y install php-mysqlnd # 便于php操作数据库
```

## Ubuntu

### 查看系统版本

```sh
$lsb_release -a 
```

### 编写仓库

```sh
$mount /dev/cdrom /mnt # 挂载
$cp /etc/apt/sources.list /opt #备份
$sudo vi /etc/apt/sources.list /opt # 编写仓库内容
deb file:///mnt jammy main
```

### 更新仓库配置

```sh
$sudo apt-get update
```

### 软件安装卸载

```sh
$sudo apt list #查询安装包列表
$sudo apt-get install make # 安装make
$sudo apt-get remove make # 卸载
```

### 软件基本操作

```sh
$sudo dpkg -l make #查询软件是否安装
$sudo apt-get install make 
$sudo dpkg -l make 
$sudo dpkg -L make # 列出软件的安装清单
```

## 时间同步

### 装包

```sh
]#yum -y install chrony
```

### 修改配置

以下是针对本机搭建时间服务器

```sh
]#vim /etc/chrony.conf
末行模式：set nu #开启行号
3 #pool 2.pool.ntp.org iburst #注释
23 allow all # 将注释去掉，删除IP内容内容加上all
26 local stratum 10 # 注释去掉，用于设置本机第10层时间服务器
```

### 重启服务

```sh
]#systemctl restart chronyd
```

### 调用同步

B机器操作：

B机器也需安装chronyd软件包

```sh
]#vim /etc/chrony.conf
# pool 2.pool.ntp.org iburst # 将内容改为 server 192.168.88.240  iburst # ip为时间服务器

]#date -s ‘2008-1-1‘ # 修改错误时间
]#systemctl restart chronyd # 重启服务
]#date # 查看时间 隔几秒钟再查看
```

## 邮件收发

### 装包配置

```sh
]#yum -y install postfix mailx # 邮件服务、编写格式
]#useradd xyy # 创建两个用户
]#useradd myy
```

### 发送邮件

```sh
]#echo ’this is test mail‘ | mail -s "content" -r xyy myy # 使用非交互式发送邮件
-s：标题
echo：输入发送的内容
-r：谁要发给谁
]#mail -u myy # 查看某个用户收到的邮件
& 1 # 输入邮件编号，回车
& exit # 退出
]#mail -s 'aa' -r myy xyy # 交互式发送
hahaha
. # 一行只有一个点表示提交
EOT
```



# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习
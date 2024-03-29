- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [SELinux运行模式的切换](#selinux运行模式的切换)
    - [临时切换](#临时切换)
    - [永久切换](#永久切换)
  - [破解root用户密码](#破解root用户密码)
    - [运行模式为紧急](#运行模式为紧急)
    - [运行模式为宽松](#运行模式为宽松)
  - [密码防护](#密码防护)
  - [**HTTPD服务**](#httpd服务)
  - [FTP服务](#ftp服务)
  - [防火墙](#防火墙)
    - [默认区域修改](#默认区域修改)
    - [public区域添加规则](#public区域添加规则)
    - [单独拒绝某主机的所有访问](#单独拒绝某主机的所有访问)
  - [程序的运行](#程序的运行)
  - [启动](#启动)
  - [重启](#重启)
  - [查看开机启动](#查看开机启动)
  - [设置开机自启](#设置开机自启)
  - [关闭自启动](#关闭自启动)
  - [字符图形模式切换](#字符图形模式切换)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)

# 学习目标

SELinux

系统故障修复

防火墙策略

服务管理

# 课堂笔记（命令）

# 课堂笔记（文本）

## SELinux运行模式的切换

> enforcing（强制）、permissive（宽松，80%）、disabled（彻底禁用）
>
> [注]：任何模式切换到彻底禁用或彻底禁用切换到其他两种都需要重启

### 临时切换

[注]：宽松(0)与强制(1)之间切换

```sh
]#getenforce #查看当前运行模式
Enforcing
]#setenforce 0 #设置模式
Permissive
```

### 永久切换

/etc/setlinux/confg

```sh
]# vim /etc/selinux/config 
SELINUX=Permissive
```

## 破解root用户密码

### 运行模式为强制

```sh
]#getenforce # 查看当前运行模式
Enforcing
]#reboot
按键盘上下键进入选择系统界面，按E键进入救援模式
将Linux该行中的ro修改为rw空格输入rd.break 按ctrl+x启动，显示switch_root:/# 
]#chroot /sysroot 切换环境
]#touch /.autorelable #使linux失忆
]#echo 1 | passwd --stdin root # 使用无交互式修改密码
]#reboot -f /#强制重启主机
```

### 运行模式为宽松

```sh
]#getenforce
Permissive
]#reboot
按键盘上下键进入选择系统界面，按E键进入救援模式
将Linux该行中的ro修改为rw空格输入rd.break 按ctrl+x启动，显示switch_root:/# 
]#chroot /sysroot 切换环境
]#echo 1 | passwd --stdin root # 使用无交互式修改密码
]#reboot -f /#强制重启主机                                                                            
```

## 密码防护

设置救援模式密码

```sh
]#grub2-setpassword
Enter password: #输入密码，密码不显示
Confirm password: #重新输入密码，密码不显示
[root@A ~]# cat /boot/grub2/user.cfg #存放grub密码的文件
```

## **HTTPD服务**

**安装**

```sh
]#yum -y install httpd
```

**开启服务**

```sh
]#rpm -ql httpd|grep sbin
/usr/sbin/httpd
]#/usr/sbin/httpd
]# pgrep -l httpd
11834 httpd
```

**测试**

```sh
]#touch /var/www/html/index.html
内容：China Number One
]#curl 192.168.88.240 #输入本机ip
China Number One
```

## FTP服务

**安装**

```sh
]#yum -y install vsftpd
```

**修改配置文件**

作用：开启无密码验证

```sh
]#vim /etc/vsftpd/vsftpd.conf
将NO改为YES，保存退出
anonymous_enable=YES
```

**开启服务**

```sh
]#/usr/sbin/vsftpd
```

**测试**

```sh
]#touch /var/ftp/dc.txt
]#curl ftp://192.168.88.240/
-rw-r--r--    1 0        0               0 Nov 14 07:19 dc.txt
drwxr-xr-x    2 0        0               6 Apr 23  2022 pub
```

## 防火墙

作用：隔离，严格过滤入站，放行出战

根据所在的网络场所区分，预设区域：

+ public 仅允许访问本机的ssh、dhcp、ping服务
+ trusted 允许任何来访
+ block 拒绝任何来访请求，明确拒绝客户端
+ drop 丢弃任何来访的数据包，不给任何回应

防火墙判定规则：

1. 查看客户端请求中来源的ip地址，查看自己所有区域中规则，那个区域有该源ip地址规则，则进入该区域
2. 进入默认区域（默认情况为public）

### 默认区域修改

[注]：访问都需要在同一个网络里进行

```sh
]# firewall-cmd    --get-default-zone   #查看默认区域
public
]#firewall-cmd --set-default-zone=trusted
success
```

### public区域添加规则

[注]：service添加的是允许的协议

```sh
]# firewall-cmd --zone=public --list-all # 查询允许的协议
]#firewall-cmd --zone=public --add-service=http # 添加http协议
]#firewall-cmd --zone=public --add-service=ftp # 添加ftp协议
```

**永久添加**

```sh
]#firewall-cmd --permanent --zone=public --add-service=http # 永久添加http协议
]#firewall-cmd --reload #加载防火墙策略
]#firewall-cmd --zone=public --list-all # 查询
```

### 单独拒绝某主机的所有访问

```sh
]#firewall-cmd --zone=block --add-source=192.168.88.2 # 将ip192.168.88.2设置黑名单
]#firewall-cmd --zone=block --remove-source=192.168.88.2#删除策略
```

## 程序的运行

## 启动

```sh
]#killall httpd
]#systemctl start httpd #启动服务httpd
```

[注]：服务已启动，就不能执行该程序，需提前使用systemctl status 程序名 检查是否启用

## 重启

```sh
]#systemctl restart httpd
```

## 查看开机启动

```sh
]#systemctl is-enabled httpd
isabled
```

## 设置开机自启

```sh
]#systemctl enable httpd
```

## 关闭自启动

```sh
]#systemctl disenable httpd
```

## 字符图形模式切换

```sh
]#init 3 # 切换字符模式 想当与 systemctl isolate multi-user.tgreget
]#init 5 # 切换到图形模式，相当于 systemctl iolate graphical.target
```

[注]：图形模式，是装机时安装了图形界面

# 快捷键

# 问题

# 补充

# 今日总结

# 昨日复习

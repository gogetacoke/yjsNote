- [今日计划](#今日计划)
- [本周复习内容](#本周复习内容)
  - [修改网卡命名规则](#修改网卡命名规则)
  - [nmcli网卡配置](#nmcli网卡配置)
  - [远程管理](#远程管理)
  - [数据传输](#数据传输)
  - [临时管理IP](#临时管理ip)
  - [日志管理](#日志管理)
  - [系统安全保护](#系统安全保护)
  - [破解root密码](#破解root密码)
  - [构建FTP服务](#构建ftp服务)
  - [防火墙策略管理](#防火墙策略管理)
  - [服务管理：程序的运行](#服务管理程序的运行)
  - [WEB服务](#web服务)
- [复习问题总结](#复习问题总结)
- [预习下周内容](#预习下周内容)

# 今日计划

- [ ] 整理一周笔记

- [ ] 制作本周思维导图

- [ ] 复习下周内容

# 本周复习内容

## 修改网卡命名规则

```
]#vim /etc/default/grub
GRUB_CMDLINE_LINUX="crashkernel=auto resume=/dev/mapper/rl-swap rd.lvm.lv=rl/root rd.lvm.lv=rl/swap rhgb quiet net.ifnames=0 biosdevname=0"

]#grub2.mkconfig -o /boot/grub2/grub.cfg # 重新生成网卡命名规则
]#reboot # 重启系统
```

## nmcli网卡配置

**查看网络命名的连接**

```
]# nmcli connection show
```

**删除错误网卡名**

```
]#nmcli connection show
NAME   UUID                                  TYPE      DEVICE 
enp1s0 5b551-763a-4a01-856c-b4a1c8c80357     ethernet  enp1s0   
]#nmcli conneciton delete enp1s0 # 可通过name或UUID删除
Connectio 'enp1s0' (5b551-763a-4a01-856c-b4a1c8c80357) successfully deleted
```

**添加新的网络命名**

```
]#nmcli connection add type ethernet ifname eth0 con-name eth0 # 设置网卡设备名，nmcli命名的命名为eth0
]#nmcli coonection show
```

**修改IP、子网掩码、网关**

```
]#nmcli connection modify eth0 ipv4.method manual ipv4.addresses ipv4.gateway 192.168.88.200 autoconnect yes

manual:手动配置
autoconnect：开机自动启用
```

**启用**

```
]#nmcil connection up eth0
```

## 远程管理

软件包：openssh

```
]#ssh root@192.168.88.90 # 远程90机器
```

**实现无密码验证**

A机器访问B机器无密码访问

```
]#ssh-keygen # 生成公私钥
]# ls -A /root/.ssh  # 查看公私钥
id_rsa  id_rsa.pub

]# ssh-copy-id root@192.168.88.89 #实现对89机器无密码访问

]# cat /root/.ssh/known_hosts  # 查看远程过设备的信息
192.168.88.89 ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMWDbwKS1zemSAWVqCDmIQ+1R7fcAMeBsV7TSX9JVjm4jtSx9+VFEFmQK2iVyWrmyr9T2v+8lxK/BZaY3RigTgw=
```

## 数据传输

```
]#scp /etc/passwd root@192.168.88.90:/root  # 将paswwd文件传输到90机器root目录下
```

## 临时管理IP

**临时添加IP**

```
]#ip a a 192.168.88.91 dev eth0 
```

**删除临时IP**

```
]#ip a d 192.168.88.91 dev eth0
```

## 日志管理

users、who、w： 查看已登录用户信息

```
]# who
root     tty1         2023-11-18 02:16
root     pts/0        2023-11-18 02:21 (192.168.88.254)
```

last、lastb：查看最近登录成功、失败用户信息

```
]# last
root     pts/0        192.168.88.254   Sat Nov 18 02:21   still logged in

]# lastb
root     tty1                          Sat Nov 18 01:32 - 01:32  (00:00)
root     tty1                          Sat Nov 18 01:32 - 01:32  (00:00)
```

## 系统安全保护

SELinu运行模式：enforcing(强制)、premissive(宽松)、disabled(禁用)

任何模式变成disabled模式都要重启系统

**临时切换运行模式**

1：enforcing

0：permissive

```
]# getenforce  # 查看当前运行模式
Enforcing

]# setenforce 0 #修改当前运行模式
]# getenforce 
Permissive
```

**永久切换**

```
]# setenforce 0 
]#vim /etc/selinux/config
SELINUX=permissive
```

## 破解root密码

**设置为宽松模式时操作**

1. 重启机器，按上下键盘，进入选择系统界面
2. 按e键进入救援模式
3. linux这一行将ro改为rw rd.rebreak
4. 按住ctrl+x启动
5. chroot /sysroot 切换到硬盘操作系统的环境
6. 使用无交互式修改密码 echo a|passwd `--`stdin root
7. reboot -f 强制重启

**未设置宽松模式操作如下**

从第6点开始

```
#toch /.autorelabel # 让SELinux失忆
#reboot -f
或则
#vim /etc/selinux/config # 修改成为宽松模式
SELINUX=permissive
# reboot -f #强制重启
```

**设置救援模式下的密码**

```
]#grub2-setpassword
```

## 构建FTP服务

**安装软件包**

```
]#yum -y install vsftpd
]#vim /etc/vsftpd/vsftpd.conf # 修改为无密码访问
anonymous_enable=YES
```

**运行程序**

```
]#systemctl restart vsftpd
```

**本机访问测试**

```
]# vim /var/ftp/test.txt
]# curl ftp://192.168.88.90
drwxr-xr-x    2 0        0               6 Apr 23  2022 pub
-rw-r--r--    1 0        0              12 Nov 18 08:06 test.tx同
```

## 防火墙策略管理

预设分区：

public：仅允许访问内本机的ssh、dhcp、ping服务

trusted：允许任何访问

block：拒绝任何来访请求，明确拒绝客户端

drop：丢弃任何来访的数据包，不给任何反应

```
]#firewall-cmd --get-default-zone # 查看默认区域
public
]# firewall-cmd --set-default-zone=trusted #修改默认区域为trusted
success

[root@node1 ~]# curl ftp://192.168.88.90 #测试访问
drwxr-xr-x    2 0        0               6 Apr 23  2022 pub
-rw-r--r--    1 0        0              12 Nov 18 08:06 test.txt
```

**public区域添加规则**

```
]# curl http://192.168.88.90 # B机器-测试访问
curl: (7) Failed to connect to 192.168.88.90 port 80: 没有到主机的路由

]# firewall-cmd --zone=public --add-service=http # 允许http访问
success

]# curl http://192.168.88.90 # B机器-测试访问
Test html

]#firewall-cmd --zone=public --list-all # 查询规则信息
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp1s0
  sources: 
  services: cockpit dhcpv6-client http ssh  # 允许访问的服务
  ports: 
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
  
 ]# firewall-cmd --permanent --zone=public --add-service=https # 添加永久https服务访问
success

]# firewall-cmd --reload # 加载防火墙永久策略
success

]# firewall-cmd --zone=public --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp1s0
  sources: 
  services: cockpit dhcpv6-client https ssh
```

**单独拒绝某主机访问**

```
]# firewall-cmd --zone=block --add-source=192.168.88.88
success

]# curl frp://192.168.88.90
curl: (1) Protocol "frp" not supported or disabled in libcurl
```

## 服务管理：程序的运行

1. systemctl start 服务名 #开启服务
2. systemctl restart 服务名 #重启服务
3. systemctl stop 服务名 #停止服务
4. systemctl status 服务名 #查看服务运行状态
5. systemctl enable 服务名 # 设置开机自启动
6. systemctl disable 服务名 # 设置禁止开机自启动
7. systemctl is-enabled 服务名 # 查看服务是否为开机自启动

## WEB服务

**三步骤策略：装包、配置、启服务**

```
]# yum -y install httpd
]# echo Test html > /var/www/html/index.html
]# systemctl restart httpd
]# curl 192.168.88.90
Test html
```

**常见重点配置**

主配置文件：/etc/httpd/comf/httpd.conf

Listen：监听地址：端口

ServerName：站点注册的DNS名称

DoucmentRoot：网页根目录

默认根目录：/var/www/html/

```
]#mkdir /var/www/web
]# echo first one html > /var/www/web/index.html
]# vim /etc/httpd/conf/httpd.conf  # 添加一个站点web
DocumentRoot "/var/www/html"
DocumentRoot "/var/www/web"

]# systemctl restart httpd
]# curl 192.168.88.90
first one html
```

[注]：DocumentRoot谁在后面就覆盖前面路径

DirectoryIndx：起始页/首页文件名

**基与文件目录进行访问控制**

当子目录没有规则，默认继承上一级目录规则

针对此目录有单独配置，则不继承上一级目录

配置文件：/etc/httpd/conf/httpd.conf

```
<Directory />
Require all denied #拒绝所有人访问
</Directory>

<Directory "/var/www">
Require all granted #允许所有人访问
</Directory>
```

**网络路径与实际路径**

客户端-浏览器：http://192.168.88.240 # 网络路径

服务端：/webroot/index.html # 实际路径

http://192.168.88.240  == /webroot

举例：

实际路径：192.168.88.240/webroot/webroot/abc/index.html

网络路径：http://192.168.88.240/webroot/abc/index.html

**调用配置文件**

/etc/httpd/conf/httpd.conf   主配置文件

/etc/httpd/conf.d/*.conf	调用配置文件

作用：避免在主配置文件中修改容易出错

例：以下创建一个test.conf配置文件进行测试，/myweb设置为根站点

```
]# vim /etc/httpd/conf.d/test.conf # 编写文件主目录，设置为允许访问
DocumentRoot /myweb
<Directory /myweb>
 Require all granted
</Directory>

]#mkdir /myweb
]#echo test > /myweb/index.html
]#systemctl restart httpd
]# curl 192.168.88.89
test
```

代码解释：编写test.conf配置文件，用于避免在主配置文件中编写出错，写入新站点/myweb，目录未进行访问控制，需要设置一个访问控制权限

**监听端口**

http默认端口：80

自定义端口时的范围：1024～65535

```
]# vim /etc/httpd/conf.d/test.conf # 添加一个端口8080
DocumentRoot /myweb
Listen 8080 # 端口
<Directory /myweb>
 Require all granted
</Directory>
]#systemctl restart httpd
]# curl 192.168.88.89:8080
test
```

## 虚拟Web主机

作用：由同一台服务器，提供多个不同的Web站点

**区分方式：**

1. 基于域名
2. 基于端口
3. 基于IP地址

**虚拟站点配置**

```
]#vim /etc/httpd/conf.d/test.conf
<VirtualHost *:80> # 监听所有IP地址的80端口
ServerName www.qq.com # 网站的域名
DocumentRoot /var/www/qq # 网页文件路径
</VirtualHost>

]#mkidr /var/www/qq
]#echo wo shi qq > /var/www/qq/index.html
]#systemctl restart httpd
]#vim /etc/hosts # 解析本机域名
192.168.88.240 www.qq.com

]#curl www.qq.com
wo shi qq
```

[注]：一但使用虚拟web主机功能，所有网站都必须使用虚拟Web方式进行呈现

**基于端口虚拟Web主机**

```
[root@server ~]# vim  /etc/httpd/conf.d/xixi.conf
<VirtualHost   *:80>
   ServerName  www.qq.com
   DocumentRoot   /var/www/qq
</VirtualHost>
Listen  8080                   
<VirtualHost   *:8080>
   ServerName   www.qq.com
   DocumentRoot    /var/www/lol
</VirtualHost>
[root@server ~]# systemctl restart httpd
[root@server ~]# curl  www.qq.com:8080
wo shi lol
[root@server ~]# curl  www.qq.com
wo shi qq
```

[注]：端口优先级最高匹配

## NFS服务基础



# 复习问题总结

1. 访问出现测试页面
   + 没有网页文件
   + 网页名称不是index.html
   + httpd的访问控制规则拒绝
   + SELinux没有关闭

# 预习下周内容


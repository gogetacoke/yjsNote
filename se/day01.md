- [学习目标](#学习目标)

- [课堂笔记（命令）](#课堂笔记命令)

- [课堂笔记（文本）](#课堂笔记文本)

- [快捷键](#快捷键)

- [问题](#问题)

- [补充](#补充)

- [今日总结](#今日总结)

- [昨日复习](#昨日复习)

# 学习目标

Web基础应用

NFS服务基础

触发挂载

# 课堂笔记（命令）

# 课堂笔记（文本）

## Httpd常见错误

```
]#systemctl restart httpd
Job for httpd.service failed because the control process exited with error code. See "systemctl status httpd.service" and "journalctl -xe" for details.
]#journalctl -xe  # 查询错误日志
```

提示：执行重启后报错，快速执行 journalctl -xe查看详细信息

## 设置网页存放路径

作用：指定网页存放路径

安装完httpd服务默认存放路径：/var/www/html

```
]#vim /etc/httpd/conf/httpd.conf
修改第122行，并指定目录：快捷方式:/documentroot
DocumentRoot "/var/www/myweb"
]#mkdir /var/www/myweb 
]#echo myweb > /var/www/myweb/index.html #编写测试网页
]#systemctl restart httpd # 重启服务
]#curl 192.168.88.240 # 测试网页
myweb
```

## 网页访问控制规则

### httpd服务默认规则

/var/www/html/    允许所有客户端访问

/   拒绝所有客户端访问

### 访问控制规则

当子目录没有规则，默认继承上一级目录规则

### 单独配置访问控制

/etc/httpd/conf/htttpd.conf

Require all denied #拒绝所有人访问

Require all granted #允许所有人访问

例：将/myweb设置为访问目录，允许所有人访问

```
]#mkdir /myweb  
]#echo xixi > /myweb/index.html
]# vim /etc/httpd/conf/htttpd.conf

DocumentRoot "/myweb"
<Directory /myweb>
Require all granted
</Directory>
]#systemctl restart httpd
]#curl 192.168.88.240
xixi
```

### 网络路径与实际路径

DocumentRoot指定的是网络路径的起始点

客户端：浏览器http://192.168.88.240 #网络路径

服务端：/webroot/index.html #实际路径

http://192.168.88.240  ==  /webroot

```
]# mkdir   /webroot/abc
]# echo wo shi abc > /webroot/abc/index.html
]# curl    192.168.88.240/abc/
```

## 配置文件的使用

–/etc/httpd/conf/httpd.conf #主配置文件

–/etc/httpd/conf.d/*.conf #调用配置文件

```
]# vim   /etc/httpd/conf.d/haha.conf
DocumentRoot    /var/www/cbd
]# mkdir   /var/www/cbd
]# echo wo shi CBD  >  /var/www/cbd/index.html
]# systemctl   restart    httpd
]# curl   192.168.88.240
wo shi CBD 
```

作用：配置内容太多，无需在主配置文件编写，可在conf.d/下创建一个.conf文件，编写详细配置

重启服务即生效    

## 端口监听

端口：数字编号起到标识作用，标识协议或进程

建议自定义端口时大于1024,端口极限65535

```
]#echo Listen 8000 >> /etc/httpd/conf.d/haha.conf
]#systemctl restart httpd
]#curl 192.168.88.240
wo shi CBD 
]#curl 192.168.88.240:8000
wo shi CBD 
```

## 虚拟Web主机

作用：由同一台服务器，提供多个不同的Web站点

### 区分方式

+ 基于域名的虚拟主机
+ 基于端口的虚拟主机
+ 基于IP地址的虚拟主机

### 为虚拟站点添加配置

```
]#vim /etc/httpd/conf.d/xixi.conf
编写内容：
<VirtualHost *：80>
 Servername www.qq.com
 DocumentRoot /var/www/qq
</VirtualHost>
<VirtualHost *：80>
 Servername www.lol.com
 DocumentRoot /var/www/lol
</VirtualHost>

]#vim /etc/hosts  #文件直接解析域名，只为本机解析
编写内容：
192.168.88.240 www.qq.com  www.lol.com
```

### 测试

```
]#mkdir /var/www/qq  /var/www/lol
]#echo i m qq > /var/www/qq/index.html
]#echo i m lol > /var/www/lol/index.html
]#systemctl restart httpd
]#curl www.qq.com
i m qq

]#curl www.lol.com
i m lol
```

### 注意

一但使用虚拟Web主机功能，所有的网站都必须虚拟Web方式进行呈现

## 基于端口的Web主机

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
i m lol
[root@server ~]# curl  www.qq.com
i m qq
```

## NFS服务

作用：为客户机提供共享使用文件夹

协议：NFS（2049）、RPC（111）

软件包：nfs-utils

系统服务：nfs-server

**虚拟机A：服务端**

```
]#mkdir /abc # 创建需要共享的文件
]#echo 123 > /abc/1.txt
]#ls -l /abc
1.txt
]#vim /etc/exports #编写nfs分享规则
/abc *(ro)   #允许所有客户端进行只读访问
]#systemctl restart nfs-server # 启动服务
```

**虚拟机B：客户端**

前提：两台机器网络互通

```
]#mkdir /mnt/mynfs
]#showmount -e 192.168.88.240 #查看对方nfs的分享
]#mount 192.168.88.240:/abc /mnt/mynfs # 挂载
]#df -h /mnt/mynfs # 查看正在挂载设备信息
]#ls -l /mnt/mynfs
1.text
```

扩充：设置开机自动挂载

编写开机自启配置信息：192.168.88.240:/abc /mnt/mynfs nfs _netdev 0 0

_netdev：声明网络设备，系统在具备网络参数后，再进行挂载体设备

## 触发挂载

autofs服务提供“按需访问”

只要访问挂载点就会触发响应，自动挂载指定设备

闲置一段时间后，会自动卸载

```
[root@pc2 ~]# yum  -y  install  autofs
[root@pc2 ~]# systemctl restart autofs
[root@pc2 ~]# ls  /misc
[root@pc2 ~]# ls  /misc/cd   #autofs默认触发挂载cdrom
```

### 自定义修改触发挂载

```
]#vim /etc/auto.master

# Sample auto.master file
# This is a 'master' automounter map and it has the following format:
# mount-point [map-type[,format]:]map [options]
# For details of the format look at auto.master(5).
#
/misc   /etc/auto.misc
/myauto /opt/xixi.txt
]#vim /opt/xixi.txt
nsd -fstype=iso9660 :/dev/cdrom
xxx -fstype=nfs 192.168.88.88:/abc  # 基于nfs触发挂载时编写
]# systemctl restart autofs
```



# 快捷键



# 问题

## 访问出现测试页面：

1. 没有网页文件
2. 网页文件名称不是index.html
3. httpd的访问控制规则拒绝
4. SELinux没有关闭
5. 防火墙

# 补充



# 今日总结



# 昨日复习

- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
  - [podman](#podman)
    - [安装软件包](#安装软件包)
    - [导入镜像](#导入镜像)
    - [删除镜像](#删除镜像)
    - [创建容器](#创建容器)
    - [删除容器](#删除容器)
    - [启停容器](#启停容器)
    - [容器放入后台](#容器放入后台)
    - [容器进入](#容器进入)
    - [容器删除](#容器删除)
    - [容器配置](#容器配置)
    - [制作镜像](#制作镜像)
    - [端口绑定](#端口绑定)
    - [目录绑定](#目录绑定)
- [课堂笔记（文本）](#课堂笔记文本)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)




# 学习目标

Linux容器基础

Liunx容器管理

podman命令行

管理容器进阶

# 课堂笔记（命令）

## podman

### 安装软件包

```sh
]#yum -y install podman
]#yum -y module install container-tools # 安装podman所需的一系列软件
```

### 导入镜像

格式：podman load -i 镜像所在地

前提：已准备好镜像

```sh
]#podman images # 查询系统是否存在镜像
]#podamn load -i /root/imag.tar.xz # 导入镜像
```

### 删除镜像

格式：podman rmi 镜像名+标签/镜像id

```sh
]#podman images
REPOSITORY            TAG         IMAGE ID      CREATED        SIZE
localhost/httpd       latest      608968fe7738  9 months ago   222 MB
localhost/rockylinux  8.6         8cf70153e062  16 months ago  202 MB

]# podman rmi localhost/httpd:latest # 根据镜像名+标签进行删除
Untagged: localhost/httpd:latest
Deleted: 608968fe7738f5d640bc6c38d28a6d3955f3d1e2c9e28dbd39958847d24253f9

]#podman rmi 608968fe7738 # 根据镜像id删除
```

### 创建容器

•podman run 命令的选项

–选项 -i，交互式方式

–选项 -t，终端

–选项 -d，后台运行

–选项 --name 起个容器名字

```sh
]#podman run --name abc01 -it /localhost/rockylinux:8.6 /bin/bash # 创建一个abc01容器
[root@321a3a81a2ee /]# ls # 已经进入容器
bin  etc   lib    lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbi润n  sys  usr

[root@321a3a81a2ee /]# exit # 退出容器
exit

[root@pc2 ~]# podman ps -a #查看正在运行的容器
CONTAINER ID  IMAGE                     COMMAND     CREATED             STATUS                      PORTS       NAMES
321a3a81a2ee  localhost/rockylinux:8.6  /bin/bash   About a minute ago  Exited (127) 5 seconds ago              abc01
```

代码解释：-it创建容器，并进入容器里

### 删除容器

```sh
]#podman rm abc01 # 删除abc01容器
321a3a81a2ee241522e449953df0c504fed61544f3f01cddee7cfe8de185cd4f

]#podman ps -a # 查看全部容器，未启动容器也能查询
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
```

### 启停容器

```sh
]#podman start abc01
]#podman stop abc01 
```

### 容器放入后台

```sh
]#podman run --name abc01 -itd localhost/rocklinux:8.6 # 创建容器abc01放入后台
dcf2855af244f2ea1800de2d9b96ded7ee343766c08e80ab7433388575c9fe65

]#podman ps -a 
CONTAINER ID  IMAGE                     COMMAND     CREATED        STATUS            PORTS       NAMES
dcf2855af244  localhost/rockylinux:8.6  /bin/bash   5 seconds ago  Up 5 seconds ago              abc01
```

### 容器进入

```sh
]#podman exec -it abc01 /bin/bash # 进入容器
[root@dcf2855af244 /]# ls
bin  etc   lib    lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr

[root@dcf2855af244 /]# exit
exit
```

### 容器删除

```sh
[root@pc2 ~]# podman rm -f abc01 # 强制删除容器
dcf2855af244f2ea1800de2d9b96ded7ee343766c08e80ab7433388575c9fe65
```

### 容器配置

前提：本机已搭建FTP服务，已经配置网络YUM

```sh
]#podman run --name abc -itd localhost/rockylinux:8.6 # 后台创建一个abc容器
]#podman exec -it abc /bin/bash # 进入容器
root@fc44c28f1205 /]# rm -rf /etc/yum.repos.d/* # 配置yum仓库
[root@fc44c28f1205 /]# vi /etc/yum.repos.d/dvd.repo
[root@fc44c28f1205 /]# yum repoinfo # 验证仓库
[root@fc44c28f1205 /]# yum -y install net-tools vim
[root@fc44c28f1205 /]# ifconfig
[root@fc44c28f1205 /]# exit
]#podman inspect abc | grep -i ipadd # 查询容器IP地址
```

代码解释：容器创建后，主机会虚拟一张网卡，且ip同容器在同一个网段，方便进行通信

### 制作镜像

前提：将以上制作的容器，封装成镜像

```sh
]#podman stop abc # 关闭正在运行的容器
]#podman commit abc myos:1.0 # 以abc为模板制作一个镜像名为myos标签为1.0
Getting image source signatures
Copying blob 879054335f94 skipped: already exists  

]#podman images # 查看系统所有镜像
REPOSITORY            TAG         IMAGE ID      CREATED         SIZE
localhost/myos        1.0         5cc3f1cbbbaf  12 seconds ago  266 MB
localhost/httpd       latest      608968fe7738  9 months ago    222 MB
localhost/rockylinux  8.6         8cf70153e062  16 months ago   202 MB

]#podman run --name test -it localhost/myos:1.0 /bin/bash # 利用新的镜像生成一个test容器
root@1e7a32e3b0fc /]#ifconfig # 验证容器是否有模板里面的功能
```

### 端口绑定

容器可以与宿主机的端口进行绑定

•从而把宿主机变成对应的服务,不用关心容器的IP地址

•我们使用 -p 参数把容器端口和宿主机端口绑定

•同一宿主机端口只能绑定一个容器服务

```sh
]#podman run --name myweb -p 80:80 -itd localhost/httpd:latest
]#curl 192.168.88.2
Welcome to Apache
```

代码解释：本机没有httpd服务，将本机80端口绑定到容器服务，容器监听本机80端口将触发容器web服务

### 目录绑定

•podman容器不适合保存任何数据

•podman可以映射宿主机文件或目录到容器中

–目标对象不存在就自动创建

–目标对象存在就直接覆盖掉

–多个容器可以映射同一个目标对象来达到数据共享的目的

•启动容器时，使用 -v 映射参数

podman run -itd -v 宿主机对象:容器内对象 镜像名称:标签

```sh
]#mkdir /webroot
]#podman run --name web1 -p 80:80 -v /webroot:/var/www/html/ -itd localhost/httpd:latest #创建虚拟机并指定端口和目录
]#echo haha > /webroot/index.html
]#curl 192.168.88.2
haha
```

# 课堂笔记（文本）



# 快捷键



# 问题

1. 常见的ssh远程管理错误

   删除家目录下的/.ssh/know_hosts文件

2. 容器的ip地址不固定，随着启停容器而改变

# 补充



# 今日总结



# 昨日复习
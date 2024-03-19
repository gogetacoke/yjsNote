- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [本地Yum仓库](#本地yum仓库)
    - [安装createrepo](#安装createrepo)
    - [生成仓库数据文件](#生成仓库数据文件)
    - [编写仓库配置文件](#编写仓库配置文件)
    - [Yum仓库更新](#yum仓库更新)
    - [刷新Yum缓存](#刷新yum缓存)
  - [网络Yum仓库](#网络yum仓库)
    - [搭建ftp服务](#搭建ftp服务)
    - [测试访问](#测试访问)
    - [配置网络仓库](#配置网络仓库)
    - [挂载仓库相互访问](#挂载仓库相互访问)
  - [DNS服务器构建](#dns服务器构建)
    - [安装软件包](#安装软件包)
    - [修改配置文件](#修改配置文件)
    - [建立地址库文件](#建立地址库文件)
    - [测试DNS服务器](#测试dns服务器)
    - [hosts与resolv](#hosts与resolv)
  - [DNS泛域名解析](#dns泛域名解析)
  - [DNS解析记录别名](#dns解析记录别名)
  - [DNS递归与迭代查询](#dns递归与迭代查询)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)



# 学习目标

自定义YUM仓库

网络YUM仓库

DNS服务基础

# 课堂笔记（命令）



# 课堂笔记（文本）

## 本地Yum仓库

前提：需用有众多软件包

### 安装createrepo

```sh
]# createrepo
bash: createrepo: 未找到命令...
安装软件包“createrepo_c”以提供命令“createrepo”？ [N/y] y
# 或
yum -y install createrepo
```

### 生成仓库数据文件

```sh
]#createrepo_c /tools/other  #在当前目录下生成仓库清单文件
Directory walk started
Directory walk done - 5 packages
Temporary output repo path: /tools/other/.repodata/
Preparing sqlite DBs
Pool started (with 5 workers)
Pool finished

]# ls -l /tools/other/
总用量 428
-rwxr-xr-x. 1 root root  67452 4月  19 2018 boxes-1.1.1-4.el7.x86_64.rpm
-rwxr-xr-x. 1 root root  31940 4月  19 2018 cmatrix-1.2a-1.i386.rpm
-rwxr-xr-x. 1 root root 263956 4月  19 2018 ntfs-3g-2014.2.15-6.el6.x86_64.rpm
-rwxr-xr-x. 1 root root  45526 4月  19 2018 oneko-1.2-19.fc24.x86_64.rpm
drwxr-xr-x. 2 root root   4096 11月 16 09:41 repodata
-rwxr-xr-x. 1 root root  14244 4月  19 2018 sl-5.02-1.el7.x86_64.rpm
```

### 编写仓库配置文件

```sh
]#vim /etc/yum.repos.d/mydvd.repo
[myrepm]
baseurl=file:///tools/other
gpgcheck=0

]#yum repoinfo
仓库ID            : myrepm
仓库名称          : myrepm
Repo-revision      : 1700098908
Repo-updated       : 2023年11月16日 星期四 09时41分48秒
Repo-pkgs          : 5
Repo-available-pkgs: 5
Repo-size          : 413 k
Repo-baseurl       : file:///tools/other
Repo-expire        : 172,800 秒 （最近 2023年11月16日 星期四 09时51分49秒）
仓库文件名      : /etc/yum.repos.d/mydvd.repo
```

### Yum仓库更新

```sh
]#mv /tools/other/sl-5.02-1.el7.x86_64.rpm /root #移动仓库中的一个rpm包

]#createrepo_c --update /tools/other # 更新仓库软件包
Directory walk started
Directory walk done - 4 packages
Loaded information about 4 packages
Temporary output repo path: /tools/other/.repodata/
Preparing sqlite DBs
Pool started (with 5 workers)
Pool finished

]#yum repoinfo # 查看仓库信息,此使还是5个包，因为仓库缓存还没有更新
仓库ID            : myrepm
仓库名称          : myrepm
Repo-revision      : 1700098908
Repo-updated       : 2023年11月16日 星期四 09时41分48秒
Repo-pkgs          : 5
Repo-available-pkgs: 5
Repo-size          : 413 k
Repo-baseurl       : file:///tools/other
Repo-expire        : 172,800 秒 （最近 2023年11月16日 星期四 09时51分49秒）
仓库文件名      : /etc/yum.repos.d/mydvd.repo
```

### 刷新Yum仓库缓存

```sh
]#yum makecache # 刷新yum缓存
仓库 'app' 在配置中缺少名称，将使用 id。
仓库 'base' 在配置中缺少名称，将使用 id。
仓库 'myrepm' 在配置中缺少名称，将使用 id。
app                                             4.2 MB/s | 4.3 kB     00:00    
base                                            3.8 MB/s | 3.9 kB     00:00    
myrepm                                          2.9 MB/s | 3.0 kB     00:00    
myrepm                                           88 kB/s | 3.7 kB     00:00    
元数据缓存已建立

]#yum repoinfo #查看仓库信息
仓库ID            : myrepm
仓库名称          : myrepm
Repo-revision      : 1700101488
Repo-updated       : 2023年11月16日 星期四 10时24分48秒
Repo-pkgs          : 4
Repo-available-pkgs: 4
Repo-size          : 399 k
Repo-baseurl       : file:///tools/other
Repo-expire        : 172,800 秒 （最近 2023年11月16日 星期四 10时25分59秒）
仓库文件名      : /etc/yum.repos.d/mydvd.repo
```

代码解释：操作顺序不能改变，本地仓库可拓展

## 网络Yum仓库

### 搭建ftp服务

**A机器**

```sh
]#yum -y install vsftpd
]#vim /etc/vsftpd/vsftpd.conf # 修改无密码访问
anonymous_enable=YES # 将NO改为YES
]#systemctl enable vsftpd #设置开机自启
]#systemctl restart vsftpd 
```

### 测试访问

```sh
]#cp /tools/other /var/ftp/rpms # 将包移动到ftp共享文件夹下
]#curl ftp://192.168.88.240/rpms/
rwxr-xr-x    1 0        0           67452 Nov 16 02:50 boxes-1.1.1-4.el7.x86_64.rpm
-rwxr-xr-x    1 0        0           31940 Nov 16 02:50 cmatrix-1.2a-1.i386.rpm
-rwxr-xr-x    1 0        0          263956 Nov 16 02:50 ntfs-3g-2014.2.15-6.el6.x86_64.rpm
-rwxr-xr-x    1 0        0           45526 Nov 16 02:50 oneko-1.2-19.fc24.x86_64.rpm
drwxr-xr-x    2 0        0            4096 Nov 16 02:50 repodata
-rwxr-xr-x    1 0        0           14244 Nov 16 02:50 sl-5.02-1.el7.x86_64.rpm
```

### 配置网络仓库

**B机器**

```sh
]#vim /etc/yum.repos.d/mydvd.repo # 编写网络仓库配置文件
[myrpms]
baseurl=ftp://192.168.88.240/rpms/
gpgcheck=0
]#yum repoinfo #获取仓库信息
]#yum -y install sl # 测试安包
```

### 挂载仓库相互访问

**A机器**

```sh
]#mkdir /var/ftp/dvd
]#mount /dev/cdrom /var/ftp/dvd # 挂载到ftp下
]#vim /etc/fstab # 编写开机自动挂载
/dev/cdrom /var/ftp/dvd iso9660 defaults 0 0

]#mount -a # 检测挂载情况
]#vim /etc/yum.repos.d/mydvd.repo
[app]
baseurl=ftp://192.168.88.240/dvd/AppStream
gpgcheck=0
[base]
baseurl=ftp://192.168.88.240/dvd/BaseOS
gpgcheck=0
[myrepm]
baseurl=ftp://192.168.88.240/rpms
gpgcheck=0


]#curl ftp://192.168.88.240/dvd/
dr-xr-xr-x    4 0        0            2048 May 15  2022 AppStream
dr-xr-xr-x    4 0        0            2048 May 15  2022 BaseOS
dr-xr-xr-x    3 0        0            2048 May 15  2022 EFI
-r--r--r--    1 0        0            2204 Mar 30  2022 LICENSE
-r--r--r--    1 0        0             883 May 15  2022 TRANS.TBL
dr-xr-xr-x    3 0        0            2048 May 15  2022 images
dr-xr-xr-x    2 0        0            2048 May 15  2022 isolinux
-r--r--r--    1 0        0              86 May 15  2022 media.repo

]#scp /etc/yum.repos.d/mydvd.repo  root@192.168.88.2:/etc/yum.repos.d/mydvd.repo # 使用scp将配置发送B机器
```

**B机器**

```sh
]#yum repoinfo # 测试仓库
```

## DNS服务器构建

DNS服务器的功能：争相解析：根据注册的域名查找器对应的IP地址

DNS服务器分类：根域名服务器、一级DNS服务器、二级DNS服务器、三级DNS服务器

域名系统：所有的域名都必须要以点作为结尾，树形结构

### 安装软件包

```sh
]# yum  -y  install   bind    bind-chroot # bind 主程序、bind-chroot 提供牢笼政策
```

### 修改配置文件

```sh
]#cp -p /etc/named.conf /root # 备份数据    cp -p ：保留数据原有者
]#vim /etc/named.conf
options{
	directory ""/var/named"; # 定义地址库文件存放路径
};
zone “tedu.cn” IN{ 	# 定义负责解析tedu.cn域名
	type master;	# 主DNS服务器
	file "tedu.cn.zone"; # 地址库文件名称
};
```

### 建立地址库文件

```sh
]#cp -p /var/named/named.localhost /var/named/tedu.cn.zone
]#vim /var/named/named.localhost
NS			server  #声明DNS服务器为server
server		A		192.168.88.240
www			A		1.1.1.1
ftp			A		2.2.2.2
]#systemctl restart named
```

### 测试DNS服务器

**本机测试**

```sh
]#nslookup www.tedu.cn
Server:         192.168.88.240
Address:        192.168.88.240#53

Name:   www.tedu.cn
Address: 1.1.1.1
```

**B机器**

```sh
]#echo nameserver 192.168.88.240 > /etc/resolv.conf # 指定DNS服务器地址
]#nslookup www.tedu.cn # 命令测试域名解析
Server:         127.0.0.1
Address:        127.0.0.1#53

Name:   www.tedu.cn
Address: 1.1.1.1
```

### hosts与resolv

> hosts：文件域名解析最高优先级
>
> resolv.conf：文件指定DNS服务器地址

## DNS泛域名解析

```sh
]#vim /var/named/tedu.cn.zone
NS			server  #声明DNS服务器为server
server		A		192.168.88.240
www			A		1.1.1.1
ftp			A		2.2.2.2
*			A		7.7.7.7
]#systemctl restart named
```

**B机器**

```sh
]#nslookup  123.tedu.cn
# nslookup 123.tedu.cn
Server:         192.168.88.240
Address:        192.168.88.240#53

Name:   123.tedu.cn
Address: 7.7.7.7
```

## DNS解析记录别名

```sh
]#vim /var/named/tedu.cn.zone
NS			server  #声明DNS服务器为server
server		A		192.168.88.240
www			A		1.1.1.1
ftp			A		2.2.2.2
*			A		7.7.7.7
vs			CNAME	ftp
]#systemctl restart named
```

**B机器**

```sh
]#nslookup  vs.tedu.cn
# nslookup vs.tedu.cn
Server:         192.168.88.240
Address:        192.168.88.240#53

Name:   vs.tedu.cn
Address: 2.2.2.2
```

## DNS递归与迭代查询

递归查询：客户端发送请求给首选DNS服务器，首选DNS服务器与其他的DNS服务器交流，最终将结果带回来过程

迭代查询：客户端发送请求给首选的DNS服务器，首选DNS服务器告知下一个DNS服务器地址

# 快捷键



# 问题

1. DNS地址记录中类型NS、A的含义

   > NS记录为域名服务器记录
   >
   > A记录为正向解析记录

2. DNS常见的资源类型

   > A记录
   >
   > MX记录
   >
   > CNAME记录
   >
   > NS记录

# 补充

1. cp -p 

   > 保持权限不变进行复制

# 今日总结



# 昨日复习
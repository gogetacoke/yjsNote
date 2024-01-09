- [项目名](#项目名)
- [项目介绍](#项目介绍)
- [项目架构图](#项目架构图)
  - [单击架构图](#单击架构图)
  - [单机-数据库分离架构图](#单机-数据库分离架构图)
  - [集群架构图](#集群架构图)
- [技术选型](#技术选型)
- [环境准备](#环境准备)
  - [配置环境](#配置环境)
    - [配置ansible环境](#配置ansible环境)
    - [配置yum](#配置yum)
- [单击搭建](#单击搭建)
  - [安装软件](#安装软件)
  - [编写测试页面](#编写测试页面)
    - [测试访问](#测试访问)
  - [数据库配置](#数据库配置)
    - [测试访问](#测试访问-1)
  - [部署wordpress](#部署wordpress)
    - [解压缩包](#解压缩包)
    - [修改权限](#修改权限)
    - [页面初始化](#页面初始化)
- [单机-数据库分离搭建](#单机-数据库分离搭建)
  - [安装软件](#安装软件-1)
  - [导出数据库](#导出数据库)
  - [测试数据库](#测试数据库)
  - [修改配置文件](#修改配置文件)
  - [测试网页](#测试网页)
  - [配置额外的web服务器](#配置额外的web服务器)
  - [测试网页](#测试网页-1)
- [集群搭建](#集群搭建)
  - [NFS搭建](#nfs搭建)
    - [概述](#概述)
    - [验证](#验证)
    - [配置NFS服务器](#配置nfs服务器)
    - [迁移文件至nfs](#迁移文件至nfs)
  - [配置代理服务器](#配置代理服务器)
    - [调度器配置](#调度器配置)
    - [高可用配置](#高可用配置)
    - [高可用主备切](#高可用主备切)
    - [服务启动](#服务启动)
    - [验证](#验证-1)
  - [配置名称解析](#配置名称解析)
- [知识总结](#知识总结)


# 项目名

> 基于LNMP结构实现高可用wordpress项目搭建

# 项目介绍

> ​	WordPress是一个功能强大的内容管理系统（CMS），用于构建网站。它具有灵活性和可扩展性，其中最重要的是它是开源软件。这意味着有一个庞大的社区不断为其开发做出贡献，并创建工具为用户提供额外功能。WordPress的强大之处在于它的潜在学习曲线。虽然WordPress的可能性令人惊叹，但也可能令人不知所措。有很多东西需要学习和探索！好消息是，一个网站不需要非常复杂才能有效。通常情况下，简单就是更好。更好的消息是，WordPress非常适合创建简单的网站，将学习曲线降低到更容易掌握的水平。从这里开始，您可以一步步地继续学习和发展您的网站。
>
> [访问官网下载](https://cn.wordpress.org/download/)
>
> 同类型项目
>
> [Discuz论坛项目](https://www.discuz.vip/)

# 项目架构图

## 单击架构图

![](../../pic/pro/d1-1.png)

## 单机-数据库分离架构图

![](../../pic/pro/d1-2.png)

## 集群架构图

![](../../pic/pro/d1-3.png)

# 技术选型

> web服务器：nginx
>
> 数据库：mysql-server
>
> 文件存储：nfs
>
> php依赖：php-fpm、php-mysqlnd、php-json
>
> ansible剧本：ansible

# 环境准备

![](../../pic/pro/d1-4.png)

> 1. 创建虚拟机、配置防火墙、SELINUX、主机名、IP地址、yum
>
> 2. 下载wordpress安装包
>
>    [点击进行下载](https://cn.wordpress.org/download/)
>
> 3. 准备一台主机用于ansible自动化管理其余服务器

## 配置环境

### 配置ansible环境

```sh
# 配置ansible
[root@pubserver ~]#yum -y install ansible
# 创建工作目录
[root@pubserver ~]#mkdir -p project01/files
[root@pubserver ~]# cd project01/
# 创建主配置文件、主机清单文件、yum配置文件
[root@pubserver project01]# vim ansible.cfg 
[defaults]
inventory = inventory
host_key_checking = false
[root@pubserver project01]#vim inventory
[webservers]
web1 ansible_host=192.168.88.11
web2 ansible_host=192.168.88.12
web3 ansible_host=192.168.88.13

[databases]
database1 ansible_host=192.168.88.21

[nfss]
nfs1 ansible_host=192.168.88.31

[haproxys]
haproxy01 ansible_host=192.168.88.5
haproxy02 ansible_host=192.168.88.6

[all:vars]
ansible_ssh_user=root
ansible_ssh_pass=a
# 测试主机
[root@pubserver project01]#ansible all -m ping
```

### 配置yum

```sh
[root@pubserver project01]# vim files/local88.repo 
[BaseOS]
name = BaseOS
baseurl = ftp://192.168.88.240/dvd/BaseOS
enabled = 1
gpgcheck = 0

[AppStream]
name = AppStream
baseurl = ftp://192.168.88.240/dvd/AppStream
enabled = 1
gpgcheck = 0

[rpms]
name = rpms
baseurl = ftp://192.168.88.240/rpms
enabled = 1
gpgcheck = 0

# 上传yum到各台主机
[root@pubserver project01]# vim 01-upload-repo.yml 
---
- name: config repos.d
  hosts: all
  tasks:
    - name: delete repos.d
      file:
        path: /etc/yum.repos.d
        state: absent

    - name: create repos.d
      file:
        path: /etc/yum.repos.d
        state: directory
        mode: '0755'

    - name: upload local88
      copy:
        src: files/local88.repo
        dest: /etc/yum.repos.d/

[root@pubserver project01]# ansible-playbook 01-upload-repo.yml 
```

# 单击搭建

## 安装软件

```sh
# 使用ansible剧本批量安装及启动
[root@pubserver project01]# vim 02-config-web1.yml 
---
- name: config web1
  hosts: web1
  tasks:
    - name: install pkgs   # 安装软件包
      yum:
        name:
          - nginx
          - mysql-server
          - php-mysqlnd
          - php-fpm
          - php-json
        state: present

    - name: start service   # 循环启动多个服务
      service:
        name: "{{item}}"
        state: started
        enabled: yes
      loop:
        - nginx
        - php-fpm
        - mysqld
[root@pubserver project01]# ansible-playbook 02-config-web1.yml
```

## 编写测试页面

```sh
[root@web1 ~]# vim /usr/share/nginx/html/index.php # 获取php版本信息
<?php
    phpinfo();
?>
```

### 测试访问

> 浏览器：http://192.168.88.11

![](../../pic/pro/d1-5.png)

## 数据库配置

```sh
# 编写一个脚本创建数据库wordpress，用户yyh，密码123456，拥有wordpress数据库全部权限
[root@pubserver project01]# vim files/config_mysql.sh
#!/bin/bash

mysql -e "create database wordpress character set utf8mb4"
mysql -e "create user yyh@localhost identified by '123456'"
mysql -e "grant all privileges on wordpress.* to yyh@localhost"

[root@pubserver project01]#chmod +x files/config_mysql.sh
[root@pubserver project01]# vim 03-config-mysql.yml 
---
- name: config mysql
  hosts: web1
  tasks:
    - name: create database
      script: files/config_mysql.sh

[root@pubserver project01]# ansible-playbook 03-config-mysql.yml
```

```sh
# mysql -e
[root@database1 ~]#mysql -e "select version()"
+-----------+
| version() |
+-----------+
| 8.0.26    |
+-----------+
```

> "mysql -e" 是 MySQL 命令行工具的一部分，用于执行 SQL 查询或命令。其中，"-e" 选项表示后面跟随的是要执行的 SQL 语句或命令。通过这种方式，可以在命令行中直接执行 MySQL 查询，而无需打开交互式 MySQL Shell。
>
> 例如，在命令行中输入 "mysql -e 'SELECT * FROM users'"，将会执行一个查询语句，从名为 "users" 的表中选择所有记录。结果将直接在命令行中返回。
>
> 请注意，执行 "mysql -e" 命令需要正确安装和配置 MySQL，并具有足够的权限来执行查询或命令。

### 测试访问

```sh
[root@web1 ~]#mysql -uyyh -p123456 wordpress # 能正常进入则表示配置正确
```

## 部署wordpress

### 解压缩包

```sh
[root@web1 ~]# tar xf wordpress-6.1.1-zh_CN.tar.gz  
[root@web1 ~]# cp -r wordpress/* /usr/share/nginx/html/
```

### 修改权限

```sh
# php程序是由php-fpm处理的，php-fpm要以apache身份运行（不是必须；也可以选择为PHP-FPM指定其他用户和组。对于安全性而言，强烈建议不要用root用户来运行PHP-FPM）
[root@web1 ~]#ps -aux|grep php-fpm # 可以查询到全是以apache用户在运行
[root@web1 ~]#ll /usr/share/nginx/html/ # 可以看到全是root root
# 为了让php-fpm程序能对html目录进行读写操作，需要为他授予权限
[root@web1 ~]#chown -R apache:apache /usr/share/nginx/html/
[root@web1 ~]#ll /usr/share/nginx/html/ # 全是apache apache
```

> 代码解释：
>
> ​	nginx使用php的页面需要使用一个中间件FastCGI，用于nginx与后端服务php通信，所以需要安装一个php-fpm，运行成功后会创建一个apache用户，通过ps -aux`|`ep php-fpm查询到该进程的用户是apache，所以使用php-fpm必须要apache用户，但也不是必须,以在/etc/php-fpm.d/ww.conf文件第55行添加acl用户；
>
> ​	通过ll /usr/share/nginx/html 查看初始nginx的文件属主与属组都是root，但是不建议使用root进行数据读写
>
> ​	使用chown -R apache :apache /usr/share/nginx/html 给该目录递归设置所属主及所属组为apache

### 页面初始化

> 浏览器访问：http://192.168.88.11
>
> 根据提示进行初始化，数据库名、数据库账密、站点名、账密
>
> 登陆

![](../../pic/pro/d1-6.png)

# 单机-数据库分离搭建

> 基于单机搭建进行

## 安装软件

```sh
[root@pubserver project01]# vim files/config_mysql2.sh
#!/bin/bash
  
mysql -e "create database wordpress character set utf8mb4"
mysql -e "create user yyh@'%' identified by '123456'"
mysql -e "grant all privileges on wordpress.* to yyh@'%'"

[root@pubserver project01]# chmod +x files/config_mysql2.sh
[root@pubserver project01]# vim 04-config-database.yml
---
- name: config database
  hosts: databases
  tasks:
    - name: install mysql    # 安装数据库服务
      yum:
        name: mysql-server
        state: present

    - name: start service    # 启动数据库服务
      service:
        name: mysqld
        state: started
        enabled: yes

    - name: create database user
      script: files/config_mysql2.sh

[root@pubserver project01]# ansible-playbook 04-config-database.yml
```

## 导出数据库

```sh
# 备份数据库wordpress中的数据到wordpress.sql文件
[root@web1 ~]# mysqldump wordpress > wordpress.sql
# 将备份文件拷贝到新数据库服务器
[root@web1 ~]# scp wordpress.sql 192.168.88.21:/root/
```

> `mysqldump`是一个MySQL数据库命令行工具，用于备份和导出数据库的结构和数据。它可以生成包含SQL语句的文本文件，这些语句可以用于恢复（导入）数据库或将数据库迁移到其他环境

## 测试数据库

```sh
[root@database1 ~]#ls 
wordpress.sql
[root@database1 ~]#mysql -uyyh -p123456 wordpress
mysql>show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| wordpress          |
+--------------------+
2 rows in set (0.00 sec)
mysql>quit

# 将wordpress.sql中的数据导入到wordpress数据库中
[root@database1 ~]#mysql wordpress < wordpress.sql
[root@database1 ~]# mysql -e "show tables" wordpress # 有数据就没问题
+-----------------------+
| Tables_in_wordpress   |
+-----------------------+
| wp_commentmeta        |
| wp_comments           |
| wp_links              |
| wp_options            |
| wp_postmeta           |
| wp_posts              |
| wp_term_relationships |
| wp_term_taxonomy      |
| wp_termmeta           |
| wp_terms              |
| wp_usermeta           |
| wp_users              |
+-----------------------+
```

## 修改配置文件

```sh
[root@web1 ~]# vim +32 /usr/share/nginx/html/wp-config.php 
...略...
 31 /** Database hostname */
 32 define( 'DB_HOST', '192.168.88.21' ); # 指定数据库地址为数据库服务器IP
...略...

[root@web1 ~]# systemctl stop mysqld
[root@web1 ~]# yum remove -y mysql-server # 卸载本机数据库，将使用数据库服务器的数据库
```

## 测试网页

> http://192.168.88.11
>
> 还能正常访问表示成功

## 配置额外的web服务器

> + 部署在多台web服务器上，搭建集群前期
> + 共享数据库

```sh
[root@pubserver project01]# vim 05-config-webservers.yml 
---
- name: config webservers
  hosts: webservers
  tasks:
    - name: install pkgs    # 安装软件包
      yum:
        name:
          - nginx
          - php-mysqlnd
          - php-fpm
          - php-json
        state: present

    - name: start service   # 循环启动多个服务
      service:
        name: "{{item}}"
        state: started
        enabled: yes
      loop:
        - nginx
        - php-fpm
 
 [root@pubserver project01]# ansible-playbook 05-config-webservers.yml
```

```sh
[root@pubserver project01]# vim 06-fetch-web1.yml 
---
- name: copy web
  hosts: web1
  tasks:
    - name: compress html    # 压缩html目录到/root下
      archive:
        path: /usr/share/nginx/html # 需要压缩的目录
        dest: /root/html.tar.gz # 压缩包类型及压缩位置
        format: gz # 压缩格式

    - name: download html    # 下载压缩文件
      fetch:
        src: /root/html.tar.gz 
        dest: files/
        flat: yes

[root@pubserver project01]# ansible-playbook 06-copy-web.yml 
```

```sh
[root@pubserver project01]# vim 07-deploy-web23.yml
---
- name: deploy web2 and web3
  hosts: web2,web3
  tasks:
    - name: unarchive to web    # 解压文件到指定位置
      unarchive:
        src: files/html.tar.gz # 压缩包所在地
        dest: /usr/share/nginx/ # 解压缩的地方

[root@pubserver project01]# ansible-playbook 07-deploy-web23.yml
```

> archive和unarchive是ansible中压缩归档模块

## 测试网页

> - 访问http://192.168.88.12/和http://192.168.88.13/将会得到与http://192.168.88.11/相同的页面

# 集群搭建

## NFS搭建

### 概述

> + 程序将文字数据保存到数据库中
> + 程序非文字数据(如图片、视频、压缩包等)保存到相应的文件目录中
> + 图片太多非常占空间，所以将它隔离出来，专门放在一个存储服务器上

### 验证

> 用户发布带有文字和图片的文章，验证文字与图片的存储信息

```sh
[root@database ~]# mysql
mysql> use wordpress;
mysql> select * from wp_posts\G

[root@web1 html]# ls /usr/share/nginx/html/wp-content/uploads/2024/01/
image.jpg
```

### 配置NFS服务器

```sh
[root@pubserver project01]# vim 08-config-nfs.yml
---
- name: config nfs
  hosts: nfs
  tasks:
    - name: install nfs        # 安装nfs
      yum:
        name: nfs-utils
        state: present

    - name: mkdir /nfs_root    # 创建共享目录
      file:
        path: /nfs_root
        state: directory
        mode: "0755"

    - name: nfs share          # 修改配置文件
      lineinfile:
        path: /etc/exports
        line: '/nfs_root 192.168.88.0/24(rw)' # 使88网段下都有读写权限访问此nfs贡献文件夹

    - name: start service      # 循环启动服务
      service:
        name: "{{item}}"
        state: started
        enabled: yes
      loop:
        - rpcbind       # nfs服务依赖rpcbind服务
        - nfs-server

[root@pubserver project01]# ansible-playbook 08-config-nfs.yml

#  查看共享输出
[root@nfs ~]# showmount -e
Export list for nfs:
/nfs_root 192.168.88.0/24
```

> 为什么启动nfs服务要启动rpcbind？
>
> ​	nfs服务开启rpcbind是指在Linux系统中使用NFS（Network File System）时，需要启动rpcbind服务。rpcbind是一个系统服务，它负责将RPC（Remote Procedure Call）程序号映射到网络端口号。NFS服务器和客户端都需要rpcbind服务来进行通信。在Debian Jessie系统中，rpcbind服务需要在系统启动时自动启动，以确保NFS服务的正常运行。如果rpcbind服务未能正确启动，可能会导致NFS服务无法正常工作

### 迁移文件至nfs

```sh
# 1. 重新下载web1的html目录
[root@pubserver project01]# cp 06-fetch-web1.yml 09-fetch-web1.yml
---
- name: copy web
  hosts: web1
  tasks:
    - name: compress html        # 压缩html目录到/root下
      archive:
        path: /usr/share/nginx/html
        dest: /root/html2.tar.gz
        format: gz

    - name: download html
      fetch:
        src: /root/html2.tar.gz  # 下载压缩文件
        dest: files/
        flat: yes

[root@pubserver project01]# ansible-playbook 09-fetch-web1.yml 

# 2. 释放压缩包到nfs服务器
[root@pubserver project01]# vim 10-deploy-nfs.yml 
---
- name: deploy nfs
  hosts: nfs
  tasks:
    - name: unarchive to web     # 将控制端压缩文件解压到指定位置
      unarchive:
        src: files/html2.tar.gz
        dest: /nfs_root/

[root@pubserver project01]# ansible-playbook 10-deploy-nfs.yml 

# 3. 清除web服务器的html目录
[root@pubserver project01]# vim 11-rm-html.yml
---
- name: rm html
  hosts: webservers
  tasks:
    - name: rm html
      file:
        path: /usr/share/nginx/html
        state: absent
        
    - name: create html
      file:
        path: /usr/share/nginx/html
        state: directory
        owner: apache
        group: apache
        mode: "0755"

[root@pubserver project01]# ansible-playbook 11-rm-html.yml

# 4. 挂载nfs到web服务器
[root@pubserver project01]# vim 12-mount-nfs.yml
---
- name: mount nfs
  hosts: webservers
  tasks:
    - name: install nfs
      yum:
        name: nfs-utils
        state: present
        
    - name: mount nfs
      mount:
        path: /usr/share/nginx/html
        src: 192.168.88.31:/nfs_root/html
        fstype: nfs
        state: mounted

[root@pubserver project01]# ansible-playbook 12-mount-nfs.yml
```

```sh
# 经测试，虽然设置了开机自启动，当重启后未能挂载上，还需手动mount -a 才能挂载上
[root@pubserver project01]# ansible webservers -m lineinfile "path=/etc/rc.d/rc.local line='mount -a'"
# 在/etc/rc.d/rc.local的文件中编写mount -a使系统启动成功后，再自动运行该命令
```

## 配置代理服务器

> 配置高可用(keepalived)、负载均衡(haproxy)功能

```sh
[root@pubserver project01]# vim 13-install-lb.yml 
---
- name: install lb
  hosts: lb
  tasks:
    - name: install pkg
      yum:
        name: haproxy,keepalived
        state: present

[root@pubserver project01]# ansible-playbook 13-install-lb.yml 
```

### 调度器配置

```sh
# 修改配置文件并启动服务
[root@pubserver project01]# vim 14-config-lb.yml
---
- name: config haproxy
  hosts: lb
  tasks:
    - name: rm lines
      shell: sed -i '64,$d' /etc/haproxy/haproxy.cfg

    - name: add lines
      blockinfile:
        path: /etc/haproxy/haproxy.cfg
        block: |
          listen wordpress
              bind 0.0.0.0:80
              balance roundrobin
              server web1 192.168.88.11:80 check inter 2000 rise 2 fall 5
              server web2 192.168.88.12:80 check inter 2000 rise 2 fall 5
              server web3 192.168.88.13:80 check inter 2000 rise 2 fall 5

          listen mon   # 配置一个监控页面
            bind 0.0.0.0:1080
            stats refresh 30s
            stats uri /mon
            stats auth admin:admin

    - name: start service
      service:
        name: haproxy
        state: started
        enabled: yes

[root@pubserver project01]# ansible-playbook 14-config-lb.yml
```

### 高可用配置

```sh
# haproxy1配置keepalived，实现高可用集群
[root@haproxy1 ~]# vim /etc/keepalived/keepalived.conf 
...略...
 12    router_id haproxy1   # 为本机取一个唯一的id
 13    vrrp_iptables        # 自动开启iptables放行规则
...略...
 20 vrrp_instance VI_1 {
 21     state MASTER        # 主服务器状态是MASTER
 22     interface eth0
 23     virtual_router_id 51
 24     priority 100
 25     advert_int 1
 26     authentication {
 27         auth_type PASS
 28         auth_pass 1111
 29     }
 30     virtual_ipaddress {
 31         192.168.88.80       # vip地址
 32     }
 33 }
 34 # 以下全部删除

# haproxy2配置keepalived
[root@haproxy1 ~]# scp /etc/keepalived/keepalived.conf 192.168.88.6:/etc/keepalived/
[root@haproxy2 ~]# vim /etc/keepalived/keepalived.conf 
...略...
 12    router_id haproxy2   # 为本机取一个唯一的id
 13    vrrp_iptables        # 自动开启iptables放行规则
...略...
 20 vrrp_instance VI_1 {
 21     state BACKUP        # 备份服务器状态是BACKUP
 22     interface eth0
 23     virtual_router_id 51
 24     priority 80         # 备份服务器优先级低于主服务器
 25     advert_int 1
 26     authentication {
 27         auth_type PASS
 28         auth_pass 1111
 29     }
 30     virtual_ipaddress {
 31         192.168.88.80
 32     }
 33 }
```

### 高可用主备切

> 当haproxy服务挂掉后，主备不会切换，因为keepalived在上述配置中只是启用VIP作用，但它并不知道当前运行了那些服务，由于haproxy绑定端口为80，写一个脚本来进行判断，0代表存活，1代表失败，keepalived中可以使用该脚本

```sh
# 编写脚本
[root@haproxy1 ~]#vim /etc/keepalived/check_http.sh
#!/bin/bash
ss -ntulp|grep :80 &> /dev/null && exit 0 || exit 1
[root@haproxy1 ~]#chmod +x /etc/keepalived/check_http.sh
[root@haproxy1 ~]#vim /etc/keepalived/keepalived.conf
 19 vrrp_script  check_80{  
 20         script "/etc/keepalived/check_80.sh" # 指定脚本位置
 21         interval 2 # 每2s运行一次
 22 }
 .......
 36     track_script{ # 调用脚本
 37          check_80
 38 }
 39 }
```

### 服务启动

```sh
# 启动服务
[root@haproxy1 ~]# systemctl enable keepalived.service --now
[root@haproxy2 ~]# systemctl enable keepalived.service --now
```

### 验证

```sh
# haproxy1上出现VIP。客户端访问http://192.168.88.80即可
[root@haproxy1 ~]# ip a s | grep 192
    inet 192.168.88.5/24 brd 192.168.88.255 scope global noprefixroute eth0
    inet 192.168.88.80/32 scope global eth0
```

## 配置名称解析

> 通过本机hosts文件名词实现名称解析

```sh
[root@myhost ~]# echo "192.168.88.80 www.danei.com" >> /etc/hosts
```

> - 如果客户端是windows主机，则使用记事本程序打开`C:\windows\System32\drivers\etc\hosts`添加名称解析

> - 当点击[http://www.danei.com](http://www.danei.com/)页面中任意链接时，地址栏上的地址，都会变成`192.168.88.11`。通过以下方式修复它：

```sh
# 在nfs服务器上修改配置文件
[root@nfs ~]# vim /nfs_root/html/wp-config.php 
# define('DB_NAME', 'wordpress')它的上方添加以下两行：
define('WP_SITEURL', 'http://www.danei.com'); 
define('WP_HOME', 'http://www.danei.com');
```

# 知识总结

1. php-fpm的使用者是apache，可通过ps -aux|grep php-fpm查询

2. nginx配合使用php-fpm时，需要指定站点根目录的所属主和属组为apache（因为都为root，防止恶意攻击）

3. 使用数据库时，创建一个账号给予该数据库的某一个库的所有权限，避免直接使用root用户

4. 用户指向数据库时，如果时本地都写：localhost，如果时其他主机写：'%'

   ```sql
   create database database character set utf8mb4; # 创建数据库
   create user user@'localhost' identified by 'passwd'; # 创建用户
   grant all privileges on database.* to user@'localhost'; # 用户给予数据库权限
   ```

5. 命令行执行mysql语句

   ```shell
   [root@database1 ~]# mysql -e 'select version()' 
   +-----------+
   | version() |
   +-----------+
   | 8.0.26    |
   +-----------+
   ```

6. 数据库导入导出

   ```sh
   # 导出
   mysqldump databases > databases.sql
   # 导入
   mysql databases < databases.sql
   ```

7. 压缩、解压缩模块

   ```sh
   archive模块用于创建归档文件，其参数包括：
   
   path：要归档的文件或目录的路径。
   dest：归档文件的目标路径。
   format：归档文件的格式，可以是tar、tar.gz、tar.bz2、tar.xz、zip等。
   ----------------------------------------------------------
   unarchive模块的参数包括：
   
   src：要解压缩的归档文件的路径，可以是本地路径或远程URL。
   dest：解压缩后的文件存放的目标路径。
   ```

8. nfs服务

   ```sh
   # nfs服务端与客户端都需安装nfs-utils服务
   # 服务端服务配置：
   1. 创建共享文件夹 /nfs_root
   2. 修改配置文件：/etc/exports
   3. 添加配置信息：/nfs_root  xxx.xxx.xxx(rw)
   4. 启动服务
   systemctl start nfs-server
   systemctl start rpc-bind
   # 客户端配置
   1. 使用mout进行挂载到文件夹
   2. df -h 查看挂载情况
   ```

9. 理解负载均衡与高可用的含义

   ```sh
   # ha代理解释
   server web1 192.168.88.11:80 check inter 2000 rise 2 fall 5
   # 代理后端服务80端口
   # check 开启健康检查
   # inter 2000 每2000毫秒做一次检查
   # rise 2 连续两次成功则标记服务存活
   # fall 5 连续五次错误则标记服务down掉
   ```

10. harpoxy服务挂掉后，vip未切换

    ```sh
    # keepalived只是提供vip，但并不知道运行了那些服务，而ha绑定的端口也是80，服务也是80端口
    # 解决办法，编写脚本监听80端口是否存在，存在返回0，不存在返回1，来进行vip转移
    [root@lb1 ~]#vim /etc/keepalived/http_check.sh
    #!/bin/bash
ss -ntulp|grep :80 &> /dev/null && exit 1 || exit 0
    [root@lb1 ~]#chmod +x /etc/keepalived/http_check.sh
    [root@haproxy1 ~]#vim /etc/keepalived/keepalived.conf
     19 vrrp_script  check_80{  
     20         script "/etc/keepalived/check_80.sh" # 指定脚本位置
     21         interval 2 # 每2s运行一次
     22 }
     .......
     36     track_script{ # 调用脚本
     37         check_80
     38 }
     39 }
    ```
    
    






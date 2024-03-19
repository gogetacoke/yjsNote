- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [数据库服务部署](#数据库服务部署)
    - [MySQL安装](#mysql安装)
    - [Rides安装](#rides安装)
  - [Jenkins全局工具配置](#jenkins全局工具配置)
    - [配置JDK环境](#配置jdk环境)
    - [配置Git工具](#配置git工具)
    - [配置Maven工具](#配置maven工具)
    - [配置邮件工具](#配置邮件工具)
  - [Jenkins后端工程构建](#jenkins后端工程构建)
    - [Jenkins创建HIS-BACKEND工程](#jenkins创建his-backend工程)
    - [测试HIS-BACKEND工程构建](#测试his-backend工程构建)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)


# 学习目标

HIS项目数据库服务

Jenkins构建HIS项目后端工程

CI/CD全流程测试

# 课堂笔记（命令）

# 课堂笔记（文本）

## 数据库服务部署

### MySQL安装

> 准备好测试数据

```perl
[root@services ~]# yum -y install mysql-server mysql
[root@services ~]# systemctl enable mysqld --now
# 设置root密码
[root@services ~]# mysqladmin -uroot -p password "123456"
# 创建业务库及账号密码
[root@services ~]#mysql -uroot -p123456
mysql> CREATE DATABASE his;                                         #创建his库
mysql> CREATE USER 'his'@'192.168.88.60' IDENTIFIED BY 'hisadmin';  #创建his用户
mysql> GRANT ALL ON his.* TO 'his'@'192.168.88.60';                 #授权his用户
[root@services ~]#mysql -uroot -p123456 his < his.sql
```

### Rides安装

```perl
[root@services ~]#yum -y install redis
# 修改配置文件
[root@services ~]# vim /etc/redis.conf 
[root@services ~]# sed -n "69p;88p;136p;507p" /etc/redis.conf 
bind 0.0.0.0 # 监听本地所有网络
protected-mode no # 关闭保护模式
daemonize yes # 开启守护进程
requirepass hisadmin # 设置密码
# 启动验证
[root@services ~]#systemctl enable redis --now
[root@services ~]#redis-cli -h 127.0.0.1:6379> -p 6379 -a hisadmin
127.0.0.1:6379>ping
PONG
```

## Jenkins全局工具配置

> 进入部署在192.168.88.30机器上的Jenkins进行配置
>
> 浏览器进入192.168.88.30:8080

### 配置JDK环境



### 配置Git工具

### 配置Maven工具

### 配置邮件工具

## Jenkins后端工程构建

### Jenkins创建HIS-BACKEND工程

### 测试HIS-BACKEND工程构建



# 问题



# 补充



# 今日总结




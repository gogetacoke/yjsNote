- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [Redis服务部署](#redis服务部署)
    - [Redis介绍](#redis介绍)
    - [安装redis](#安装redis)
    - [基本操作](#基本操作)
    - [修改配置](#修改配置)
  - [Redis常用命令](#redis常用命令)
    - [mset、mget](#msetmget)
    - [keys、type](#keystype)
    - [exists](#exists)
    - [ttl、expire](#ttlexpire)
    - [move、select](#moveselect)
    - [del、flushdb、flushall](#delflushdbflushall)
  - [部署LNP+Redis](#部署lnpredis)
    - [部署LNP环境](#部署lnp环境)
      - [软件安装](#软件安装)
      - [配置动静分离](#配置动静分离)
      - [配置php-fpm](#配置php-fpm)
      - [测试配置](#测试配置)
      - [配置redis](#配置redis)
    - [配置PHP支持redis](#配置php支持redis)
      - [安装软件](#安装软件)
      - [调用模块](#调用模块)
      - [加载模块](#加载模块)
    - [测试](#测试)
  - [创建redis集群](#创建redis集群)
    - [环境准备](#环境准备)
    - [配置redis](#配置redis-1)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)


# 学习目标

部署Redis服务

部署LNP+Redis

创建Redis集群

# 课堂笔记（命令）

# 课堂笔记（文本）

## Redis服务部署

### Redis介绍

> + Remote Dictionary Server （远程字典服务器）
> + 非关系型数据库（nosql）
> + 一款高性能的分布式内存数据库
> + 支持多种数据类型：字符、列表、散列、集合
> + 支持master-slave模式数据备份

### 安装redis

```sh
# yum安装即可
[root@redis64 ~]# yum -y install redis
# 服务启动
[root@redis64 ~]#systemctl enable redis --now
# 查看端口
[root@redis64 ~]#ss -ntulp|grep 6379
tcp   LISTEN 0      128        127.0.0.1:6379      0.0.0.0:*    users:(("redis-server",pid=18684,fd=6))
# 查看进程
[root@redis64 ~]# ps -C redis-server
    PID TTY          TIME CMD
  18684 ?        00:00:00 redis-server
```

### 基本操作

```sh
# 连接服务
[root@redis64 ~]#redis-cli
127.0.0.1:6379>keys * # 获取所有变量
(empty list or set)
127.0.0.1:6379>set name yyh # 存储变量Key--Value
OK
127.0.0.1:6379>get name # 根据key获取值
"yyh"
127.0.0.1:6379>exit # 端开连接

# 查看redis数据保存目录
[root@redis64 ~]#ls /var/lib/redis
dump.rdb
```

### 修改配置

```sh
[root@redis64 ~]#systemctl stop redsi
[root@redis64 ~]#vim /etc/redis.conf
69 bind 192.168.88.64 # 设置IP，让本机之外的机器可以通过192.168.88.64访问本机的redis
92 port 6364 # 设置端口
207 requirepass 123456  # 设置访问密码 

# 连接时不指定密码
[root@redis64 ~]# redis-cli  -h 192.168.88.64 -p 6364
192.168.88.64:6364> keys * 不输入密码无法正常访问
(error) NOAUTH Authentication required.
192.168.88.64:6364> auth 123456
OK
192.168.88.64:6364> keys *
(empty list or set)

# 连接时指定密码
[root@redis64 ~]# redis-cli  -h 192.168.88.64 -p 6364 -a 123456
192.168.88.64:6364> keys *
(empty list or set)
```

## Redis常用命令

### mset、mget

> 设置多个变量、获取多个变量值

```perl
[root@redis64 ~]# redis-cli  -h 192.168.88.64 -p 6364 -a 123456
192.168.88.64:6364> mset name yyh age 18 sex boy
192.168.88.64:6364> mget name age sex
1) "yyh"
2) "18"
3) "boy"
```

### keys、type

> keys *： 获取所有变量
>
> key ?：获取变量只有一个字符的（key ??）
>
> key xxx ：判断变量是否存在
>
> type name：获取变量类型

```perl
192.168.88.64:6364> keys *
1) "age"
2) "name"
3) "sex"
192.168.88.64:6364> set x 123
OK
192.168.88.64:6364> keys ?
1) "x"
192.168.88.64:6364> keys ???
1) "age"
2) "sex"

192.168.88.64:6364> type age
string
```

### exists

> 判断变量是否存在：0 不存在  1存在

```perl
192.168.88.64:6364> exists bbb
(integer) 0
192.168.88.64:6364> exists age
(integer) 1
```

### ttl、expire

> ttl：检查变量可以在内存里存多久
>
> expire：设置变量的过期时间，不设置变量永不过期  默认以秒为单位

```perl
# -1 代表永久存储  -2代表过期
192.168.88.64:6364> ttl age
(integer) -1

# 设置x变量30s后过期  1代表设置过期时间成功
192.168.88.64:6364> expire x 30
(integer) 1
192.168.88.64:6364> ttl x
(integer) 29 # 29 代表剩余过期时间
# 过期返回 -2
192.168.88.64:6364> ttl x
(integer) -2
```

### move、select

> move：移动变量到指定库
>
> select：切换到指定库

```sh
# 切换到1库；默认在0库
192.168.88.64:6364> select 1
192.168.88.64:6364[1]> select 1
# 移动age便来age变量到1库
192.168.88.64:6364> move age 1
192.168.88.64:6364[1]> keys *
1) "age"
```

### del、flushdb、flushall

> del：删除指定变量
>
> flushdb：删除当前库所有变量
>
> flushall：删除内存中所有变量（慎用）

```sh
192.168.88.64:6364> del sex
192.168.88.64:6364> keys *
1) "name"


192.168.88.64:6364> set a 123
OK
192.168.88.64:6364> keys *
1) "a"
2) "name"
192.168.88.64:6364>flushdb
192.168.88.64:6364> keys *
(empty list or set)


192.168.88.64:6364> select 1
192.168.88.64:6364[1]> keys *
1) "age"
192.168.88.64:6364> flushall
OK
```

## 部署LNP+Redis

### 部署LNP环境

#### 软件安装

```sh
# 安装nginx
[root@redis64 ~]# tar -zxf nginx-1.22.1.tar.gz 
# 安装php和所需依赖
[root@redis64 ~]# yum -y install gcc make pcre-devel zlib-devel php php-fpm php-devel
# 编译安装
[root@redis64 ~]#cd nginx-1.22.1/
[root@redis64 nginx-1.22.1]# ./configure && make && make install
[root@redis64 nginx-1.22.1]# ls /usr/local/nginx/
conf  html  logs  sbin
```

#### 配置动静分离

```sh
 [root@redis64 nginx-1.22.1]#vim +65 /usr/local/nginx/conf/nginx.conf
 65         location ~ \.php$ {
 66             root           html;
 67             fastcgi_pass   127.0.0.1:9000;
 68             fastcgi_index  index.php;
 69            # fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_scri    pt_name;
 70             include        fastcgi.conf;
 71         }
 
 # 检查nginx配置文件语法
[root@redis64 nginx-1.22.1]#/usr/local/nginx/sbin/nginx -t 
[root@redis64 nginx-1.22.1]#/usr/local/nginx/sbin/nginx 
[root@redis64 nginx-1.22.1]#ss -ntulp|grep 80
tcp   LISTEN 0      128          0.0.0.0:80        0.0.0.0:*    users:(("nginx",pid=24537,fd=6),("nginx",pid=24536,fd=6))
```

#### 配置php-fpm

```sh
[root@redis64 nginx-1.22.1]#vim +38 /etc/php-fpm.d/www.conf
38 ;listen = /run/php-fpm/www.sock # 注释
39 listen = 127.0.0.1:9000  # 添加，使用非socke方式运行
[root@redis64 nginx-1.22.1]#systemctl start php-fpm
[root@redis64 nginx-1.22.1]#ss -ntulp|grep 9000
tcp   LISTEN 0      128        127.0.0.1:9000      0.0.0.0:*   
```

#### 测试配置

```sh
[root@redis64 nginx-1.22.1]#vim /usr/local/ngixn/html/a.php
<?php
echo "hello word \n"
?>
[root@redis64 nginx-1.22.1]#curl http://localhost/a.php
hello word
```

#### 配置redis

> 允许本机回环地址和IP访问

```sh
[root@redis64 nginx-1.22.1]#vim +69 /etc/redis.conf
bind 127.0.0.1 192.168.88.64 # 本机也可以访问，其他服务也可以访问
[root@redis64 nginx-1.22.1]#systemctl restart redis
[root@redis64 nginx-1.22.1]#ss -ntulp|grep redis-server
tcp   LISTEN 0      128    192.168.88.64:6364      0.0.0.0:*    users:(("redis-server",pid=25041,fd=7))                                                                                                                                  
tcp   LISTEN 0      128        127.0.0.1:6364      0.0.0.0:*    users:(("redis-server",pid=25041,fd=6))   

# 两个地址都能访问
[root@redis64 ~]# redis-cli -h 127.0.0.1 -p 6364 -a 123456
[root@redis64 ~]# redis-cli -h 192.168.88.64 -p 6364 -a 123456
```

### 配置PHP支持redis

```sh
[root@redis64 ~]#php -m | grep -i redis # php模块中没有支持redis的模块，需要手动安装
```

#### 安装软件

```sh
[root@redis64 ~]# tar -xf redis-cluster-4.3.0.tgz
[root@redis64 ~]# cd redis-4.3.0/ 进源码目录
[root@redis64 ~]#phpize # 获取版本信息，并生成./configure文件
# --with-php-config 指定php的配置文件路径，以便php集成
[root@redis64 redis-4.3.0]# ./configure --with-php-config=/usr/bin/php-config  
[root@redis64 redis-4.3.0]# make && make install 编译并安装
Installing shared extensions:     /usr/lib64/php/modules/ # 显示模块安装的位置
[root@redis64 redis-4.3.0]# ls /usr/lib64/php/modules/redis.so  查看模块
/usr/lib64/php/modules/redis.so
```

#### 调用模块

```sh
[root@redis64 redis-4.3.0]# vim /etc/php.ini  # 编辑php进程主配置文件
737 extension_dir = "/usr/lib64/php/modules/"   # 指定模块所在目录
739 extension = "redis.so"  指定模块名
```

#### 加载模块

```sh
[root@redis64 redis-4.3.0]# systemctl  restart php-fpm
[root@redis64 ~]# php -m | grep redis
redis
```

### 测试

```sh
# 编写一个php脚本测试
root@redis64 ~]# vim /usr/local/nginx/html/redis.php
<?php
$redis = new redis();
$redis->connect("127.0.0.1", "6364");
$redis->auth("123456");
$redis->set("name","yyh");
echo "save ok\n";
?>

# 访问网页
[root@redis64 redis-4.3.0]# curl 192.168.88.64/redis.php
save ok
# 查询是否存储
[root@redis64 redis-4.3.0]# redis-cli -h 127.0.0.1 -p 6364 -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6364> keys *
1) "naem"
127.0.0.1:6364> get class
"yyh"
```

## 创建redis集群

> 解决redis服务的单点故障
>
> 数据的自动备份
>
> 数据分布式存储

### 环境准备

![](../pic/dba/d8-6.png)

### 配置redis

> 6台机器按照如下进行配置

```sh
[root@redis51 ~]# yum -y install redis
[root@redis51 ~]#vim /etc/redis.conf
92 port  6379   # 端口号
69 bind   192.168.88.51  # IP地址
838 cluster-enabled  yes    # 启用集群功能  
846 cluster-config-file  nodes-6379.conf   # 存储集群信息文件
852 cluster-node-timeout  5000 # 集群中主机通信超时时间
[root@redis51 ~]#systemctl start redis
tcp  0  0 192.168.88.51:6379  0.0.0.0:*   LISTEN      21201/redis-serve  
tcp  0  0 192.168.88.51:16379   0.0.0.0:*   LISTEN   21201/redis-serve
```

### 检测初始集群

```perl
"查询是否开启集群
1.查询端口16379是否开启
2.cluster info 查询集群状态信息
3.查询配置信息
"
]#sed -n '838p' /etc/redis.conf
cluster-enabled yes
]#redis-cli -h 192.168.88.51
192.168.88.51:6379> cluster info
cluster_state:fail
[root@redis51 ~]# cat /var/lib/redis/nodes-6379.conf 
f5ca7e454190004b613241205b2fd34e458f9bfa :0@0 myself,master - 0 0 0 connected
vars currentEpoch 0 lastVoteEpoch 0
# 在启用集群功能后，查看redis的数据目录中的nodes-xx文件中，将记录myself,master标识本机为主
```

### 开始创建集群

> 在6台机器上任选一台上使用下方命令创建集群都可以

> + \-\-cluster-replicas 1 给每个master服务器分配一台slave服务器，每个主至少要分配1台slave服务器，不然无法实现redis服务的高可用。
>
> + 创建集群时，会自动创建主从角色，默认把主机列表中的前3台服务器创建为
>
>   Master角色的redis服务器，剩下的均配置为slave角色服务器。
>
> + 创建集群时，会自动给master角色的主机分配hash槽 ，通过hash槽实现数据的分布式存储。
>
> + 创建集群时会把默认的16384个槽平均的分配给集群中的3个master服务器。可以通过查看集群信息查看每个master服务器占用hash槽的个数。
>
> + 集群中slave角色的主机会自动同步master角色主机的数据，实现数据的自动备份，
>
>   分别连接集群中的3台slave服务器查看变量

```perl
]#redis-cli create --cluster create 192.168.88.51:6379 192.168.88.52:6379 192.168.88.53:6379 192.168.88.54:6379 192.168.88.55:6379 192.168.88.56:6379 --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460 # 分配第一个集群5460个hash槽
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
......
Can I set the above configuration? (type 'yes' to accept): yes
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered. # 创建成功标识
# 输入完yes后10-20s将会完成创建，创建不成功，参考下方创建失败解决方案
# 集群创建默认将前三台机器作为主，后三台作为从
```

### 创建失败

> 1. 执行创建集群命令后在6台机器上分别执行下方操作
> 2. 再在6台机器上任选一台重新创建集群即可

```perl
]#systemctl stop redis
# 将redis恢复到开启集群功能后的初始状态
]#rm -rf /var/lib/redis/*
]#systemctl start redis
```

### 查询集群信息

> + 集群已搭建成功
> + 集群内任意一台机器都可查询

> info信息详解：
>
> 第一列：主服务器ip地址
>
> 第二列：主服务器ID
>
> 第三列：存储变量个数
>
> 第四列：hash槽个数 （hash槽的作用在集群存储工程过程里讲）
>
> 第五列：从服务器数量

```perl
"查询集群基本信息"
]#redis-cli --cluster info 192.168.88.51:6379
192.168.88.51:6379 (f5ca7e45...) -> 0 keys | 5461 slots | 1 slaves.
192.168.88.53:6379 (f0aa4e1c...) -> 0 keys | 5461 slots | 1 slaves.
192.168.88.52:6379 (6982bcae...) -> 0 keys | 5462 slots | 1 slaves.
[OK] 0 keys in 3 masters.
0.00 keys per slot on average.

"查看初始创建集群详细信息"
]#redis-cli --cluster check 192.168.88.51:6379
92.168.88.51:6379 (f5ca7e45...) -> 0 keys | 5461 slots | 1 slaves.
192.168.88.53:6379 (f0aa4e1c...) -> 0 keys | 5461 slots | 1 slaves.
192.168.88.52:6379 (6982bcae...) -> 0 keys | 5462 slots | 1 slaves.
......
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

### 测试集群

#### 单点故障

> + 当master角色的主机宕机后，对应的slave机器将会自动升级为master
>
> + master角色的主机恢复后，自动做slave角色的主机

```perl
"关闭master主机51，观察对应的从54是否主动接替主的工作"
# 查询初始54机器的配置信息,myself,slave  
[root@redis54 ~]# cat /var/lib/redis/nodes-6379.conf 
c3a7623a5c2c5949463f30b2af50c90491053440 192.168.88.54:6379@16379 myself,slave f5ca7e454190004b613241205b2fd34e458f9bfa 0 1705887807000 4 connected

# 停止51机器的redis服务，再观察，myself,master
[root@redis51 ~]# systemctl stop redis
[root@redis54 ~]# cat /var/lib/redis/nodes-6379.conf 
c3a7623a5c2c5949463f30b2af50c90491053440 192.168.88.54:6379@16379 myself,master - 0 1705891913000 7 connected 0-5460
f5ca7e454190004b613241205b2fd34e458f9bfa 192.168.88.51:6379@16379 master,fail - 1705891908157 1705891907555 1 disconnected

# 查询集群基本信息,此时54接替51的主
[root@redis54 ~]# redis-cli --cluster info 192.168.88.52:6379
Could not connect to Redis at 192.168.88.51:6379: Connection refused
192.168.88.52:6379 (6982bcae...) -> 2 keys | 5462 slots | 1 slaves.
192.168.88.54:6379 (c3a7623a...) -> 3 keys | 5461 slots | 0 slaves.
192.168.88.53:6379 (f0aa4e1c...) -> 2 keys | 5461 slots | 1 slaves.

# 恢复原先51主时，此时51主机将变为从机了 myself,slave 
[root@redis51 ~]# systemctl start redis
f5ca7e454190004b613241205b2fd34e458f9bfa 192.168.88.51:6379@16379 myself,slave c3a7623a5c2c5949463f30b2af50c90491053440 0 1705893511205 1 connected
```

#### 实现分布式存储

> + 存储的值，根据集群算法得出一个值，当值在当初创建集群时给每个集群划分的hash槽的范围中，则该值将储存在该集群中
> + slave主机没有卡槽，做同步master主机的数据

```perl
# 查询集群的卡槽
[root@redis51 ~]#redis-cli --cluster check 192.168.88.51:6379
M: f5ca7e454190004b613241205b2fd34e458f9bfa 192.168.88.51:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: f0aa4e1c609222ed4e8c762942ea7ba8cbf8941f 192.168.88.53:6379
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: c3a7623a5c2c5949463f30b2af50c90491053440 192.168.88.54:6379
   slots: (0 slots) slave
   replicates f5ca7e454190004b613241205b2fd34e458f9bfa
S: 61182b08958d02840c2ca0b4c9324cfab867b3ed 192.168.88.55:6379
   slots: (0 slots) slave
   replicates 6982bcae74df508bc437bee09097a89dd45cadf2
M: 6982bcae74df508bc437bee09097a89dd45cadf2 192.168.88.52:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 71609ea549c17e1e873999da2614b1b5a05b9209 192.168.88.56:6379
   slots: (0 slots) slave
   replicates f0aa4e1c609222ed4e8c762942ea7ba8cbf8941f
```

```perl
# 每次储存数据的hash槽根据算法自动计算
[root@redis51 ~]# redis-cli -c -h 192.168.88.53 -p 6379
192.168.88.53:6379> set name bob
-> Redirected to slot [5798] located at 192.168.88.52:6379
OK
192.168.88.52:6379> set age 18
-> Redirected to slot [741] located at 192.168.88.51:6379
OK
192.168.88.51:6379> set sex boy # 未输出信息，则直接存储再51上
OK
192.168.88.51:6379> set rank 1
-> Redirected to slot [13998] located at 192.168.88.53:6379
OK
192.168.88.53:6379> 
```

> + 任选一台机器执行数据存储命令即可
>
> + 观察每次set变量后屏幕输出的信息以及OK后redis命令行的IP
>
>   第一次储存name变量后，输出hash槽为5798，根据查询的集群信息卡槽5798在5461-10922中，且该范围在集群52中，所以数据直接储存在52的redis内
>
> + 命令行未输出，则数据存储在当前命令行IP对应的redis中



# 快捷键

# 问题

# 补充

# 今日总结

# 昨日复习
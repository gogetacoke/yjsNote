- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
	- [主从复制](#主从复制)
		- [服务准备](#服务准备)
		- [查询主从复制信息](#查询主从复制信息)
		- [配置一主一从](#配置一主一从)
			- [配置](#配置)
			- [验证](#验证)
		- [配置一主多从](#配置一主多从)
			- [配置](#配置-1)
			- [验证](#验证-1)
		- [配置主从从](#配置主从从)
			- [配置](#配置-2)
			- [验证](#验证-2)
	- [带有验证主从复制](#带有验证主从复制)
		- [设置密码-重启](#设置密码-重启)
		- [设置密码-不重启](#设置密码-不重启)
	- [哨兵服务](#哨兵服务)
		- [服务准备](#服务准备-1)
		- [配置主从](#配置主从)
		- [配置哨兵](#配置哨兵)
		- [启动哨兵服务](#启动哨兵服务)
		- [测试配置](#测试配置)
	- [数据持久化](#数据持久化)
		- [服务准备](#服务准备-2)
		- [RDB](#rdb)
			- [存盘频率](#存盘频率)
			- [测试](#测试)
			- [RDB文件恢复](#rdb文件恢复)
			- [RDB缺点](#rdb缺点)
		- [AOF](#aof)
			- [存盘频率](#存盘频率-1)
			- [启用](#启用)
			- [AOF文件恢复](#aof文件恢复)
	- [Redis基本操作](#redis基本操作)
		- [字符类型](#字符类型)
			- [set](#set)
			- [数字操作](#数字操作)
			- [append](#append)
			- [strlen](#strlen)
			- [getrange](#getrange)
		- [列表类型](#列表类型)
		- [散列类型](#散列类型)
		- [集合类型](#集合类型)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)

# 学习目标

主从复制

哨兵服务

持久化

数据类型

# 课堂笔记（命令）

# 课堂笔记（文本）

## 主从复制

![](../pic/dba/d10-2.png)

### 服务准备

![](../pic/dba/d10-1.png)

```perl
"三台主机同步设置"
[root@host61 ~]# yum -y install redis 
[root@host61 ~]# vim /etc/redis.conf
69 bind 192.168.88.61  设置服务使用的Ip地址
92 port 6379 设置服务使用的端口号 使用默认端口即可
[root@host61 ~]#systemctl start redis
```

### 查询主从复制信息

> 检查当前主机是否为主从，默认机器都是master

```perl
[root@R61 ~]# redis-cli -h 192.168.88.61
192.168.88.61:6379> info replication
# Replication
role:master # 默认就是主服务器
connected_slaves:0 # 没有从服务器
......
```

### 配置一主一从

> 61为主  默认为主无需配置
>
> 62为从
>
> 从机器不允许写，只允许读

#### 配置

```perl
[root@R62 ~]# redis-cli -h 192.168.88.62
192.168.88.62:6379> replicaof 192.168.88.61 6379 # 指定本机作为61机器的从服务器
OK
192.168.88.62:6379> config rewrite # 将配置永久写入文件
OK
192.168.88.62:6379> info replication
# Replication
role:slave # 角色变为从
master_host:192.168.88.61
master_port:6379

# 查询主的信息
[root@R61~]# redis-cli -h 192.168.88.61
192.168.88.61:6379> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.88.62,port=6379,state=online,offset=28,lag=0
```

#### 验证

```perl
"主写入数据"
192.168.88.61:6379> mset name yyh age 18 sex boy
OK
192.168.88.61:6379> keys *
1) "age"
2) "sex"
3) "name"

"查询从是否同步"
192.168.88.62:6379> keys *
1) "sex"
2) "age"
3) "name"

"验证从是否可以写入"
192.168.88.62:6379> set rank 1
(error) READONLY You can't write against a read only replica.
```

### 配置一主多从

> **[注]：**slave角色的主机，本机有数据时将会被替换成主机同步的数据，原有数据将会被覆盖

#### 配置

> 将63作为第二从服务器，且配置同62配置从服务器一样

```perl
[root@R63 ~]# redis-cli -h 192.168.88.63
192.168.88.63:6379> info replication
# Replication
role:master # 默认为主
connected_slaves:0
.....
192.168.88.63:6379> replicaof 192.168.88.61 6379 # 指定从服务器
OK
192.168.88.63:6379> config rewrite # 永久生效配置
OK
192.168.88.63:6379> info replication
# Replication
role:slave # 变为从
master_host:192.168.88.61
master_port:6379

"在主61上查看主从复制信息"
192.168.88.61:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.88.62,port=6379,state=online,offset=931,lag=1
slave1:ip=192.168.88.63,port=6379,state=online,offset=931,lag=1 # 指定成功
```

#### 验证

```perl
192.168.88.61:6379> keys *
1) "age"
2) "sex"
3) "name"

"由于从会自动同步主的数据"
192.168.88.63:6379> keys *
1) "age"
2) "sex"
3) "name"

"验证能否写入"
192.168.88.63:6379> set rank 1
(error) READONLY You can't write against a read only replica.
```

### 配置主从从

>+ 在一个主从结构中，将从服务器再配置一个服务器，此时原从服务器可作为新服务器的主，此时结构就成为了主从从结构

#### 配置

```perl
"未删除前主服务器的复制信息"
192.168.88.61:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.88.62,port=6379,state=online,offset=931,lag=1
slave1:ip=192.168.88.63,port=6379,state=online,offset=931,lag=1

"未配置前62主机的复制信息"
192.168.88.62:6379> info replication
# Replication
role:slave
.....
slave_read_only:1
connected_slaves:0
....

"将63作为61的主删除掉"
192.168.88.63:6379> replicaof no one # 恢复为主角色
OK
192.168.88.63:6379> config rewrite
OK
192.168.88.63:6379> keys * # 恢复成功后，原先同步的数据不会被删除
1) "age"
2) "sex"
3) "name"

"删除后的主复制信息" 
192.168.88.61:6379> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.88.62,port=6379,state=online,offset=7553,lag=0

"配置62做为63的主"
192.168.88.63:6379> replicaof 192.168.88.62 6379
OK
192.168.88.63:6379> config rewrite
OK

"配置后查询62机器的复制信息"
192.168.88.62:6379> info replication
# Replication
role:slave
....
slave_read_only:1
connected_slaves:1
slave0:ip=192.168.88.63,port=6379,state=online,offset=2037,lag=0
```

#### 验证

```perl
"删除61主的所有缓存信息"
192.168.88.61:6379> flushall  # 谨慎 执行
OK
192.168.88.61:6379> keys *
(empty list or set)

"查询62 63机器的缓存信息"
192.168.88.62:6379> keys *
(empty list or set)

192.168.88.63:6379> keys *
(empty list or set)
```

## 带有验证主从复制

> master角色主机设置一个密码，访问redis时需要输入密码

### 设置密码-重启

> + 通过修改配置文件设置，需要重启redis服务

```perl
"61机器设置密码"
[root@R61 ~]# vim  +507 /etc/redis.conf 
 507  requirepass 123456
 
"验证密码"
[root@R61 ~]# systemctl restart redis #重启服务生效配置
[root@R61 ~]# redis-cli -h 192.168.88.61
192.168.88.61:6379> keys * # 设置了密码时，无密码访问能登陆，但无法查询数据
(error) NOAUTH Authentication required.
192.168.88.61:6379> auth 123456 # 验证密码后可以
OK
192.168.88.61:6379> keys *
(empty list or set)
192.168.88.61:6379> info replication
# Replication
role:master
connected_slaves:0 # 此时从服务器变为了0个

"未配值密码查询从服务的复制信息"
192.168.88.62:6379> info replication
# Replication
role:slave
master_host:192.168.88.61
master_port:6379
master_link_status:down # 没设置密码，与master主机是断开状态

"修改62主机配置文件设置master主机的密码"
[root@R61 ~]# vim  +293 /etc/redis.conf
 293 masterauth 123456
[root@R61 ~]#systemctl restart redis # 重启生效配置
[root@R61 ~]# redis-cli -h 192.168.88.62
192.168.88.62:6379> info replication
# Replication
role:slave
master_host:192.168.88.61
master_port:6379
master_link_status:up # 变为up状态
```

### 设置密码-不重启

> + 在不重启redis服务的情况下完成带有验证的主从复制

![](../pic/dba/d10-3.png)

> (根据主从从配置中，以下用63作为64主来做测试）

```perl
[root@R62 ~]# redis-cli -h 192.168.88.62
192.168.88.62:6379> config get requirepass # 获取本机的密码
1) "requirepass"
2) ""
192.168.88.62:6379> config set requirepass 654321 # 设置密码
OK
192.168.88.62:6379> keys * # 此时是立即生效
(error) NOAUTH Authentication required.
192.168.88.62:6379> auth 654321 # 输入密码后才能进行操作
OK
192.168.88.62:6379> config rewrite # 写入配置文件，永久生效
OK
192.168.88.62:6379> config get requirepass
1) "requirepass"
2) "654321"

"查询复制信息"
192.168.88.62:6379> info replication
# Replication
role:slave
......
connected_slaves:0 # 当前下没有从服务器了

"查询63的复制信息"
[root@R63 ~]# redis-cli -h 192.168.88.63
192.168.88.63:6379> config get masterauth # 通过获取主账号信息，查询到未配置主的密码
1) "masterauth"
2) ""
192.168.88.63:6379> info replication # 过5s后查询，此时主状态未down
# Replication
role:slave
master_host:192.168.88.62
master_port:6379
master_link_status:down #  没设置密码 与master主机是断开状态

"在63上设置主62的密码"
192.168.88.63:6379> config set masterauth 654321
OK
192.168.88.63:6379> config rewrite
OK
192.168.88.63:6379> info replication
# Replication
role:slave
master_host:192.168.88.62
master_port:6379
master_link_status:up  # 已经变为up状态

"查询62主机复制信息"
192.168.88.62:6379> info replication
# Replication
role:slave
.....
connected_slaves:1
slave0:ip=192.168.88.63,port=6379,state=online,offset=4256,lag=1
```

## 哨兵服务

> + 负责监视从结构中的master角色服务器上redis服务的状态，当监视不到master角色主机的redis服务时，把对应的slave服务器升级未master服务器，宕及的master服务器修复后，配置为但前master服务器的slave服务器
> + 如果主从结构中的redis服务设置连接密码的话必须全每台数据库都要设置密码且密码要一样，要么全都不设置密码。
> + 如果Redis服务有密码宕机的服务器启动服务后，要人为指定主服务器的连接密码。
>
> 哨兵服务+主从(一主多从、主从从)结构：可以实现redis服务的高可用和数据备份功能

### 服务准备

![](../pic/dba/d10-4.png)

> 安装redis服务

### 配置主从

> 配置67作为68的主服务器

```perl
[root@R68 ~]# redis-cli -h 192.168.88.68
192.168.88.68:6379> replicaof 192.168.88.67 6379
OK
192.168.88.68:6379> config rewrite
OK
192.168.88.68:6379> info replication
# Replication
role:slave
master_host:192.168.88.67
master_port:6379
master_link_status:up

[root@R67 ~]# redis-cli -h 192.168.88.67
192.168.88.67:6379> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.88.68,port=6379,state=online,offset=70,lag=0
```

### 配置哨兵

```perl
[root@R69 ~]# vim /etc/redis-sentinel.conf
15 bind 192.168.88.69  # 指定哨兵服务工作的地址
21 port 26379  # 指定哨兵服务运行时监听端口
26 daemonize yes  # 以守护进程方式运行服务，让哨兵服务一直监视主从状态，如果为no，哨兵服务会在一段时间处休眠状态，无法及时让从服务器顶替主服务器
# 添加监视信息 
"
mymaster被监视主机名称
192.168.88.67 6379  当前主从结构中主服务器的IP、端口
1 哨兵服务器的台数(票数)
"
84 sentinel monitor mymaster 192.168.88.67 6379 1 
86 # sentinel auth-pass <master-name> <password> # 指定被监视主服务器的密码
```

> 在设置主从redis密码时，要么不设置密码，要么主从的密码都配置一样

### 启动哨兵服务

```perl
[root@R69 ~]#systemctl start redis-sentinel
[root@R69 ~]# ss -ntulp|grep 2637
tcp   LISTEN 0      128    192.168.88.69:26379      0.0.0.0:*    users:(("redis-sentinel",pid=1213,fd=6))
"查看监视信息"
[root@host69 ~]# tail -f /var/log/redis/sentinel.log   
```

### 测试配置

> + 停止master主机的redis 服务，原slave角色会升级为主，哨兵服务会自动监视新的master服务，宕机的master 主机恢复后自动配置为当前主的从服务器。

```perl
"stop主机的redis服务"
[root@host67 ~]# systemctl  stop redis

"查询哨兵服务器日志"
[root@R69 ~]# tail -f /var/log/redis/sentinel.log 
1213:X 22 Jan 2024 15:27:37.210 # +promoted-slave slave 192.168.88.68:6379 192.168.88.68 6379 @ mymaster 192.168.88.67 6379
.....
1213:X 22 Jan 2024 15:27:37.270 * +slave slave 192.168.88.67:6379 192.168.88.67 6379 @ mymaster 192.168.88.68 6379
1213:X 22 Jan 2024 15:28:07.293 # +sdown slave 192.168.88.67:6379 192.168.88.67 6379 @ mymaster 192.168.88.68 6379

"查询slave角色信息"
[root@host68 ~]# redis-cli  -h 192.168.88.68 -p 6379
192.168.88.68:6379> info replication
# Replication
role:master
connected_slaves:0

"宕机主机恢复后自配置为当前主的从服务器"
192.168.88.67:6379> info replication
# Replication
role:slave
master_host:192.168.88.68
master_port:6379
master_link_status:up
```

## 数据持久化

### 服务准备

> 准备一台新机器：127.0.0.1
>
> 安装启动redis

### RDB

> + 数据持久默认方式
> + 按照指定时间间隔，将内存中的数据集快照写入硬盘

```sh
"定义rdb文件名,可自定义rbd文件名"
]#vim +253 /etc/redis.conf
253 dbfilename dump.rdb
"查询rdb存储位置"
]# ls /var/lib/redis
```

#### 存盘频率

> 查看 redis服务存储数据到硬盘的存盘频率
>
> 变量个数：增、删、改次数之和

```sh
[root@redis70 ~]# vim /etc/redis.conf      
	 save  秒 变量个数
 218 save 900 1
 219 save 300 10
 220 save 60 10000  
 "
 save
 bgsave
 "
```

> 60  一分钟内存储变量个数大于10000个，将存储到rdb文件中
>
> 300 五分钟内存储变量个数大于10个
>
> 900 十五分钟内存储变量的个数大于1个
>
> 如果1分钟内存储变量为9000个，且当前1分钟内容不会写入rdb文件中，当时间过渡到5分钟时此时9000>10满足存储，直接存储到rbd文件中

**补充**

> 不想存储的数据需要等满足存储频率时才存储到磁盘，敲击如下命令可马上就存储在磁盘中

> + `SAVE`命令：这是一个阻塞式的命令，它会将当前数据库的数据以同步（Synchronous）的方式保存到磁盘上的持久化文件。当执行`SAVE`命令时，Redis会阻塞所有其他客户端的请求，直到持久化完成为止。这意味着在持久化期间，Redis无法处理其他命令请求，可能会导致性能问题。
>
>   ```perl
>   127.0.0.1>set name 1
>   ok
>   127.0.0.1>save
>   ```
>
> + `BGSAVE`命令：这是一个非阻塞式的命令，它会在后台以异步（Asynchronous）的方式进行持久化操作。执行`BGSAVE`命令后，Redis会创建一个子进程来进行持久化操作，而主进程可以继续处理其他请求。通过这种方式，`BGSAVE`命令不会阻塞其他客户端的请求，不会造成性能问题。
>
>   ```perl
>   127.0.0.1>set name 1
>   ok
>   127.0.0.1>bgsave
>   ```

#### 测试

> 修改频率测试

```perl
[root@redis70 ~]#systemctl stop redis      
[root@redis70 ~]# rm -rf /var/lib/redis/*    
[root@redis70 ~]# vim  +219 /etc/redis.conf      
save 900 1
#save 300 10
save 120 10     # 2分钟内且有>=10个变量改变 就把内存里的数据复制到dump.rdb文件里
save 60 10000
[root@redis70 ~]# systemctl start redis
"服务启动后，要在2分钟内存储大于10个变量"
[root@redis70 ~]# redis-cli
127.0.0.1:6379> mset a 1  b 2  c 3  d 4 
OK
127.0.0.1:6379> mset   x 1 y 2  z 3 k 6  i 7  z 9   f 22  zz 99  cc  66
127.0.0.1:6379> exit
"两分钟后再进行查看"
[root@redis70 ~]# ls /var/lib/redis/
dump.rdb
```

#### RDB文件恢复

```perl
"
1. 拷贝dump.rdb文件就是对数据进行备份
2. 进入redis中是执行flushall清空所有变量
3. 停止服务，将备份文件覆盖/var/lib/redis/dump.rdb文件,即为恢复
4. 启动服务，查询数据
"
[root@redis70 ~]# cp /var/lib/redis/dump.rdb  /opt/
[root@redis70 ~]# ls /opt/*.rdb
/opt/dump.rdb
[root@redis70 ~]# redis-cli
127.0.0.1:6379> FLUSHALL 
OK
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> 

# 开始恢复
[root@redis70 ~]# systemctl stop redis
[root@redis70 ~]# rm -rf /var/lib/redis/dump.rdb 
[root@redis70 ~]# \cp /opt/dump.rdb  /var/lib/redis/ # 拷贝的文件注意查看属组关系是否时redis.redis 不然redis服务启动会失败
[root@redis70 ~]# systemctl start redis
[root@redis70 ~]# redis-cli
127.0.0.1:6379> keys *
 1) "i"
 2) "d"
 3) "x"
```

#### RDB缺点

### AOF

> + aof同mysql的binlog日志类似，存储执行的命令
> + 弥补RDB的缺点

#### 存盘频率

```perl
[root@localhost ~]# grep -i -n "append" /etc/redis.conf
728:# appendfsync always # 实时同步，性能要求高
729:appendfsync everysec # 每1s进行一次同步，默认使用
730:# appendfsync no # 不进行同步，性能要求最高
```

> - `appendfsync always`：每个写操作都会立即被同步到磁盘上的持久化文件，保证最高的数据安全性和可靠性，但会降低性能。
> - `appendfsync everysec`：每秒钟进行一次同步，可以提高性能和吞吐量，但在发生故障时可能会有一秒钟的数据丢失。
> - `appendfsync no`：不进行同步，完全依赖于操作系统缓存来处理写操作，性能最高，但在发生故障时会丢失大量数据。

#### 启用

> + 可通过修改主配置文件，重启服务生效
> + 推荐使用此方法：再redis服务中使用config set 命令进行设置，不需要重启服务立即生效

```perl
[root@redis70 ~]# redis-cli
127.0.0.1:6379> config set  appendonly yes    # 启用aof文件
OK
127.0.0.1:6379> config get  appendonly  # 查看是否启用
1) "appendonly"
2) "yes"
127.0.0.1:6379>
127.0.0.1:6379> config rewrite  # 永久生效配置
OK
127.0.0.1:6379> exit # 断开连接

[root@redis70 ~]# ls /var/lib/redis/   # 多出一个.aof文件  
appendonly.aof dump.rdb
[root@redis70 ~]#wc –l  /var/lib/redis/appendonly.aof  # 查看文件行数，默认将相同的操作记录为1行
[root@redis70 ~]# redis-cli 
127.0.0.1:6379> set x 1  
127.0.0.1:6379> set y 2  
127.0.0.1:6379> set z 3  
127.0.0.1:6379>keys * 
1) "x"
2) "z"
3) "y"
127.0.0.1:6379> exit # 断开连接
[root@redis70 ~]#wc –l  /var/lib/redis/ appendonly.aof  查看文件行数
22 /var/lib/redis/appendonly.aof
```

#### AOF文件恢复

> 同RDB恢复一致拷贝备份文件覆盖即可

```perl
"拷贝备份"
[root@redis70 ~]# cp /var/lib/redis/appendonly.aof  /opt/

"删除数据"
[root@redis70 ~]# redis-cli
127.0.0.1:6379> flushall 
127.0.0.1:6379> exit

"恢复数据"
[root@redis70 ~]#systemctl stop redis # 停止服务
# 删除数据库目录中所有文件
[root@redis70 ~]# rm -rf  /var/lib/redis/*
# 拷贝备份文件到数据库目录中
 [root@redis70 ~]# cp /opt/appendonly.aof  /var/lib/redis/
 [root@redis70 ~]# systemctl start redis # 启动服务
 [root@redis70 ~]# redis-cli
127.0.0.1:6379> keys * #  查询恢复的数据
1) "v4"
2) "v3"
.....
```

## Redis基本操作

### 字符类型

#### set

```perl
"set 命令完整格式
set key value [expiration EX seconds|PX milliseconds] NX|XX
EX：过期时间按秒计算
PX：过期时间按毫秒计算
NX：不覆盖赋值(针对已经存在的变量再次赋值)
XX：覆盖赋值(针对已经存在的变量再次赋值)
"

127.0.0.1:6379> set name yyh ex 30 # 变量过期时间为30秒
OK
127.0.0.1:6379> ttl name # ttl查询变量存活时长
(integer) 27
127.0.0.1:6379> type name # 获取变量类型
string

"覆盖赋值与不覆盖赋值"
127.0.0.1:6379> set name yyh
OK
127.0.0.1:6379> set name cxj nx #不覆盖赋值
(nil)
127.0.0.1:6379> get name
"yyh"
127.0.0.1:6379> set name coke xx # 覆盖赋值
OK
127.0.0.1:6379> get name
"coke"

"set 命令的完整格式"
127.0.0.1:6379> set age 18 ex 50 nx
OK
127.0.0.1:6379> get age
"18"
127.0.0.1:6379> ttl age
(integer) 43
```

#### 数字操作

> + 字符串类型可以存储任何形式的字符串，当存储的字符串是整数形式时，Redis 提供了一个实用的命令 INCR，其作用是让当前键值递增，并返回递增后的值。
> +  当要操作的键不存在时会默认键值为0，所以第一次递增后的结果是1。
>
> incr：自增1
>
> incrby：自增指定数
>
> decr：自减1
>
> decr：自减指定数

```perl
127.0.0.1:6379> set  num 1  //创建变量
"自增"
127.0.0.1:6379> INCR num    //+1
(integer) 2
127.0.0.1:6379> GET num     
"2"

"增加指定数"
127.0.0.1:6379> INCRBY num 2   //+2
(integer) 4
"自减"
127.0.0.1:6379> DECR num     //-1
(integer) 3
"自减指定数"
127.0.0.1:6379> DECRBY num 3   // -3
(integer) 0
```

#### append

> 给指定变量所对应的值末尾追加值

```perl
127.0.0.1:6379> SET hi Hello   //创建变量hi
OK
127.0.0.1:6379> APPEND hi " World"   # 因为字符串包含空格，需要使用引号
(integer) 11        # 返回值为hi的总长度
127.0.0.1:6379> GET hi
"Hello World"
```

#### strlen

> 获取values的长度
>
> [注]：中文字符返回字节数，1个中文默认2字节，注意看字符集

```perl
127.0.0.1:6379> STRLEN hi
(integer) 11
- 中文字符返回字节数
127.0.0.1:6379> SET name 张三
OK
127.0.0.1:6379> STRLEN name
(integer) 6   # UTF-8编码的中文，由于“张”和“三”两个字的UTF-8编码的长度都是3，所以此例中会返回6。
```

#### getrange

> 获取指定values部分数据
>
> 变量值的长度按照索引进行取舍，第一个为0以此内推

```perl
127.0.0.1:6379> set zfc  ABCEF   //创建变量
OK
127.0.0.1:6379> GET zfc  //输出变量值
"ABCEF"
127.0.0.1:6379> GETRANGE zfc 0 1  //输出第1个到第2个字符
"AB"
127.0.0.1:6379> GETRANGE zfc 2 4  //输出第3个到第5个字符
"CEF"
127.0.0.1:6379> GETRANGE zfc -2 -1 //输出倒数第2个到第1个字符
"EF"
```

### 列表类型

### 散列类型

### 集合类型





# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习
# 主从DNS搭建

## 概述

> 主域名服务器
>
> + 特定DNS区域的官方服务器，具有唯一性
> + 负责维护该区域内所有的“域名<\-\->IP地址”记录
>
> 从域名服务器
>
> + 也称为辅助域名服务器，可以没有
> + 其维护的“域名<\-\->IP地址”记录取决于主域名服务器

## 服务搭建

### 环境准备

> 三台机器关闭防火墙，selinux以便测试

| 主机名     |              |             |
| ---------- | ------------ | ----------- |
| dns-master | 192.168.88.5 | DNS主服务器 |
| dns-slave  | 192.168.88.6 | DNS从服务器 |
| pc         | 192.168.88.7 | 客户机      |

### 搭建主服务器

```shell
# 软件安装及配置
[root@dns-master ~]# dnf -y install bind bind-chroot bind-utils
[root@dns-master ~]#vim /etc/named.conf
options {
        directory "/var/named";  # 指定地址库文件所在地
        allow-transfer { 192.168.88.6; }; # 指定从服务器的IP地址
        
};      

zone "coke.cn" IN { # 指定根域名
        type master;
        file "coke.cn.zone";
}; 
# 修改地址库文件
[root@dns-master ~]# cp -p /var/named/{named.localhost,coke.cn.zone} 
[root@dns-master ~]# vim /var/named/named.cn.zone 
$TTL 1D
@       IN SOA  @ rname.invalid. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      server
        NS		server2
server  A       192.168.88.5  # 指定主域名服务器
server2 A		192.168.88.6 # 指定从域名服务器
www     A       192.168.4.100 # 指定www的解析结果
[root@dns-master ~]#systemctl restart named
# 指定域名服务器
[root@dns-master ~]#echo nameserver 192.168.88.5 > /etc/resolv.conf 
# 测试
[root@dns-master ~]#nslookup www.coke.cn
Server:         192.168.88.5
Address:        192.168.88.5#53

Name:   www.coke.cn
Address: 192.168.1.100
```

### 搭建从服务器

```shell
# 装包配置
[root@dns-slave ~]# dnf -y install bind bind-chroot
[root@dns-slave ~]#vim /etc/named.conf
options {
        directory       "/var/named";
};

zone "coke.cn" IN { # 指定权威域名
        type slave; # 指定当前服务器为从服务器
        file "coke.cn.slave"; # 指定从主服务器下载下来的地址库文件名称
        masters { 192.168.88.5; }; # 指定主服务器
        masterfile-format text; # 指定从主服务器同步下来的内容为文本内容；默认为加密内容
};
# 验证
[root@dns-slave ~]# systemctl enable named --now
[root@dns-slave ~]# ls /var/named/coke.cn.slave 
/var/named/coke.cn.slave
[root@dns-slave ~]# cat /var/named/coke.cn.slave
$ORIGIN .
$TTL 86400      ; 1 day
coke.cn                 IN SOA  coke.cn. rname.invalid. (
                                0          ; serial
                                86400      ; refresh (1 day)
                                3600       ; retry (1 hour)
                                604800     ; expire (1 week)
                                10800      ; minimum (3 hours)
                                )
                        NS      server.coke.cn.
$ORIGIN coke.cn.
server                  A       192.168.88.5
www                     A       192.168.1.100

# 未加上同步内容为文本
[root@dns-slave ~]#cat /var/named/coke.cn.slave 
e�m;KQ� cokecn,cokecnrnameinvalidQ�     :�*0/Q� cokecnservercokecn*Q�serverc
```

### 客户机测试

> 手动指定域名服务器
>
> nslookup 域名 域名服务器地址

```shell
[root@pc ~]#dnf -y install bind-utils
[root@pc ~]# nslookup www.coke.cn 192.168.88.5
Server:         192.168.88.5
Address:        192.168.88.5#53

Name:   www.coke.cn
Address: 192.168.1.100

[root@pc ~]# nslookup www.coke.cn 192.168.88.6
Server:         192.168.88.6
Address:        192.168.88.6#53

Name:   www.coke.cn
Address: 192.168.1.100
```

### 主从同步

> 以上配置完后，主修改内容从还不能进行同步

```shell
# 修改主地址库文件
[root@dns-master ~]# vim /var/named/coke.cn.zone 
$TTL 1D
@       IN SOA  @ rname.invalid. (
                                   202402240    ; serial # 数据版本号，主从在同步时通过版本号去同步要不要同步数据的更新；通过数据版本号来进行更新，当主的版本号大于从的从就进行更新；数据版本号是以10个数字来组成，一般建议时间年月加第几次修改：2024022401;若需要修改匹配的规则就需要修改一下版本号
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      server
server  A       192.168.88.5
www     A       192.168.2.100
[root@dns-master ~]#systemct restart named
# 将从的地址库文件删除并重启
[root@dns-slave ~]#rm -rf /var/named/coke.cn.slave
[root@dns-slave ~]#systemctl restart named
```

> [注]：每次修改的版本号必须大于上一次设置的版本号

### 主从测试

```shell
# 未修改版本号测试
[root@pc ~]# nslookup www.coke.cn 192.168.88.5
Server:         192.168.88.5
Address:        192.168.88.5#53

Name:   www.coke.cn
Address: 192.168.2.100

[root@pc ~]# nslookup www.coke.cn 192.168.88.6
Server:         192.168.88.6
Address:        192.168.88.6#53

Name:   www.coke.cn
Address: 192.168.1.100

# 修改版本号测试
[root@pc2 ~]# nslookup www.coke.cn 192.168.88.6
Server:         192.168.88.6
Address:        192.168.88.6#53

Name:   www.coke.cn
Address: 192.168.2.100
```

### 调优建议

> 通过上述测试存在的问题：
>
> 从同步主的数据时限过长，久久不能进行同步
>
> 解决办法：
>
> + 删除从同步的文件，重启服务
> + 调整刷新时间

```shell
# 将从中的刷新时间调短
[root@dns-slave ~]#vim /var/named/coke.cn.slave 
$ORIGIN .
$TTL 86400      ; 1 day
coke.cn                 IN SOA  coke.cn. rname.invalid. (
                                2024022403 ; serial
                                30      ; refresh (1 day) #单位为秒
                                3600       ; retry (1 hour)
                                604800     ; expire (1 week)
                                10800      ; minimum (3 hours)
                                )
                        NS      server.coke.cn.
$ORIGIN coke.cn.
server                  A       192.168.88.5
www                     A       192.168.6.100
```

> SOA参数序列解释：
>
> - `2024022403`：序列号（serial number）。每次对区域数据进行更改并成功同步到从属服务器后，这个数字都应该递增。格式通常是YYYYMMDDnn，其中nn是一个版本序号，这样其他DNS服务器可以通过比较自己的序列号与主服务器的序列号来判断是否需要获取新的区域数据。
> - `1D`：刷新（refresh）。这是从服务器检查主服务器上是否有区域数据更新的时间间隔，单位为秒。在这个案例中，从服务器每过一天就会向主服务器请求一次区域数据。
> - `1H`：重试（retry）。如果从服务器尝试刷新失败，它将在一小时后再次尝试联系主服务器。
> - `1W`：过期（expire）。若从服务器连续多次（根据retry设置决定）无法从主服务器获取更新，则认为主服务器失效，从服务器将不再响应对此区域的查询，直到达到这个过期时间（一周）。
> - `3H`：最小TTL（minimum）。这是针对该区域所有资源记录的最低有效TTL值。即使某个资源记录的TTL更低，任何DNS服务器也必须至少保留这个值所指定的时间长度的数据。在此例中，即使某个资源记录的TTL小于3小时，解析器也至少需要缓存3小时。

### 注意

> **调短刷新时间存在的潜在缺点：**
>
> - 增加网络流量：频繁的刷新会导致主从服务器之间的通信更频繁，特别是在大型部署中，可能会增加不必要的网络带宽使用。
> - 增加主服务器负载：主服务器需要处理更多的区域传输请求，这可能在高并发场景下对主服务器造成额外的压力。
> - 对于很少更改的区域来说效率不高：如果DNS区域数据非常稳定，不经常变动，则过于频繁的刷新会显得资源利用率较低。
>
> `refresh`时间间隔设置应当基于业务需求和服务器性能来综合考虑，以平衡实时性与资源消耗。对于要求较高数据一致性的场景，较短的刷新时间是有益的；而对于数据相对静态且对实时性要求不高的环境，保持默认或较长的时间间隔更为合适。
>
> 一般而言，30分钟至几个小时的刷新间隔是常见的配置选择。

## 总结
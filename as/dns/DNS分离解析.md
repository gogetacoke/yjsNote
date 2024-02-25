# DNS分离解析

## 概述

> 当客户机的DNS查询请求的时候
>
> + 能够区分不同客户机的来源地址
> + 为不同类别的客户机提供不同的解析结果(IP地址)

## 服务搭建

### 环境准备

> 三台机器关闭防火墙，selinux以便测试

| 主机名 | IP           | 用途        |
| ------ | ------------ | ----------- |
| server | 192.168.88.5 | DNS主服务器 |
| pc1    | 192.168.88.6 | 客户机      |
| pc2    | 192.168.88.7 | 客户机      |

### 搭建基础DNS服务

```shell
# 软件安装及配置
[root@server ~]# dnf -y install bind bind-chroot bind-utils
[root@server ~]#vim /etc/named.conf
options {
        directory "/var/named";  # 指定地址库文件所在地
};      

zone "coke.cn" IN { # 指定根域名
        type master;
        file "coke.cn.zone";
}; 
# 修改地址库文件
[root@server ~]# cp -p /var/named/{named.localhost,coke.cn.zone} 
[root@server ~]# vim /var/named/named.cn.zone 
$TTL 1D
@       IN SOA  @ rname.invalid. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      server
server  A       192.168.88.5  # 指定域名服务器
www     A       192.168.4.100 # 指定www的解析结果
[root@server ~]#systemctl restart named
# 指定域名服务器
[root@server ~]#echo nameserver 192.168.88.5 > /etc/resolv.conf 
# 测试
[root@server ~]#nslookup www.coke.cn
Server:         192.168.88.5
Address:        192.168.88.5#53

Name:   www.coke.cn
Address: 192.168.1.100

# pc1 2 机器都进行安装配置测试
[root@pc1 ~]#dnf -y install bind-utils
[root@server ~]#echo nameserver 192.168.88.5 > /etc/resolv.conf 
[root@pc1 ~]#nslookup www.coke.cn
Server:         192.168.88.5
Address:        192.168.88.5#53

Name:   www.coke.cn
Address: 192.168.1.100
```

### 分离结构

```shell
# 修改配置文件
[root@server ~]#vim /etc/named.conf
options {
        directory "/var/named";
};

view  "d1"{
        match-clients { 192.168.88.6; }; # 指定符合条件地址访问view；这里可以是一个IP或在一个网段192.168.88.0/24
        zone "coke.cn" IN {
                type master;
                file "coke.cn.zone";
        };
};

view  "d2"{
        match-clients { any; }; # 在分离结构中必须有一个any；dns解析是从上到下进行匹配
        zone "coke.cn" IN {
                type master;
                file "coke.cn.other";  # 指定客户机从上到下匹配都匹配不成功后访问的地址库文件
        };
};
# 创建新的地址库文件
[root@server ~]# cp -p /var/named/{coke.cn.zone,coke.cn.other} 
[root@server ~]# vim /var/named/coke.cn.other 
$TTL 1D
@       IN SOA  @ rname.invalid. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      server
server  A       192.168.88.5
www     A       1.2.3.4
[root@server ~]# systemctl restart named
```

### 客户机测试

```shell
# 192.168.88.5机器测试
[root@server ~]# nslookup www.coke.cn
Server:         192.168.88.5
Address:        192.168.88.5#53

Name:   www.coke.cn
Address: 1.2.3.4

# 192.168.88.6机器测试
[root@pc1 ~]# nslookup www.coke.cn
Server:         192.168.88.5
Address:        192.168.88.5#53

Name:   www.coke.cn
Address: 192.168.1.100

# 192.168.88.7机器测试
[root@pc2 ~]# nslookup www.coke.cn
Server:         192.168.88.5
Address:        192.168.88.5#53

Name:   www.coke.cn
Address: 1.2.3.4
```

## 总结

> 分离解析的作用：针对不同客户端解析不同的结果
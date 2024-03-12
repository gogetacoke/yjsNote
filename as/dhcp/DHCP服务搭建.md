# 目标

> 实现地址分发

# 概述

> DHCP （Dynamic Host Configuration Protoocol）
>
> 动态主机配置协议，由IETF组织制定，用来简化主机地址分配管理
>
> 主要分配以下入网参数：
>
> + IP地址 子网掩码 广播地址
> + 默认网关地址 DNS服务器地址

# 原理

> DHCP地址分配的四次会话：
>
> + DISCOVERY > OFFER > REQUEST > ACK
>   + DISCOVERY：发现，由客户端进行发起；客户端配置了DHCP后如果该机器没有IP地址，将会发起广播，DHCP服务器有接收广播的能力
>   + OFFER：由DHCP服务器给出的响应，响应内容包括(IP、dns等等)
>   + REQUEST：客户端作出回应（客户对此IP是否满意）
>   + ACK：确认IP是否已经分配给客户端，确认前还会做检查，检查当前IP是否被其余机器所占用
>
> 服务端基本概念
>
> + 租期：允许客户机租用IP地址的时间期限，单位为秒
> + 作用域：分配给客户机的IP地址所在的网段
> + 地址池：用来动态分配的IP地址的范围

# 服务安装

```shell
# 检查环境信息
]#getenforce # 是否为permissive或disable
]#dnf remov -y firewalld # 卸载防火墙

# 安装dhcp服务
]#dnf -y install dhcp-server
```

# 修改配置文件

```shell
]#vim /etc/dhcp.dhcpd.conf
subnet 192.168.88.0 netmask 255.255.255.0 { # 指定分配的网段、子网掩码(结合本机的IP)
  range 192.168.88.100 192.168.88.200; # 指定分配IP地址的范围
  option domain-name-servers 8.8.8.8; # 指定域名解析服务器
#  option domain-name "internal.example.org"; # 指定符合此域名的主机才会分配，这里注释不用
  option routers 192.168.88.254; # 指定网关地址
  # option broadcast-address 10.5.5.31; # 指定广播地址，按需设置
  default-lease-time 600; # 设置IP租期，默认遵循此设置
  max-lease-time 7200;
} 
```

> [注]：模板文件所在位置
>
> ```shell
> # 打开/etc/dhcp.dhcpd.conf 会出现以下内容
> 
> #
> # DHCP Server Configuration file.
> #   see /usr/share/doc/dhcp*/dhcpd.conf.example
> #   see dhcpd.conf(5) man page
> #
> 
> # 或安装dhcp-client出现下面这种情况
> 
> #
> # DHCP Server Configuration file.
> #   see /usr/share/doc/dhcp-server/dhcpd.conf.example
> #   see dhcpd.conf(5) man page
> #
> 
> """
> /usr/share... 都是一样的指定了该文件的模板位置，无需关系，只需要在命令行敲击: r /usr/share.... 即可
> 删除不需要的内容
> """
> ```

# 服务验证

```shell
# 重启生效服务
]#systemctl start dhcpd



# 在同网络下新建一台虚拟机进行测试
]#dnf -y install dhcp-client # 安装测试工具
# 测试
]# dhclient -d eth0
Internet Systems Consortium DHCP Client 4.3.6
Copyright 2004-2017 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/eth0/52:54:00:ef:72:e4
Sending on   LPF/eth0/52:54:00:ef:72:e4
Sending on   Socket/fallback
Created duid "\000\004fi\010\314\036}C\014\213u.\350\256\257\032y".
DHCPDISCOVER on eth0 to 255.255.255.255 port 67 interval 4 (xid=0x2a964f78)
DHCPREQUEST on eth0 to 255.255.255.255 port 67 (xid=0x2a964f78)
DHCPOFFER from 192.168.88.11
DHCPACK from 192.168.88.11 (xid=0x2a964f78)
bound to 192.168.88.100 -- renewal in 21277 seconds.
```


- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
  - [nmcli](#nmcli)
    - [查看系统存在网卡链接](#查看系统存在网卡链接)
    - [删除网卡命名](#删除网卡命名)
    - [网卡重命名](#网卡重命名)
    - [激活网卡](#激活网卡)
    - [修改网卡信息](#修改网卡信息)
    - [总结](#总结)
  - [ssh](#ssh)
  - [scp](#scp)
  - [ping](#ping)
- [课堂笔记（文本）](#课堂笔记文本)
  - [修改网卡内核命名规则](#修改网卡内核命名规则)
  - [ssh远程无密码验证](#ssh远程无密码验证)
  - [临时管理ip地址](#临时管理ip地址)
    - [临时添加](#临时添加)
    - [临时删除](#临时删除)
  - [登录用户信息](#登录用户信息)
    - [已登录用户](#已登录用户)
    - [登录失败/成功用户](#登录失败成功用户)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)



# 学习目标

配置Linux网络

日志管理

# 课堂笔记（命令）

## nmcli

### 查看系统存在网卡链接

```
]# nmcli connection show 
NAME        UUID                                  TYPE      DEVICE 
有线连接 1  bf84ae0d-470d-453c-bc69-ffb2a0f83767  ethernet  eth0   
virbr0      8eda539d-5249-4fe1-9196-9a5e03317853  bridge    virbr0 
enp1s0      f5190272-0544-4e60-b9b1-d5017672e45a  ethernet  -- 
```

### 删除网卡命名

格式：nmcli connection delete 网卡名/UUID

```
]# nmcli connection delete virbr0
```

代码解释：删除名字virbro链接，也可以输入UUID，空格进行补全

### 网卡重命名

```
]# nmcli connection  add  type  ethernet      ifname  eth0    con-name   eth0
解析： nmcli connection add   type   以太网设备
网卡设备名为 eth0    nmcli命令的命名为 eth0

连接 “eth0”（a05cdd87-0fa4-4062-b867-7702f7538183）已成功添加
]# nmcli   connection   show # 查询添加是否成功
```

### 激活网卡

```
]#nmcli connection up eth0
]#ifconfig # 查询
```

### 修改网卡信息

作用：修改IP、子网掩码、网关地址

```
]#nmcli connection  modify eth0 
ipv4.method manaul 
ipv4.addresses 192.168.66.88.77/24
ipv4.gateway 192.168.88.200
autoconnect yes
命令参数解读：
nmcli connection 修改 外号 
ipv4.方法 手工配置
ipv4.地址 192.168.88.77/24
ipv4.网关 192.168.88.200
每次开机自动启用以上所有参数
]# nmcli connection up eth0     #激活
]# ifconfig   |   head   -2
]# route   -n        #查看网关地址信息（了解）
```

### 总结

![](../pic/en/1.png)

## ssh

作用：远程管理机器

格式：ssh 以什么身份登录@IP

[注]：不使用身份直接输入IP地址登录，将会以本机登录的身份尝试远程管理对方

```
]# rpm -qa | grep openssh # 检测机器是否安装ssh服务
openssh-askpass-8.0p1-13.el8.x86_64
openssh-8.0p1-13.el8.x86_64
openssh-clients-8.0p1-13.el8.x86_64
openssh-server-8.0p1-13.el8.x86_64
#ls /usr/sbin/sshd#提供远程管理功能的程序
/usr/sbin/sshd
]# pgrep -l  sshd   #搜索sshd进程
1181 sshd
]#ssh root@192.168.88.2
```

## scp

作用：数据传递工具

格式：scp [-r] 本地路径 用户名@服务器:路径

-r：复制有目录时加上

例：将passwd复制到88.2机器上opt目录改名为1.txt

```
]# scp /etc/passwd root@192.168.88.3:/opt/1.txt
oot@192.168.88.3's password: 
passwd                                        100% 2557     2.1MB/s   00:00   
]# ssh root@192.168.88.3
]# ls -l /opt/
总用量 4
-rw-r--r--. 1 root root 2557 11月 13 16:49 1.txt
```

## ping

格式：ping [选项] IP

-c：指定ping包个数

```
]# ping -c2  192.168.88.2
PING 192.168.88.2 (192.168.88.2) 56(84) bytes of data.
64 bytes from 192.168.88.2: icmp_seq=1 ttl=64 time=1.44 ms
64 bytes from 192.168.88.2: icmp_seq=2 ttl=64 time=0.272 ms

--- 192.168.88.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 0.272/0.854/1.437/0.583 ms
```



# 课堂笔记（文本）

## 修改网卡内核命名规则

修改为：eth0、eth1、eth2...

**net.ifnames=0** **biosdevname=0** 

```
]# vim /etc/default/grub 
GRUB_CMDLINE_LINUX="crashkernel=auto resume=/dev/mapper/rl-swap rd.lvm.lv=rl/root rd.lvm.lv=rl/swap rhgb quiet net.ifnames=0 biosdevname=0 "
]# grub2-mkconfig  -o  /boot/grub2/grub.cfg  #生效网卡规则
]# reboot     
]# ifconfig | gead -2 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.131  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::c7c3:192f:15e5:a75e  prefixlen 64  scopeid 0x20<link>
```

代码解释：进入grub文件中，在第6行结尾quit后敲空格输入（光标移动到第6行按end键快速到最后） net.ifname=0 biosdevname=0 保存并退出，输入重新生成网卡命名命令，重启机器，输入ifconfig查看

## ssh远程无密码验证

作用：无密码远程登录另一台机器

例：A机器无密码访问B机器

A机器：

```
]# ssh-keygen #生成公私钥，一路敲击回车即可
The key's randomart image is:
+---[RSA 3072]----+
|  o.oo.oEoo      |
| +o.. o...oo.    |
|. oo o  ...=.    |
| o .. o  +oX     |
|  . .o..S @ B    |
|     ..= + B +   |  
|      . + + + .  |
|       .  .+     |
|          .o.    |
+----[SHA256]-----+
]# ls -l /home/.ssh/ # 查询公私钥
id_rsa  id_rsa.pub  known_hosts
]# ssh-copy-id root@192.168.88.2
```

B机器：

```
[root@pv3 ~]# ls /root/.ssh/
id_rsa.pub
```

测试：

```
]# ssh root@192.168.88.2
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Mon Nov 13 17:40:16 2023 from 192.168.88.240
```

代码解释：在A机器上使用命令ssh_keygen生成公私钥，使用命令ssh-copy-id传递给需要密码输入的机器即可

## 临时管理ip地址

### 临时添加

格式：ip  a a 需要设置的ip/24 dev 网卡名

```
]# ip a a 192.168.88.9/24 dev eth0
]# ip a s
eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:23:80:14 brd ff:ff:ff:ff:ff:ff
    inet 192.168.88.240/24 brd 192.168.88.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
    inet 192.168.88.9/24 scope global secondary eth0
       valid_lft forever preferred_lft forever
```

### 临时删除

格式：ip a d 需要删除的ip/23 dev 网卡名

```
]# ip a d 192.168.88.9/24 dev eth0
]# ip a s
 eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:23:80:14 brd ff:ff:ff:ff:ff:ff
    inet 192.168.88.240/24 brd 192.168.88.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
```

## 登录用户信息

### 已登录用户

作用：查询当前已登录的用户

users、who（常用）、w

```
]# users
root

]# who
root     pts/0        2023-11-14 08:45 (192.168.88.254)

]# w
 10:24:01 up  1:44,  1 user,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    192.168.88.254   08:45    1.00s  0.18s  0.01s w
```

代码解释：

pts 代表当前使用的是图形命令终端

### 登录失败/成功用户

last

登录成功

```
]# last
root     pts/1        192.168.88.2     Tue Nov 14 10:27   still logged in密码
root     pts/2        192.168.88.2     Tue Nov 14 10:26   still logged in
```

lastb

登录失败

```
]# lastb
root     ssh:notty    192.168.88.3     Tue Nov 14 10:28 - 10:28  (00:00)
root     ssh:notty    192.168.88.3     Tue Nov 14 10:28 - 10:28  (00:00)
123      ssh:notty    192.168.88.254   Mon Nov 13 15:47 - 15:47  (00:00)
```

# 快捷键



# 问题



# 补充



# 今日总结

前提：一台崭新机器的操作，完成今天操作

1. 光驱设备已加载虚拟机

2. 挂载驱动，并设置为开机自动挂载

3. mount -a

4. 构建yum仓库

5. yum repoinfo

6. 更改hostname

7. 设置网卡ip、子网掩码：nmcli con modify eths160 ipv4.method manual ipv4.addresses 192.168.88.2 ipv4.gateway 192.168.88.200 autoconnect yes

8. 激活网卡：nmcli connection up eth0

9. 修改网卡别名：nmcli connection add type ehernet ifname eth0 con-name eth0

10. 修改网卡命名规则：vim /etc/defaults/grub  -> net.ifnames=0 biosdevname=0

   更新配置：grub2-mkconfig -o /boot/grub2/grub.cfg

11. 查看系统网卡信息：nmcli connection show

12. 未生效 就 reboot -f

# 昨日复习
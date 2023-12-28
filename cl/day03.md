- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
  - [blockinfile模块](#blockinfile模块)
- [课堂笔记（文本）](#课堂笔记文本)
  - [Ceph概述](#ceph概述)
    - [Ceph实现的存储方式](#ceph实现的存储方式)
    - [Ceph组成部分](#ceph组成部分)
  - [部署Ceph](#部署ceph)
    - [节点准备](#节点准备)
    - [配置名称解析](#配置名称解析)
    - [配置时间服务器](#配置时间服务器)
    - [安装基础依赖软件](#安装基础依赖软件)
    - [安装启动容器仓库服务器](#安装启动容器仓库服务器)
    - [导入镜像](#导入镜像)
    - [配置仓库服务器](#配置仓库服务器)
    - [修改镜像名称](#修改镜像名称)
    - [推送镜像](#推送镜像)
    - [安装ceph](#安装ceph)
    - [进入管理容器](#进入管理容器)
    - [集群添加主机](#集群添加主机)
    - [添加MON](#添加mon)
    - [添加OSD硬盘](#添加osd硬盘)
  - [实现块存储](#实现块存储)
    - [块存储基础](#块存储基础)
    - [存储池](#存储池)
  - [镜像](#镜像)
    - [添加镜像](#添加镜像)
    - [扩容镜像](#扩容镜像)
    - [删除镜像](#删除镜像)
  - [客户端](#客户端)
    - [安装客户端](#安装客户端)
    - [集群密钥文件](#集群密钥文件)
    - [创建镜像](#创建镜像)
    - [映射](#映射)
    - [格式化使用](#格式化使用)
    - [删除](#删除)
- [快捷键](#快捷键)
- [问题](#问题)
  - [什么是块存储？](#什么是块存储)
  - [什么是文件系统？](#什么是文件系统)
  - [什么是对象存储？](#什么是对象存储)
  - [ceph有哪些组成部分？](#ceph有哪些组成部分)
  - [cephadm有哪些作用？](#cephadm有哪些作用)
  - [在配置ceph时，为什么需要ntp？](#在配置ceph时为什么需要ntp)
  - [在配置ceph时，需要使用容器，容器仓库服务器的作用是什么？](#在配置ceph时需要使用容器容器仓库服务器的作用是什么)
  - [容器和镜像的关系是什么？](#容器和镜像的关系是什么)
  - [容器和虚拟机有什么异同？](#容器和虚拟机有什么异同)
  - [ceph mon部署时，有哪些注意事项？](#ceph-mon部署时有哪些注意事项)
  - [ceph存储池的作用是什么？](#ceph存储池的作用是什么)
  - [ceph存储池使用的PG是什么？有什么作用？](#ceph存储池使用的pg是什么有什么作用)
  - [rbd镜像作用是什么？](#rbd镜像作用是什么)
  - [ceph客户端如何配置？](#ceph客户端如何配置)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)

# 学习目标

Ceph

部署Ceph集群

Ceph块存储

# 课堂笔记（命令）

## blockinfile模块

```shell
# 与lineinfile类似，lineinfile只能添加一行，当前模块可以添加多行
- name: add block
      blockinfile:   # 类似于lineinfile模块，可在目标文件中加多行
        path: /etc/hosts
        block: |
          192.168.88.11 ceph1
          192.168.88.12 ceph2
          192.168.88.13 ceph3
          192.168.88.240 quay.io
```

# 课堂笔记（文本）

## Ceph概述

### Ceph实现的存储方式

> - 块存储：提供像普通硬盘一样的存储，为使用者提供“硬盘”
> - 文件系统存储：类似于NFS的共享方式，为使用者提供共享文件夹
> - 对象存储：像百度云盘一样，需要使用单独的客户端

### Ceph组成部分

> Ceph存储集群至少需要一个Ceph监视器、Ceph管理器和Ceph OSD(对象存储守护程序)。运行Ceph文件系统客户端时，需要Ceph元数据服务器。

> - 监视器：Ceph Monitor（ceph-mon）维护集群状态图，包括监视器图、管理器图、OSD图、MDS图和CRUSH图。这些映射是Ceph守护进程相互协调所需的关键集群状态。监视器还负责管理守护程序和客户端之间的身份验证。为了冗余和高可用性，通常至少需要三台Monitor。
> - 管理器：Ceph Manager（ceph-mgr）负责跟踪ceph集群的运行时指标和当前状态，包括存储利用率、当前性能指标和系统负载。Ceph Manager守护进程还托管基于python的模块来管理和公开Ceph集群信息，包括基于web的Ceph仪表板和REST API。高可用性通常需要至少两台Manager。
> - Ceph OSD：ceph-osd存储数据，处理数据复制、恢复、重新平衡，并通过检查其他Ceph OSD守护进程的心跳来为Ceph监视器和管理器提供一些监视信息。为了实现冗余和高可用性，通常至少需要三个Ceph OSD。
> - MDS：ceph-mds代表Ceph文件系统存储元数据(即，Ceph块设备和Ceph对象存储不使用MDS)。Ceph元数据服务器允许POSIX文件系统用户执行基本命令(如ls、find等)。)而不会给Ceph存储集群带来巨大的负担。
> - RGW：Rados Gateway，是一个提供对象存储功能的组件，可以通过RESTful接口向外部应用程序提供可扩展和高可用的存储服务。

## 部署Ceph

### 节点准备

![](../pic/cl/cl-d3-1.png)

> 使用ansible配置yum以及主机名

### 配置名称解析

```shell
[root@pubserver ceph]# vim 02_modify_hosts.yml
---
- name: add names
  hosts: all
  tasks:
    - name: add hosts
      blockinfile:
        path: /etc/hosts
        block: |
          192.168.88.11 ceph1
          192.168.88.12 ceph2
          192.168.88.13 ceph3
          192.168.88.240 quay.io
```

### 配置时间服务器

```shell
[root@pubserver ceph]#vim 03_install_ntp.yml
---
- name: ntp set up
  hosts: all
  tasks:
    - name: uninstall ntp
      yum:
        name: chrony
        state: absent

    - name: install ntp
      yum:
        name: chrony
        state: present
        
    - name: modify chofing
      lineinfile:
        path: /etc/chrony.conf
        regexp: '^pool'
        line: 'pool 192.168.88.240 iburst'

    - name: start ntp
      service:
        name: chronyd
        state: started
        enabled: yes
```

### 安装基础依赖软件

```shell
[root@pubserver ceph]# cat 04_install_pkg.yml 
---
- name: install pkg
  hosts: cephs
  tasks:
    - name: install python39 podman lvm2
      yum:
        name: python39,podman,lvm2
        state: presen
```

### 安装启动容器仓库服务器

```shell
# 1.安装包
[root@pubserver ~]# yum install -y docker-distribution-2.6.2-2.git48294d9.el7.x86_64.rpm
# 2. 启动服务
[root@pubserver ~]# systemctl enable docker-distribution --now
[root@pubserver ~]#ss -ntulp|grep 5000
```

### 导入镜像

```shell
[root@ceph2 ceph-server]# ls
altermanager_v0.23.0.tar  ceph_v17.tar
cephadm                   node-exporter_v1.3.1.tar
ceph-grafana_8.3.5.tar    prometheus_v2.33.4.tar
[root@ceph2 ceph-server]#for i in *.tar;do podman load -i $i;done # 循环加载镜像
```

### 配置仓库服务器

```shell
# 配置ceph1-ceph3使用pubserver作为仓库服务器
[root@pubserver ceph]# vim 05-config-registry.yml
---
- name: config registry
  hosts: cephs
  tasks:
    - name: modify config
      blockinfile:
        path: /etc/containers/registries.conf
        block: |
          [[registry]]
          location = "quay.io:5000"  # 指定服务器地址
          insecure = true              # 允许使用http协议
```

### 修改镜像名称

```shell
# 修改镜像名称，以便可以将其推送到自建镜像服务器
[root@ceph1 ceph-server]# podman tag quay.io/ceph/ceph:v17 quay.io:5000/ceph/ceph:v17
[root@ceph1 ceph-server]# podman tag quay.io/ceph/ceph-grafana:8.3.5 quay.io:5000/ceph/ceph-grafana:8.3.5
[root@ceph1 ceph-server]# podman tag quay.io/prometheus/prometheus:v2.33.4 quay.io:5000/prometheus/prometheus:v2.33.4
[root@ceph1 ceph-server]# podman tag quay.io/prometheus/node-exporter:v1.3.1 quay.io:5000/prometheus/node-exporter:v1.3.1
[root@ceph1 ceph-server]# podman tag quay.io/prometheus/alertmanager:v0.23.0 quay.io:5000/prometheus/alertmanager:v0.23.0
[root@ceph1 ceph-server]#podman images # 查看修改情况

# quay.io 远程仓库域名
# 5000 仓库端口
# 不修改默认为80
```

### 推送镜像

```shell
# 将镜像推送到镜像服务器，以便其他节点可以通过服务器下载镜像
[root@ceph1 ceph-server]# podman push quay.io:5000/ceph/ceph:v17
[root@ceph1 ceph-server]# podman push quay.io:5000/ceph/ceph-grafana:8.3.5 
[root@ceph1 ceph-server]# podman push quay.io:5000/prometheus/prometheus:v2.33.4 
[root@ceph1 ceph-server]# podman push quay.io:5000/prometheus/node-exporter:v1.3.1 
[root@ceph1 ceph-server]# podman push quay.io:5000/prometheus/alertmanager:v0.23.0 

[root@ceph1 ceph-server]# curl 192.168.88.240:5000/v2/_catalog # 查询推送情况
```

### 安装ceph

> - 所有Ceph集群都需要至少一个监视器Monitor，以及至少与存储在集群上的对象副本一样多的OSD。引导初始监视器是部署Ceph存储集群的第一步。
> - Monitor部署还为整个集群设置了重要的标准，例如池的副本数量、每个OSD的放置组数量、心跳间隔、是否需要身份验证等。这些值中的大部分是默认设置的。
> - 创建集群

```shell
# 1. 在ceph1上初始化集ceph集群。
# 集群初始化完成后，将自动生成ssh免密密钥，存放在/etc/ceph/目录下
[root@ceph1 ceph-server]# ./cephadm bootstrap --mon-ip 192.168.88.11 --initial-dashboard-password=123456 --dashboard-password-noupdate
# 2. ceph将会以容器化的方式部署，查看生成了6个容器。
[root@ceph1 ceph-server]# podman ps

# 3. 拷贝密钥文件至其他节点
[root@ceph1 ceph-server]# ssh-copy-id -f -i /etc/ceph/ceph.pub ceph2
[root@ceph1 ceph-server]# ssh-copy-id -f -i /etc/ceph/ceph.pub ceph3


```

### 进入管理容器

```shell
# 4. 进入管理容器，查看ceph状态
[root@ceph1 ceph-server]# ./cephadm shell   # 进入管理容器
[ceph: root@ceph1 /]# ceph -s  # 查看ceph状态
  cluster:
    id:     1ddfccf2-77b4-11ed-8941-000c2953b002
    health: HEALTH_WARN
            OSD count 0 < osd_pool_default_size 3
 
  services:
    mon: 1 daemons, quorum ceph1 (age 11m)
    mgr: ceph1.vnoivz(active, since 10m)
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs: 

# 5. 查看相关容器状态，显示所有容器均已启动
[ceph: root@ceph1 /]# ceph orch ls
NAME           PORTS        RUNNING  REFRESHED  AGE  PLACEMENT  
alertmanager   ?:9093,9094      1/1  91s ago    3m   count:1    
crash                           1/3  91s ago    4m   *          
grafana        ?:3000           1/1  91s ago    4m   count:1    
mgr                             1/2  91s ago    4m   count:2    
mon                             1/5  91s ago    4m   count:5    
node-exporter  ?:9100           1/3  91s ago    4m   *          
prometheus     ?:9095           1/1  91s ago    4m   count:1 


# 6. 查看集群中现有主机
[ceph: root@ceph1 /]# ceph orch host ls
HOST   ADDR           LABELS  STATUS  
ceph1  192.168.88.11  _admin          
1 hosts in cluster
```

### 集群添加主机

```shell
# 7. 向集群中添加其他主机
[ceph: root@ceph1 /]# ceph orch host add ceph2 192.168.88.12
[ceph: root@ceph1 /]# ceph orch host add ceph3 192.168.88.13
# 注：删除错误的主机命令为：ceph orch host rm 主机名 --force

# 8. 查看集群中主机
[ceph: root@ceph1 /]# ceph orch host ls
HOST   ADDR           LABELS  STATUS  
ceph1  192.168.88.11  _admin          
ceph2  192.168.88.12                  
ceph3  192.168.88.13                  
3 hosts in cluster
```

### 添加MON

```shell
# 9. 扩容MON节点。一共有3台MON，位于ceph1-ceph3
[ceph: root@ceph1 /]# ceph orch apply mon --placement="3 ceph1 ceph2 ceph3" 

# 10. 查看mon状态
[ceph: root@ceph1 /]# ceph -s
  cluster:
    id:     a4b69ab4-79dd-11ed-ae7b-000c2953b002
    health: HEALTH_WARN
            OSD count 0 < osd_pool_default_size 3
 
  services:
    mon: 3 daemons, quorum ceph1,ceph3,ceph2 (age 2m)
    mgr: ceph1.gmqorm(active, since 15m), standbys: ceph3.giqaph
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:     
 
[ceph: root@ceph1 /]# ceph mon stat
e3: 3 mons at {ceph1=[v2:192.168.88.11:3300/0,v1:192.168.88.11:6789/0],ceph2=[v2:192.168.88.12:3300/0,v1:192.168.88.12:6789/0],ceph3=[v2:192.168.88.13:3300/0,v1:192.168.88.13:6789/0]}, election epoch 14, leader 0 ceph1, quorum 0,1,2 ceph1,ceph3,ceph2

# 11. ceph2和ceph3上也将会出现相关容器
[root@ceph2 ~]# podman ps
[root@ceph3 ~]# podman ps
```

> ceph自愈能力，容器删除后，将自动进行修复

### 添加OSD硬盘

```shell
[ceph: root@ceph1 /]# ceph orch daemon add osd ceph1:/dev/vdb
[ceph: root@ceph1 /]# ceph orch daemon add osd ceph1:/dev/vdc
[ceph: root@ceph1 /]# ceph orch daemon add osd ceph1:/dev/vdd
[ceph: root@ceph1 /]# ceph orch daemon add osd ceph2:/dev/vdb
[ceph: root@ceph1 /]# ceph orch daemon add osd ceph2:/dev/vdc
[ceph: root@ceph1 /]# ceph orch daemon add osd ceph2:/dev/vdd
[ceph: root@ceph1 /]# ceph orch daemon add osd ceph3:/dev/vdb
[ceph: root@ceph1 /]# ceph orch daemon add osd ceph3:/dev/vdc
[ceph: root@ceph1 /]# ceph orch daemon add osd ceph3:/dev/vdd

# 2. 在节点上查询容器信息，将会发现又有新的osd容器出现
[root@ceph1 ~]# podman ps

# 3. 此时ceph的状态将会是HEALTH_OK，ceph集群搭建完成。
[ceph: root@ceph1 /]# ceph -s
  cluster:
    id:     a4b69ab4-79dd-11ed-ae7b-000c2953b002
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph1,ceph3,ceph2 (age 2m)
    mgr: ceph1.gmqorm(active, since 2h), standbys: ceph3.giqaph
    osd: 9 osds: 9 up (since 35s), 9 in (since 59s)
 
  data:
    pools:   1 pools, 1 pgs
    objects: 2 objects, 449 KiB
    usage:   186 MiB used, 180 GiB / 180 GiB avail
    pgs:     1 active+clean
```

## 实现块存储

### 块存储基础

> - 块设备存取数据时，可以一次存取很多。字符设备只能是字符流

```shell
[root@ceph1 ~]# ll /dev/sda
brw-rw---- 1 root disk 8, 0 Dec 12 13:15 /dev/sda
# b表示block，块设备

[root@ceph1 ~]# ll /dev/tty
crw-rw-rw- 1 root tty 5, 0 Dec 12 13:31 /dev/tty
# c表示character，字符设备
```

> - 块存储，就是可以提供像硬盘一样的设备。使用块存储的节点，第一次连接块设备，需要对块设备进行分区、格式化，然后挂载使用。
> - ceph中的块设备叫做rbd，是rados block device的简写，表示ceph的块设备。rados是Reliable, Autonomic Distributed Object Store的简写，意思是“可靠、自主的分布式对象存储”。
> - Ceph块设备采用精简配置，可调整大小，并将数据存储在多个OSD上。
> - ceph提供存储时，需要使用存储池。为了给客户端提供存储资源，需要创建名为存储池的容器。存储池类似于逻辑卷管理中的卷组。卷组中包含很多硬盘和分区；存储池中包含各节点上的硬盘。
> - 查看基础存储池信息

```shell
# 查看存储池。默认有一个名为.mgr的存储池，编号为1
[ceph: root@ceph1 /]# ceph osd lspools
1 .mgr

# 查看存储详细使用情况
[ceph: root@ceph1 /]# ceph df
--- RAW STORAGE ---
CLASS     SIZE    AVAIL     USED  RAW USED  %RAW USED
hdd    180 GiB  180 GiB  187 MiB   187 MiB       0.10
TOTAL  180 GiB  180 GiB  187 MiB   187 MiB       0.10
 
--- POOLS ---
POOL  ID  PGS   STORED  OBJECTS     USED  %USED  MAX AVAIL
.mgr   1    1  449 KiB        2  449 KiB      0     57 GiB

# 查看.mgr存储池的副本数量
[ceph: root@ceph1 /]# ceph osd pool get .mgr size
size: 3
```

> - 在ceph中，操作块存储的命令也是rbd。
> - 创建存储池。如果不指定操作哪一个存储池，rbd命令将会操作名为rbd的存储池。该存储池不存在，需要自己创建。

```shell
# 不指定存储池名字执行查看操作。提示名为rbd的存储池不存在
[ceph: root@ceph1 /]# rbd ls
rbd: error opening default pool 'rbd'
Ensure that the default pool has been created or specify an alternate pool name.
rbd: listing images failed: (2) No such file or directory
```

### 存储池

> - 创建存储池是需要指定存储池中PG的数量。
> - Placement Group简称PG，可翻译为归置组。
> - PG只是一个组而已，用于把存储的对象分组管理。
> - 将数据放入集群时，对象被映射到pg，而这些pg被映射到OSD。这减少了我们需要跟踪的每个对象的元数据的数量以及我们需要运行的进程的数量。
> - 创建并使用存储池

```shell
#1.创建名为rdb的存储池
[ceph: root@ceph1 /]#ceph osd pool create rdb 100
pool 'rbd' created

# 2. 设置rbd存储池的应用类型是rbd。还可以是rgw或cephfs
# 语法：ceph osd pool application enable <pool-name> <app-name>
[ceph: root@ceph1 /]# ceph osd pool application enable rbd rbd

# 3. 查看
[ceph: root@ceph1 /]# ceph osd pool ls 
.mgr
rbd
[ceph: root@ceph1 /]# ceph df
--- RAW STORAGE ---
CLASS     SIZE    AVAIL     USED  RAW USED  %RAW USED
hdd    180 GiB  180 GiB  191 MiB   191 MiB       0.10
TOTAL  180 GiB  180 GiB  191 MiB   191 MiB       0.10
 
--- POOLS ---
POOL  ID  PGS   STORED  OBJECTS     USED  %USED  MAX AVAIL
.mgr   1    1  897 KiB        2  2.6 MiB      0     57 GiB
rbd    2   99      0 B        0      0 B      0     57 GiB

# 4. 执行命令。不指定存储池，默认操作名为rbd的存储池。
[ceph: root@ceph1 /]# rbd ls   # 无输出内容，也不会报错
```

## 镜像

> - 在存储池中划分空间提供给客户端作为硬盘使用。
> - 划分出来的空间，术语叫做镜像。

### 添加镜像

```shell
[ceph: root@ceph1 /]#rbd create img1 --size 10G # 添加一块镜像名为img1
[ceph: root@ceph1 /]#rbd ls # 列出已有镜像
img1
[ceph: root@ceph1 /]#rbd info  img1 # 查看镜像详情
```

### 扩容镜像

```shell
[ceph: root@ceph1 /]#rbd resize img1 --size 20G # 扩容到20G
[ceph: root@ceph1 /]#rbd info img1
rbd image 'img1':
        size 20 GiB in 5120 objects
        order 22 (4 MiB objects)
        ....
[ceph: root@ceph1 /]#rbd resize img1 --size 10G --allow-shrink# 缩减到10G；谨慎操作      
```

### 删除镜像

```shell
[ceph: root@ceph1 /]#rbd rm img1
```

## 客户端

> - 客户端使用ceph块存储需要解决的问题：
>   - 怎么用？装软件 ceph-common
>   - ceph集群在哪？通过配置文件说明集群地址 
>   - 权限。keyring文件

### 安装客户端

```shell
[root@client1 ~]# yum -y install ceph-common
[root@client1 ~]#rbd ls #报错
```

### 集群密钥文件

```shell
[root@ceph1 ~]#cat /etc/ceph/ceph.client.admin.keyring
[root@ceph1 ~]#scp /etc/ceph/ceph.client.admin.keyring /etc/ceph/ceph.conf 192.168.88.10:/etc/ceph # 将配置拷贝到客户端
[root@client1 ~]#rbd ls # 不报错
```

> ceph1机器为主节点

### 创建镜像

```shell
[root@client1 ~]#rdb create img1 --size 10G
[root@client1 ~]#rbd ls
img1
```

### 映射

> 将ceph镜像映射为本地硬盘

```shell
[root@client1 ~]#lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    253:0    0  20G  0 disk 
└─vda1 253:1    0  20G  0 part /
[root@client1 ~]#rbd map img1 # 映射到本地，rbd为固定名称，0是编号
/dev/rbd0
[root@client1 ~]#lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
rbd0   252:0    0  10G  0 disk 
vda    253:0    0  20G  0 disk 
└─vda1 253:1    0  20G  0 part /
[root@client1 ~]# rbd showmapped # 镜像img1映射为了本地硬盘rbd0
id  pool  namespace  image  snap  device   
0   rbd              img1   -     /dev/rbd0
```

### 格式化使用

```shell
[root@client1 ~]# mkdir /data 
[root@client1 ~]# mkfs.xfs /dev/rbd0 # 格式化
[root@client1 ~]# mount /dev/rbd0 /data/ # 挂载到data目录
[root@client1 ~]# df -h /data/ # 查看挂载情况
Filesystem      Size  Used Avail Use% Mounted on
/dev/rbd0        10G  105M  9.9G   2% /data
[root@client1 ~]# cp /etc/hosts /data/ # 测试使用
[root@client1 ~]# ls /data/
hosts
```

### 删除

```shell
[root@client1 ~]# rbd status img1  # 查看谁正在使用
Watchers:
        watcher=192.168.88.10:0/2774412581 client.44243 cookie=18446462598732840961
        
# 按以下步骤删除img1
[root@client1 ~]# umount /dev/rbd0 # 卸载
[root@client1 ~]# rbd unmap img1 # 取消映射
[root@client1 ~]# rbd rm img1 # 删除镜像
Removing image: 100% complete...done.
```



# 快捷键



# 问题

## 什么是块存储？

> 1. 块存储是一种数据存储方式，它将数据分成固定大小的块，每个块都有一个唯一的地址。
> 2. 块存储通常用于存储大型文件或数据库，因为它可以更有效地管理和访问大量数据。
> 3. 块存储中，数据被分成块，每个块都有一个唯一的地址。这些块可以被存储在不同的位置，例如硬盘、网络存储设备或云存储中。

## 什么是文件系统？

> 1. 文件系统存储是一种数据存储方式，它将数据组织成文件和目录的形式，以便于用户访问和管理。
> 2. 文件系统存储通常用于存储小型文件和文档，例如文本文件、图片和音频文件等。在文件系统存储中，数据被组织成文件和目录的形式，每个文件都有一个唯一的文件名和路径。
> 3. 文件系统存储通常使用一些特殊的算法来管理数据，例如ext4、xfs、nfs。这些算法可以提高数据的可靠性和可用性，同时也可以提高数据的访问速度和效率。

## 什么是对象存储？

> 1. 对象存储是一种数据存储方式，它将数据组织成对象的形式，每个对象都有一个唯一的标识符和元数据。
> 2. 对象存储通常用于存储大型文件和非结构化数据，例如视频、音频、图片和文档等。
> 3. 在对象存储中，数据被组织成对象的形式，每个对象都有一个唯一的标识符和元数据，例如文件名、文件类型、创建时间和修改时间等。
> 4. 对象存储通常使用一些特殊的算法来管理数据，这些算法可以提高数据的可靠性和可用性，同时也可以提高数据的访问速度和效率。

## ceph有哪些组成部分？

> 1. Ceph Monitor：负责监控Ceph集群中每个OSD（对象存储设备）和MDS（元数据服务），并进行监控、故障检测和状态报告。
>
> 2. Ceph OSD：负责存储和检索数据，OSD将数据存储为对象，并管理对象的副本。
>
> 3. Ceph MDS：负责管理文件系统的元数据，例如目录结构、文件属性等。
>
> 4. Ceph RGW：RGW（RADOS Gateway）是一种对象存储网关，允许应用程序通过RESTful接口访问Ceph集群中的对象存储池。
>
> 5. 管理器：Ceph Manager（ceph-mgr）负责跟踪ceph集群的运行时指标和当前状态，包括存储利用率、当前性能指标和系统负载。
>
> Ceph还包括一些附加组件，如CephFS（Ceph文件系统）、RBD（块存储设备）等，这些组件可以扩展Ceph的功能和使用场景。

## cephadm有哪些作用？

> 1. cephadm是一个Ceph管理工具，它可以帮助用户快速、简便地部署和管理Ceph集
> 2. cephadm可以自动化地完成Ceph集群的安装、配置、升级和维护等任务，同时还提供了一些方便的命令和工具，例如ceph orch命令和cephadm shell命令等。
> 3. 使用cephadm可以大大简化Ceph集群的管理工作，提高管理效率和可靠性。

## 在配置ceph时，为什么需要ntp？

> 1. 在Ceph集群中，所有节点的时钟必须保持同步，以确保数据的一致性和可靠性。因此，配置Ceph集群时需要使用NTP（网络时间协议）来同步所有节点的时钟。
> 2. NTP可以确保所有节点的时钟都与一个可靠的时间源保持同步，从而避免数据不一致和其他问题。在Ceph集群中，通常使用一个或多个NTP服务器来同步所有节点的时钟。

## 在配置ceph时，需要使用容器，容器仓库服务器的作用是什么？

> 1. 容器仓库服务器是用来存储和分发容器镜像的服务器。它可以让用户方便地管理和分享自己的镜像，也可以从中获取他人分享的镜像。
> 2. 常见的容器仓库服务器有Docker Hub、阿里云容器镜像服务、腾讯云容器镜像服务等，也可以搭建自己的仓库服务器。在使用容器技术时，容器仓库服务器是非常重要的一环，它可以提高开发效率，降低运维成本。

## 容器和镜像的关系是什么？

> 1. 容器和镜像是容器技术中的两个重要概念。镜像是容器的基础，它是一个只读的模板，包含了运行容器所需的所有文件和配置信息。
> 2. 容器则是镜像的一个可运行实例，它可以被启动、停止、删除等。在启动容器时，Docker会在镜像的基础上创建一个可写的容器层，容器层中的所有修改都只会影响到当前容器，不会影响到镜像本身。因此，容器可以看作是镜像的一个运行时状态。

## 容器和虚拟机有什么异同？

> 1. 容器和虚拟机都是用来实现资源隔离的技术，但它们的实现方式有所不同。
>
>    ​	虚拟机是通过在物理服务器上运行一个虚拟化层来实现资源隔离的，每个虚拟机都有自己的操作系统和内核，因此虚拟机之间的隔离性非常好。
>
>    ​	而容器则是通过在宿主机上运行一个容器引擎来实现资源隔离的，容器之间共享宿主机的操作系统和内核，因此容器之间的隔离性相对较差。
>
> 2. 虚拟机的隔离性更好，但启动和运行虚拟机需要更多的资源，而容器的启动和运行则更加轻量级和快速。

## ceph mon部署时，有哪些注意事项？

> 1. 确保每个mon节点的主机名和IP地址都正确无误，否则会导致mon无法正常启动。
>
> 2. 在部署mon时，需要指定mon节点的数量，一般建议至少部署3个mon节点，以保证系统的高可用性。奇数部署
>
> 3. 在部署mon时，需要指定mon节点的认证方式，可以选择使用cephx认证或者无认证方式。建议使用cephx认证，以提高系统的安全性。

## ceph存储池的作用是什么？

> 1. ceph存储池是ceph存储集群中的一个重要概念，它用于存储和管理数据。
> 2. 在ceph中，存储池可以看作是一个逻辑上的容器，它可以包含多个对象，每个对象可以是一个文件、一个块设备或者一个对象。
> 3. 存储池可以设置不同的副本数、存储策略和访问权限等属性，以满足不同的业务需求。通过使用存储池，用户可以方便地管理和维护ceph存储集群中的数据，提高数据的可靠性和可用性。

## ceph存储池使用的PG是什么？有什么作用？

> PG是什么：
>
> ​	用于实现数据的分布和复制。PG的全称是Placement Group，即放置组。每个PG包含一组对象，它们被分布在ceph存储集群中的不同OSD上。PG的数量和大小可以根据实际需求进行调整，以满足不同的业务需求
>
> 作用：
>
> 1. 实现数据的分布和复制：PG可以将数据分布到不同的OSD上，以实现数据的负载均衡和高可用性。同时，PG还可以通过复制机制来提高数据的可靠性和可用性。
>
> 2. 实现数据的恢复和重平衡：当某个OSD发生故障或者新增了OSD时，PG可以自动进行数据的恢复和重平衡，以保证系统的稳定性和可靠性。
>
> 3. 实现数据的快速定位和访问：PG可以将数据分布到不同的OSD上，以实现数据的快速定位和访问。同时，PG还可以通过缓存机制来提高数据的访问速度和效率。

## rbd镜像作用是什么？

> rbd镜像是ceph存储集群中的一个重要概念，它用于存储和管理块设备。rbd镜像可以看作是一个虚拟的块设备，它可以被挂载到客户端上，用于存储和管理数据。
>
> 1. 提供块设备存储：rbd镜像可以提供块设备级别的存储，可以被客户端挂载为块设备，用于存储和管理数据。
>
> 2. 支持快照和克隆：rbd镜像支持快照和克隆功能，可以方便地进行数据备份和恢复。
>
> 3. 支持动态扩容和缩容：rbd镜像支持动态扩容和缩容，可以根据实际需求进行容量的调整。
>
> 3. 支持多种接口：rbd镜像支持多种接口，如librbd、rbd-nbd、rbd-fuse等，可以满足不同的业务需求。

## ceph客户端如何配置？

> 安装ceph客户端软件：在客户端上安装ceph客户端软件ceph-common，以便于访问ceph存储集群。
>
> 配置ceph.conf文件：在客户端上配置ceph.conf文件，指定ceph存储集群的相关信息，如mon节点的IP地址、认证方式等。
>
> 配置ceph.client.keyring文件：在客户端上配置ceph.client.keyring文件，用于认证客户端的身份。
>
> 挂载rbd镜像：如果要使用rbd镜像，需要在客户端上挂载rbd镜像，以便于访问和使用。

# 补充

1. 更新yum仓库  yum --update xx
2. 更新yum缓存  yum makecache

# 今日总结



# 昨日复习
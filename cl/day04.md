- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [快照](#快照)
    - [准备操作](#准备操作)
    - [制作快照](#制作快照)
    - [恢复快照](#恢复快照)
    - [快照保护/取消](#快照保护取消)
    - [删除快照](#删除快照)
    - [删除镜像](#删除镜像)
  - [克隆](#克隆)
    - [准备操作](#准备操作-1)
    - [克隆镜像](#克隆镜像)
    - [使用克隆镜像](#使用克隆镜像)
    - [查看镜像详情](#查看镜像详情)
    - [合并父子镜像](#合并父子镜像)
    - [删除镜像](#删除镜像-1)
    - [查询独立镜像](#查询独立镜像)
  - [开机自动挂载](#开机自动挂载)
    - [创建镜像](#创建镜像)
    - [测试比较](#测试比较)
    - [设置开机自动挂载](#设置开机自动挂载)
  - [文件系统存储](#文件系统存储)
    - [创建存储池](#创建存储池)
    - [启用MDS服务](#启用mds服务)
    - [客户端使用cephfs](#客户端使用cephfs)
    - [配置开机自动挂载](#配置开机自动挂载)
  - [对象存储](#对象存储)
    - [配置服务器端](#配置服务器端)
    - [配置客户端](#配置客户端)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
  - [复制一个文件夹中多个文件](#复制一个文件夹中多个文件)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)


# 学习目标

块存储应用案例

分布式文件系统

对象存储

# 课堂笔记（命令）



# 课堂笔记（文本）

## 快照

> - 快照可以保存某一时间点时的状态数据
> - 快照是映像在特定时间点的只读逻辑副本
> - 希望回到以前的一个状态，可以恢复快照
> - 使用镜像、快照综合示例

### 准备操作

> 1. 创建镜像 rdb create img1 --size 10G
> 2. 映射到客户端 rbd map img1
> 3. 格式化 mkfs.xfs /dev/rdb0
> 4. 挂载 mount /dev/rdb0 /mnt
> 5. 写入数据 cp /etc/{hosts,passwd} /mnt

### 制作快照

```shell
[root@client1 ~]# rbd snap create img1 --snap img1-sn1 # 制作快照名为 img1-sn1
Creating snap: 100% complete...done.
[root@client1 ~]# rbd snap ls img1 # 查询基于img1的快照
SNAPID  NAME      SIZE    PROTECTED  TIMESTAMP               
     4  img1-sn1  10 GiB             Sat Dec 17 10:46:07 2022
```

### 恢复快照

```shell
# 删除/mnt/中的数据
[root@client1 ~]# rm -f /mnt/*

# 通过快照还原数据
[root@client1 ~]# umount /mnt/
[root@client1 ~]# rbd unmap /dev/rbd0
# 回滚img1到快照img1-sn1
[root@client1 ~]# rbd snap rollback img1 --snap img1-sn1
# 重新挂载
[root@client1 ~]# rbd map img1
/dev/rbd0
[root@client1 ~]# mount /dev/rbd0 /mnt/
[root@client1 ~]# ls /mnt/   # 数据还原完成
hosts  passwd
```

### 快照保护/取消

```shell
[root@client1 ~]# rbd snap protect img1 --snap img1-sn1 # 保护快照
[root@client1 ~]# rbd snap rm img1 --snap img1-sn1 # 无法删除
Removing snap: 0% complete...failed.
rbd: snapshot 'img1-sn1' is protected from removal.
2023-12-27T11:37:45.294+0800 7f31da9b6700 -1 librbd::Operations: snapshot is protected

[root@client1 ~]# rbd snap unprotect img1 --snap img1-sn1 # 取消保护
```

### 删除快照

```shell
[root@client1 ~]# rbd snap rm img1 --snap img1-sn1 # 成功删除
```

### 删除镜像

```shell
# 前提先删除镜像存在的快照
[root@client1 ~]#unmout /mnt
[root@client1 ~]#rdb unmap # 取消映射
[root@client1 ~]#rdb rm img1 # 删除镜像
```

## 克隆

> - 不能将一个镜像同时挂载到多个节点，如果这样操作，将会损坏数据
> - 如果希望不同的节点，拥有完全相同的数据盘，可以使用克隆技术
> - 克隆是基于快照的，不能直接对镜像克隆
> - 快照必须是受保护的快照，才能克隆
> - 克隆流程

### 准备操作

> 1. 创建镜像 rdb create img1 --size 10G
> 2. 映射到客户端 rbd map img1
> 3. 格式化 mkfs.xfs /dev/rdb0
> 4. 挂载 mount /dev/rdb0 /mnt
> 5. 写入数据 for i in {1..20};do echo "Hello World $i" > /mnt/file$i.txt;done
> 6. 卸载 umount /mnt
> 7. 取消映射 rbd unmap img1
> 8. 制作快照 rbd snap create img1 --snap img1-sn1
> 9. 保护快照 rbd snap protect img1 --snap img1-sn1

### 克隆镜像

```shell
[root@client1 ~]# rbd snap ls img1  # 查询基于img1的快照
SNAPID  NAME      SIZE    PROTECTED  TIMESTAMP               
     6  img1-sn1  10 GiB             Wed Dec 27 14:16:27 2023
     
# 基于快照img1-sn1克隆镜像img1-sn1-1     
[root@client1 ~]# rbd clone  img1 --snap img1-sn1 img1-sn1-1
# 基于快照img1-sn1克隆镜像img1-sn1-2
[root@client1 ~]# rbd clone  img1 --snap img1-sn1 img1-sn1-2
[root@client1 ~]# rbd ls
img1
img1-sn1-1
img1-sn1-2
```

### 使用克隆镜像

```shell
[root@client1 ~]# rbd map img1-sn1-1
[root@client1 ~]# mkdir data
[root@client1 ~]# mount /dev/rbd0 data
[root@client1 ~]# ls data
file10.txt  file14.txt  file18.txt  file2.txt  file6.txt
file11.txt  file15.txt  file19.txt  file3.txt  file7.txt
file12.txt  file16.txt  file1.txt   file4.txt  file8.txt
file13.txt  file17.txt  file20.txt  file5.txt  file9.txt
```

### 查看镜像详情

```shell
[root@client1 ~]# rbd info img1 --snap img1-sn1 # 查询快照详情
rbd image 'img1':
        size 10 GiB in 2560 objects
        order 22 (4 MiB objects)
        snapshot_count: 1
        id: adb0b7a72cc0
        block_name_prefix: rbd_data.adb0b7a72cc0
        format: 2
        features: layering, exclusive-lock, object-map, fast-diff, deep-flatten
        op_features: 
        flags: 
        create_timestamp: Wed Dec 27 14:07:31 2023
        access_timestamp: Wed Dec 27 14:37:43 2023
        modify_timestamp: Wed Dec 27 14:07:31 2023
        protected: True  # 受保护
[root@client1 ~]# rbd info img1-sn1-1
rbd image 'img1-sn1-1':
        size 10 GiB in 2560 objects
        order 22 (4 MiB objects)
        snapshot_count: 0
        id: adc15f6a330
        block_name_prefix: rbd_data.adc15f6a330
        format: 2
        features: layering, exclusive-lock, object-map, fast-diff, deep-flatten
        op_features: 
        flags: 
        create_timestamp: Wed Dec 27 14:20:03 2023
        access_timestamp: Wed Dec 27 14:20:03 2023
        modify_timestamp: Wed Dec 27 14:20:03 2023
        parent: rbd/img1@img1-sn1  # 父对象是rbd池中img1镜像的img1-sn1快照
        overlap: 10 GiB
```

### 合并父子镜像

> - img1-sn1-2是基于img1的快照克隆来的，不能独立使用。
> - 如果父镜像删除了，子镜像也无法使用。
> - 将父镜像内容合并到子镜像中，子镜像就可以独立使用了。

```shell
# 把img1的数据合并到子镜像img1-sn1-1中
[root@client1 ~]# rbd flatten img1-sn1-1
# 查看状态，它就没有父镜像了
[root@client1 ~]# rbd info img1-sn1-1
rbd image 'img1-sn1-1':
        size 10 GiB in 2560 objects
        order 22 (4 MiB objects)
        snapshot_count: 0
        id: adaebfe090ce
        block_name_prefix: rbd_data.adaebfe090ce
        format: 2
        features: layering, exclusive-lock, object-map, fast-diff, deep-flatten
        op_features: 
        flags: 
        create_timestamp: Wed Dec 27 14:19:59 2023
        access_timestamp: Wed Dec 27 14:19:59 2023
        modify_timestamp: Wed Dec 27 14:19:59 2023
```

### 删除镜像

```shell
# 删除父镜像，如果镜像正在被使用，则先取消
[root@client1 ~]# umount /data/
[root@client1 ~]# rbd unmap img1-sn1-2
# 1. 删除镜像img1-sn1-2
[root@client1 ~]# rbd rm img1-sn1-2
# 2. 取消img1-sn1-2的保护
[root@client1 ~]# rbd snap unprotect img1 --snap img1-sn1
# 3. 删除img1-sn1快照
[root@client1 ~]# rbd snap rm img1 --snap img1-sn1
# 4. 删除img1
[root@client1 ~]# rbd rm img1
```

### 查询独立镜像

```shell
# 因为img1-sn1-1已经是独立的镜像了，所以它还可以使用
# ceph1上的镜像没有受到影响
[root@client1 ~]# cat /data/file1.txt 
Hello World 1
```

## 开机自动挂载

### 创建镜像

```shell
[root@client1 ~]#rbd create img1 --size 10G
[root@client1 ~]#rbd create imga --size 10G
```

### 测试比较

```shell
[root@client1 ~]#rbd map img1
[root@client1 ~]#rbd map imga
[root@client1 ~]# ll /dev/rbd/rbd/
total 0
lrwxrwxrwx 1 root root 10 Dec 27 15:26 img1 -> ../../rbd0
lrwxrwxrwx 1 root root 10 Dec 27 15:26 imga -> ../../rbd1

[root@client1 ~]#rbd unmap imga
[root@client1 ~]#rbd unmap img1
[root@client1 ~]#rbd map imga
[root@client1 ~]#rbd map img1
total 0
lrwxrwxrwx 1 root root 10 Dec 27 15:33 img1 -> ../../rbd1
lrwxrwxrwx 1 root root 10 Dec 27 15:33 imga -> ../../rbd0

# 执行映射顺序，影响分区编号
# /dev/rbd/rbd/img1始终存在只是指向的软链接随映射顺序变化

[root@client1 ~]#rbd unmap imga
[root@client1 ~]#rbd unmap img1
```

### 设置开机自动挂载

```shell
[root@client1 ~]#rbd map img1
[root@client1 ~]#vim /etc/ceph/rbdmap  # 指定挂载的镜像及用户名、密钥
rbd/img1        id=admin,keyring=/etc/ceph/ceph.client.admin.keyring
#poolname/imagename     id=client,keyring=/etc/ceph/ceph.client.keyring

[root@client1 ~]#mkdir /data
[root@client1 ~]# vim /etc/fstab  # 追加
/dev/rbd/rbd/img1   /data   xfs    noauto  0  0
# noauto的意思是，等rbdmap服务启动后，再执行挂载


# 启动rbdmap服务
[root@client1 ~]# systemctl enable rbdmap --now
# rbdmap服务的作用是将Ceph分布式文件系统中的RBD映射为本地块设备，方便客户端应用程序使用分布式存储集群中的存储资源，类似开机自动 rbd map xx

[root@client1 ~]# df -h /data/
Filesystem      Size  Used Avail Use% Mounted on
/dev/rbd0        10G  105M  9.9G   2% /data
```

## 文件系统存储

> - 文件系统：相当于是组织数据存储的方式。
> - 格式化时，就是在为存储创建文件系统。
> - Linux对ceph有很好的支持，可以把ceph文件系统直接挂载到本地。
> - 要想实现文件系统的数据存储方式，需要有MDS组件

### 创建存储池

```shell
# 1. 新建一个名为data1的存储池，目的是存储数据，有100个PG
[root@client1 ~]# ceph osd pool create data01 100

# 2. 新建一个名为metadata1的存储池，目的是存储元数据
[root@client1 ~]# ceph osd pool create metadata01 100

# 3. 创建名为myfs1的cephfs，数据保存到data1中，元数据保存到metadata1中
[root@client1 ~]# ceph fs new myfs01 metadata01 data01

# 4. 查看存储池
[root@client1 ~]# ceph osd lspools
1 .mgr
2 rbd
3 data01
4 metadata01
[root@client1 ~]# ceph df
--- RAW STORAGE ---
CLASS     SIZE    AVAIL     USED  RAW USED  %RAW USED
hdd    180 GiB  180 GiB  324 MiB   324 MiB       0.18
TOTAL  180 GiB  180 GiB  324 MiB   324 MiB       0.18
 
--- POOLS ---
POOL        ID  PGS   STORED  OBJECTS     USED  %USED  MAX AVAIL
.mgr         1    1  449 KiB        2  1.3 MiB      0     57 GiB
rbd          3   32  7.3 MiB       46   22 MiB   0.01     57 GiB
data01       4   63      0 B        0      0 B      0     57 GiB
metadata01   5   70  2.3 KiB       22   96 KiB      0     57 GiB

# 5. 查看文件系统
[root@client1 ~]# ceph fs ls
name: myfs01, metadata pool: metadata01, data pools: [data01 ]
```

### 启用MDS服务

```shell
# 通过容器启动
[root@client1 ~]# ceph orch apply mds myfs01 --placement="2 ceph1 ceph2 ceph3" # 需要两台设备，需从这三台中选择

[root@client1 ~]# ceph -s
  cluster:
    id:     681788a8-a3c5-11ee-bdd0-5254005ec665
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph1,ceph2,ceph3 (age 7h)
    mgr: ceph2.xgwntz(active, since 7h), standbys: ceph1.slqbyl
    mds: 1/1 daemons up, 1 standby
    osd: 9 osds: 9 up (since 7h), 9 in (since 23h)
...略...
```

### 客户端使用cephfs

```shell
# 挂载文件系统需要密码。查看密码
[root@client1 ~]# cat /etc/ceph/ceph.client.admin.keyring 
[client.admin]
    key = AQBmhINh1IZjHBAAvgk8m/FhyLiH4DCCrnrdPQ==

# -t 指定文件系统类型。-o是选项，提供用户名和密码
[root@client1 ~]# mkdir /mydata
[root@client1 ~]# mount.ceph 192.168.88.11,192.168.88.12,192.168.88.13:/ /mydata -o name=admin,secret=AQC5u5ZjnTA1ERAAruLAI8F1W1nyOgxZSx0UXw== 
[root@client1 ~]# df -h /mydata/
Filesystem       Size  Used Avail Use% Mounted on
192.168.88.11,192.168.88.12,192.168.88.13:/   57G     0   57G   0% /mydata
```

### 配置开机自动挂载

```shell
[root@client1 ~]# vim /etc/fstab
192.168.88.11,192.168.88.12,192.168.88.13:/ /mydata ceph name=admin,secret=AQAYiYplOGTGOhAAIxO3VzvAV2+24i3lzs1ZHA== 0 0

# 源    挂载点  文件系统  选项  0 0
[root@client1 ~]# echo "mount -a" >> /etc/rc.d/rc.loacl # 添加一个开机自动运行mount -a 命令，因为上述配置了开机自动挂载后，重启无效，需要手动执行mount -a
```

## 对象存储

### 配置服务器端

> - 需要专门的客户端访问
> - 键值对存储方式
> - 对象存储需要rgw组件
> - 安装部署

```shell
# 1. 在ceph1/ceph2上部署rgw服务，名为myrgw
[root@client1 ~]# ceph orch apply rgw myrgw --placement="2 ceph1 ceph2" --port 8080
[root@client1 ~]# ceph -s
  cluster:
    id:     a4b69ab4-79dd-11ed-ae7b-000c2953b002
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph1,ceph3,ceph2 (age 101m)
    mgr: ceph1.gmqorm(active, since 6h), standbys: ceph3.giqaph
    mds: 1/1 daemons up, 1 standby
    osd: 9 osds: 9 up (since 6h), 9 in (since 5d); 1 remapped pgs
    rgw: 2 daemons active (2 hosts, 1 zones)  # rgw信息
...略...
```

### 配置客户端

> - ceph对象存储提供了一个与亚马逊S3（Amazon Simple Storage Service）兼容的接口
> - 在S3中，对象被存储在一个称作桶（bucket）的器皿中。这就好像是本地文件存储在目录中一样。

```shell
# 创建用户可以访问dashborad页面进行操作，点击object gateway-user 进行创建	
# 1. 安装amazon S3 cli工具（客户端工具）
[root@client1 ~]# yum install -y awscli

# 2. 在ceph中创建一个用户
[root@client1 ~]# radosgw-admin user create --uid=testuser --display-name="Test User" --email=test@tedu.cn --access-key=12345 --secret=67890

# 3. 初始化客户端
[root@client1 ~]# aws configure --profile=ceph
AWS Access Key ID [None]: 12345
AWS Secret Access Key [None]: 67890
Default region name [None]:   # 回车
Default output format [None]: # 回车

# 4. 创建名为testbucket的bucket，用于存储数据
[root@client1 ~]# vim /etc/hosts  # 添加以下内容
192.168.88.11 ceph1
192.168.88.12 ceph2
192.168.88.13 ceph3
[root@client1 ~]# aws --profile=ceph --endpoint=http://ceph1:8080 s3 mb s3://testbucket

# 5. 上传文件
[root@client1 ~]# aws --profile=ceph --endpoint=http://ceph1:8080 --acl=public-read-write s3 cp /etc/hosts s3://testbucket/hosts.txt

# 6. 查看bucket中的数据
[root@client1 ~]# aws --profile=ceph --endpoint=http://ceph1:8080 s3 ls s3://testbucket
2022-12-17 17:05:58        241 hosts.txt

# 7. 下载数据
[root@client1 ~]# wget -O zhuji http://ceph1:8080/testbucket/hosts.txt
```

# 快捷键



# 问题



# 补充

## 复制一个文件夹中多个文件

```shell
cp /etc/{hosts,passwd} /mnt
```



# 今日总结



# 昨日复习
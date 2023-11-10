- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
  - [lsblk](#lsblk)
  - [fdisk](#fdisk)
    - [MBR创建](#mbr创建)
    - [GPT创建](#gpt创建)
  - [mkfs](#mkfs)
  - [blkid](#blkid)
  - [mount](#mount)
    - [临时挂载-mbr/gpt](#临时挂载-mbrgpt)
    - [开机自动挂载-mbr/gpt](#开机自动挂载-mbrgpt)
  - [df -h](#df--h)
  - [parted](#parted)
  - [swapon](#swapon)
    - [临时设置](#临时设置)
    - [开机自动挂载](#开机自动挂载)
- [课堂笔记（文本）](#课堂笔记文本)
  - [硬盘分区思路](#硬盘分区思路)
  - [分区规划](#分区规划)
    - [MBR分区模式](#mbr分区模式)
    - [GPT分区模式](#gpt分区模式)
    - [交换空间（虚拟内存）](#交换空间虚拟内存)
- [快捷键](#快捷键)
- [问题](#问题)
- [总结](#总结)
  - [/etc/fstab文件有误：修复办法](#etcfstab文件有误修复办法)
  - [分区总结](#分区总结)

# 学习目标

磁盘空间管理

交换空间

# 课堂笔记（命令）

## lsblk

作用：列出当前系统识别的硬盘

```linux
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0          11:0    1 1024M  0 rom  
vda         252:0    0   20G  0 disk 
├─vda1      252:1    0    1G  0 part /boot
└─vda2      252:2    0   19G  0 part 
  ├─rl-root 253:0    0   17G  0 lvm  /
  └─rl-swap 253:1    0    2G  0 lvm  [SWAP]
vdb         252:16   0   20G  0 disk 
├─vdb1      252:17   0    2G  0 part /mypart1
└─vdb2      252:18   0    1G  0 part 
vdc         252:32   0   20G  0 disk 

```

## fdisk

作用：用来划分区模式

格式：fdisk /硬盘

常见指令：

- m 列出指令帮助
- p 查看现有的分区表（存放分区信息的表格）
- n 新建分区
- d 删除分区
- q 放弃更改并退出
- w 保存更改并退出

默认是MBR分区创建模式

例：以下将80G硬盘分成3个分区分别为10G1个扩展分区2个10G的逻辑分区

```linux
fdisk /dev/vdb  # 进入分区模式
命令(输入 m 获取帮助)：p
Disk /dev/vdb：80 GiB，85899345920 字节，167772160 个扇区
单元：扇区 / 1 * 512 = 512 字节
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x3155fff0

# 新分一个10G
命令(输入 m 获取帮助)：n
分区类型
   p   主分区 (0个主分区，0个扩展分区，4空闲)
   e   扩展分区 (逻辑分区容器)
选择 (默认 p)：   # 敲击回车

将使用默认回应 p。
分区号 (1-4, 默认  1):  # 敲击回车
第一个扇区 (2048-167772159, 默认 2048): # 敲击回车
上个扇区，+sectors 或 +size{K,M,G,T,P} (2048-167772159, 默认 167772159): +10G  # 输入+10G

创建了一个新分区 1，类型为“Linux”，大小为 10 GiB。

# 创建三个分区后
命令(输入 m 获取帮助)：n
分区类型
   p   主分区 (3个主分区，0个扩展分区，1空闲)
   e   扩展分区 (逻辑分区容器)
选择 (默认 e)：  #  默认敲击回车，创建逻辑分区

将使用默认回应 e。
已选择分区 4
第一个扇区 (62916608-167772159, 默认 62916608): 
上个扇区，+sectors 或 +size{K,M,G,T,P} (62916608-167772159, 默认 167772159): 

创建了一个新分区 4，类型为“Extended”，大小为 50 GiB。
命令(输入 m 获取帮助)：n  # 以下创建扩展分区
所有主分区都在使用中。
添加逻辑分区 5
第一个扇区 (62918656-167772159, 默认 62918656): 
上个扇区，+sectors 或 +size{K,M,G,T,P} (62918656-167772159, 默认 167772159): +20G

创建了一个新分区 5，类型为“Linux”，大小为 20 GiB。

命令(输入 m 获取帮助)：n
所有主分区都在使用中。
添加逻辑分区 6
第一个扇区 (104863744-167772159, 默认 104863744): 
上个扇区，+sectors 或 +size{K,M,G,T,P} (104863744-167772159, 默认 167772159): +20G

创建了一个新分区 6，类型为“Linux”，大小为 20 GiB。
命令(输入 m 获取帮助)：p
Disk /dev/vdb：80 GiB，85899345920 字节，167772160 个扇区
单元：扇区 / 1 * 512 = 512 字节
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x3155fff0

设备       启动      起点      末尾      扇区 大小 Id 类型
/dev/vdb1            2048  20973567  20971520  10G 83 Linux
/dev/vdb2        20973568  41945087  20971520  10G 83 Linux
/dev/vdb3        41945088  62916607  20971520  10G 83 Linux
/dev/vdb4        62916608 167772159 104855552  50G  5 扩展
/dev/vdb5        62918656 104861695  41943040  20G 83 Linux
/dev/vdb6       104863744 146806783  41943040  20G 83 Linux
```



### MBR创建

```linux
[root@localhost ~]# fdisk    /dev/vdb   
n 创建新的分区----->分区类型 回车----->分区编号 回车---->起始扇区 回车----->在last结束时 +2G
p 查看分区表
n 创建新的分区----->分区类型 回车----->分区编号 回车---->起始扇区 回车----->在last结束时 +1G
w 保存并退出  
```

代码解释：默认为mbr分区



### GPT创建

前提：虚拟机已添加了块磁盘

```linux
fdisk /dev/vdd
欢迎使用 fdisk (util-linux 2.32.1)。
更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。
设备不包含可识别的分区表。
创建了一个磁盘标识符为 0x028b0bce 的新 DOS 磁盘标签。
命令(输入 m 获取帮助)：g
已创建新的 GPT 磁盘标签(GUID: 06F98712-6E1A-5846-B779-2CB506F67A34)。
命令(输入 m 获取帮助)：n
分区号 (1-128, 默认  1): 
第一个扇区 (2048-41943006, 默认 2048): 
上个扇区，+sectors 或 +size{K,M,G,T,P} (2048-41943006, 默认 41943006): +2G
创建了一个新分区 1，类型为“Linux filesystem”，大小为 2 GiB。
命令(输入 m 获取帮助)：n 
```

代码解释：g代表创建gpt分区模式，

## mkfs

作用：格式化文件系统

格式：mkfs.格式化文件系统类型   /硬盘

mkfs 支持格式 ext4和xfs文件系统

```linux
# ext4
mkfs.ext4 /dev/vdb1
mke2fs 1.45.6 (20-Mar-2020)
正在分配组表： 完成                            
正在写入inode表： 完成                            
创建日志（16384 个块）完成
写入超级块和文件系统账户统计信息： 已完成
# xfs
mkfs.xfs /dev/vdc1
```

代码解释：

将vdb1分区格式成ext4文件系统



## blkid

查询分区类型

```linux
mkfs.ext4 /dev/vdb1 # 格式化文件系统ext4
blkid /dev/vdb1 # 查看文件系统类型
/dev/vdb1: UUID="02ba7640-f423-474c-b48f-826c994dceeb" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="76c0241f-01"
```

代码解释：主要查看TYPE的内容

## mount

### 临时挂载-mbr/gpt

挂载上诉创建的磁盘

```linux
mkdir /mypart1
mount /dev/vdb1 /mypart1
df -h # 显示正在挂载的设备使用情况
```

### 开机自动挂载-mbr/gpt

编写格式：

设备路径 挂载点 文件系统类型 参数（默认defaults） 备份标记（0关闭,1开启）  检测顺序

```linux
vim /etc/fstab
# 进入命令模式按住大写G，再按o进入插入模式，输入以下内容
/dev/vdb1 /mypart1 ext4 defaults 0 0
# wq保存并退出
mount -a # 检测书写是否正确，无输出表示没问题
df -h|grep mypart1 # 查看挂载信息
```

[注]：前提需要先进行临时挂载，再来进行编写自动挂载，不然会有报错

## df -h

作用：显示挂载社保的使用情况

```linux
df -h
devtmpfs             625M     0  625M    0% /dev
tmpfs                655M     0  655M    0% /dev/shm
tmpfs                655M  9.3M  645M    2% /run
tmpfs                655M     0  655M    0% /sys/fs/cgroup
/dev/mapper/rl-root   17G  5.1G   12G   30% /
/dev/vda1           1014M  255M  760M   26% /boot
tmpfs                131M   24K  131M    1% /run/user/0
/dev/vdb1            2.0G  6.0M  1.8G    1% /mypart1
```

代码解释：主要查看磁盘大小以及挂载信息

## parted

查看磁盘信息

```linux
parted /dev/vdc print

Model: Virtio Block Device (virtblk)
Disk /dev/vdc: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start   End     Size    Type      File system     标志
 1      1049kB  2149MB  2147MB  primary   linux-swap(v1)
 2      2149MB  4296MB  2147MB  primary
 3      4296MB  6443MB  2147MB  primary
 4      6443MB  8591MB  2147MB  extended
 5      6445MB  8591MB  2146MB  logical   xfs
```

代码解释：主分区（primary）、扩展分区（extended）、逻辑分区（logical）

## swapon

作用：设置或查看交换空间

交换空间：虚拟内存

swapon 启用

swapoff 停用

[-a]：检测交换区（前提设置了开机自动启用交换空间）

### 临时设置

```linux
mkswap /dev/vdc1 # 格式化交换文件系统
正在设置交换空间版本 1，大小 = 2 GiB (2147479552  个字节)
无标签，UUID=eae60c01-5a57-4c1e-a5c2-737b3eebc667

blkid /dev/vdc1 # 查看文件系统类型
/dev/vdc1: UUID="eae60c01-5a57-4c1e-a5c2-737b3eebc667" TYPE="swap" PARTUUID="437cae44-01"

swapon # 查看交换空间组成的成员信息
NAME      TYPE      SIZE   USED PRIO
/dev/dm-1 partition   2G 162.5M   -2

swapon /dev/vdc1 # 启用交换分区
swapon # 查看成员信息
fre -h # 查看交换空间大小

swapoff /dev/vdc1 # 停用交换分区
```

### 开机自动挂载

```linux
vim /etc/fstab # 增加一组信息
/dev/vdc1 swap swap defaults 0 0
swapon # 查看交换空间组成的成员信息
swapon -a # 专门检测交换分区的书写
swapon
```

# 课堂笔记（文本）

 ## 硬盘分区思路

识别硬盘-》分区规划-》格式化-》挂载使用

lsblk-fdisk-mkfs-mount /etc/fstab

## 分区规划

分区方案：MBR与GPT

### MBR分区模式

–分区类型：主分区、扩展分区(占用所有剩余空间)、逻辑分区

–**最多只能有4个主分区**

–**扩展分区可以没有，至多有一个**

–1~4个主分区，或者 3个主分区+1个扩展分区（n个逻辑分区）

–最大支持容量为 2.2TB 的磁盘

–扩展分区不能格式化，空间不能直接存储数据

–可以用于存储数据的分区：主分区与逻辑分区

创建方式：参考fdisk-mbr创建

### GPT分区模式

•GPT，GUID Partition Table

–全局唯一标识分区表

–突破固定大小64字节的分区表限制

–可支持4个以上的主分区，最大支持18EB容量

1 EB = 1024 PB = 1024 x 1024 TB

创建方式：参考fdisk-gpt创建

### 交换空间（虚拟内存）

作用：当物理内存占满了，可以将内存的中数据，暂时放入交换空间中，缓解真实物理内存的压力

利用硬盘分区制作交换空间

创建方式：参考swapon

# 快捷键



# 问题

1. 扇区默认大小

   512字节 B

2. 磁盘->磁道（磁盘的一圈一圈）->扇区（磁道切割成一片一片）



# 总结

## /etc/fstab文件有误：修复办法

前因：/etc/fstab中配置信息错误

重启后进入命令界面，输入root密码

修改/etc/fstab文件内容，进行排查

> 1. 无法排查：注释配置的信息，保存，重启
> 2. 知道错误：修改错误，重启

## 分区总结

1. 识别硬盘 ：lsblk 查看识别的硬盘

2. 硬盘分区 ：fdisk命令 parted命令

3. 分区模式：MBR分区方案 GPT分区方案

4. 格式化 ：mkfs.ext4 mkfs.xfs blkid

5. 挂在使用：mount 手动挂载 与/etc/fstab开机自动挂载

6. 利用mount -a检测开机自动挂载

7. Linux中新硬盘经历那些步骤才能存档

   识别硬盘-》划分分区-》格式化-》挂载使用

8. 分区模式分为那两种

   MBR

   GPT

9. MBR常见的分区类型有那三种

   主分区、扩展分区、逻辑分区

10. 刷新分区表命令是

    partprobe
    
11. 格式化交换分区的命令是什么？启用交换分区的命令是什么？如何查看交换区成员？
    
mkswap
    
swapon
    
swapon
    
12. 查看文件戏台类型的命令

    blkid

13. 

    

    

    

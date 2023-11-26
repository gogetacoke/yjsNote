云计算系统管理 ——

01. IP地址组成及作用

地址组成（点分十进制）：一共32个二进制位，表示为4个十进制数，以 . 隔开

作用：用来标识一个节点（连网设备）的网络地址

02. 测试网络通信的命令

ping 对方IP地址

03. 简述Linux目录/、/boot、/home、/root、/bin、/dev、/etc的用途

/：整个Linux文件系统的根目录

/boot：存放系统内核、启动菜单配置等文件

/home：存放普通用户的默认家目录（同名子目录）

/root：管理员的家目录

/bin、/sbin：存放系统命令、可执行的程序

/dev：存放各种设备文件

/etc：存放各种系统配置、系统服务配置文件

04. 简述Linux命令cat、cd、ls、ifconfig、hostname、vim、grep、rm、cp、mv、mkdir、touch、tar、find、useradd、usermod、passwd、id、gpasswd、groupadd的用途

cat：查看文件文件内容（适合查看内容较少的文件）

cd：切换路径

ls：查看目录内容

ifconfig：查看网卡IP地址

hostname：查看及修改主机名

vim：修改文本文件内容

grep：在文本文件内容中，过滤关键字符串

rm：删除文件或目录

cp：复制

mv：移动

mkdir：创建目录

touch：创建文本文件

tar：制作归档

find：查找数据所在路径

useradd：创建用户

usermod：修改已存在用户属性

passwd：设置用户密码

id：查看用户基本信息

gpasswd：用户加入组

groupadd：创建组

05. vim的三个模式

命令模式、插入模式、末行模式

06. Linux默认解释器程序

/bin/bash

07. 通配符 *、?、[]、{}的作用

*：任意多个任意字符

?：单个字符

[a-z]：多个字符或连续范围中的一个，若无则忽略

{a,min,xy}：多组不同的字符串，全匹配

08. 红帽系列Linux查询rpm软件

rpm -qa

rpm -ql 软件名 #查询软件安装清单

rpm -qf 文件路径 #查询该文件由那个软件包产生的

09. 为红帽系列Linux主机指定可用的yum软件源

vim /etc/yum.repos.d/文件名.repo

[仓库id]

name = 仓库描述文字

baseurl = 仓库的访问地址

enabled = 1

gpgcheck = 0

10. 普通的系统用户账号

普通系统用户：属于本地账号，其登录名、密码存放在本系统的/etc/passwd、/etc/shadow等文件中

11. /etc/passwd七个字段

用户名:密码占位符X：UID：基本组GID：用户描述信息：家目录：登录系统解释器

12. 命令别名的作用，设置永久有效的命令别名

别名的作用：为需要频繁使用而又冗长的命令行建立一个更短、更好记的命令字

vim /etc/bashrc

alias 别名='实际命令行'

13. 使用crontab编辑计划任务时，每一条任务记录的格式组成

分钟 小时 日期 月份 星期 任务命令行

云计算应用管理 ——

01. 简述Linux命令chmod、chown、setfacl、getfacl、pstree、ps、top、pgrep、kill、killall、rsync、ssh、scp的用途

chmod：修改用户权限

chown：修改归属关系

setfacl：设置ACL策略

getfacl：查看ACL策略

pstree：查看进程树

ps：查看进程信息

top：动态查看进程信息

pgrep：检索进程

kill：按照PID杀死进程

killall：按照进程名杀死进程

rsync：同步数据

ssh：远程管理指令

scp：远程复制指令

02. 设置文档的访问权限时，数值777、755、700、644、600、000各自表示何种权限

777 ==》rwxrwxrwx

755 ==》rwxr-xr-x

700 ==》rwx------

644 ==》rw-r--r--

600 ==》rw-------

000 ==》---------

03. msdos分区模式的特点有哪些，如何使用fdisk工具调整硬盘的分区表

msdos分区模式：可以划分1~4个主分区，或者0~3个主分区+1个扩展分区（n个逻辑分区），操作的磁盘<2.2TB

fdisk /dev/磁盘名

主要操作指令：n 新建、d 删除、p 查看分区表、w 保存退出、q 不保存退出

04. LVM逻辑卷存储方案如何实现，其中主要命令工具的用法

LVM存储方案：1个或多个零散存储设备（物理卷） ==》整合为更大的虚拟磁盘（卷组） ==》从此虚拟磁盘内再划分出虚拟的分区（逻辑卷），主要优势：设备化零为整、容量动态伸缩

主要命令工具：

vgcreate 卷组名 物理存储设备...

vgextend 卷组名 新增加的物理存储设备...

lvcreate -L 大小 -n 逻辑卷名 卷组名

lvextend -L 新的大小 /dev/卷组名/逻辑卷名

05. 交换空间的作用是什么

交换空间可以使用一部分硬盘空间来模拟内存，缓解物理内存不足的问题

06. 使用nmcli创建一个新连接，并为其配置静态IP地址等参数

nmcli connection add con-name "连接名" ifname "接口名" type ethernet

nmcli connection modify "连接名" ipv4.method manual ipv4.address "IP地址/掩码长度" ipv4.gateway "默认网关"

nmcli connection modify "连接名" ipv4.dns DNS服务器地址 autoconnect yes

nmcli connection up "连接名"

07. 指定DNS服务器配置文件

/etc/resolv.conf

08. 红帽7系列防火墙服务预设的安全区域public、trusted、drop的作用和特点

public区域：为默认区域，只允许针对本机的 SSH 服务、dhcp、ping操作，其他都拒绝

trusted区域：对本机的任何访问都被允许

drop区域：访问本机的任何数据包都会被拒绝

09. SELinux是什么，对Linux系统有什么影响

SELinux机制：基于内核的安全增强机制，能够为Linux系统中的文档、进程等对象提供一套预设的保护规则

对Linux系统的影响：在强制保护的模式下，即便是root也不能违反其保护规则（除非更改或回避规则）

10. 实现ssh无密码远程管理

本机生成公钥(锁)与私钥(钥匙)进行验证

本机将公钥(锁)传递给对方

系统&服务管理进阶 ——

01. 配置httpd网站服务器时，快速添加新的虚拟主机

1）为每一个虚拟主机建立一份独立的配置文件，放到/etc/httpd/conf.d/目录下，配置文件名称以.conf结尾

2）配置 <VirtualHost IP地址:端口号> .. .. </VirtualHost> 区段标记，其中使用语句ServerName指定站点名称，使用DocumentRoot指定网页目录

3）然后重新启动httpd服务

02. NFS指的是什么

NFS，Network File System：文档资源存放在其他主机的目录上，网络文件系统

安装软件包为nfs-utils，配置文件为/etc/exports ，服务名为nfs-server

03. 简述从源代码编译安装软件的基本过程，其中每个环节的用途、命令工具

安装开发工具gcc与make

tar 解包：将下载的源码包解压释放

./configure 配置：建立安装清单（指定安装目录、需要的功能等）

make 编译：根据安装清单将源代码文件制作成二进制的可执行程序文件或相关模块

make install 安装：将可执行文件、相关模块、配置、文档等安装到系统中

04. RAID阵列指的是什么，RAID0、RAID1、RAID10、RAID5、RAID6各级别的特点对比

RAID0：条带模式，至少2块磁盘，通过并发读写提高效率

RAID1：镜像模式，至少2块磁盘，通过镜像备份提高磁盘设备的可靠性

RAID10：条件+镜像模式，相当于RAID1+RAID0，至少4块磁盘，读写效率及可靠性都更高

RAID5：高性价比模式，至少3块磁盘，其中1块磁盘容量用来存放恢复校验数据

RAID6：相当于扩展版的RAID5，至少4块磁盘，其中2块磁盘容量用来存放恢复校验数据

07. DNS服务器的作用

DNS服务器的作用：为客户机提供“域名-->IP地址”的信息查询服务

08. DHCP服务器作用

DHCP服务器：为客户机提供IP地址等参数

9. 构建时间服务器思路

1)安装软件包chrony

2)修改配置文件/etc/chrony.conf

3)重启时间服务chronyd

10. 自定义Yum仓库，如何生成仓库数据文件

createrepo

11. 如何更新自定义仓库

1. 仓库数据文件的更新

2. Yum仓库缓存的更新

]# createrepo --update /tools/other/ #更新仓库数据文件

]# yum makecache #更新缓存数据

]# yum repolist

12. 利用ftp服务构建网络yum思路

服务端关闭防火墙与SELinux

1.服务端安装vsftpd软件包

2.开启无需验证功能

3.服务端启动vsftpd服务，设置服务开机自启动

4.服务端在/var/ftp/创建挂载目录dvd

5.服务端将光驱设备或光盘镜像文件挂载到/var/ftp/dvd

客户端关闭防火墙与SELinux

1.客户端书写yum配置文件

vim /etc/yum.repos.d/centos.repo

[centos]

name=Linux7.8

baseurl=ftp://服务端IP地址/dvd

enabled=1

gpgcheck=0

13. 容器与传统的虚拟机对比有哪些优点

相比于传统的虚拟化技术，容器更加简洁高效

传统虚拟机需要给每个VM安装操作系统

容器使用的共享公共库和程序
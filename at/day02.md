- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [常用模块](#常用模块)
    - [yum\_repository模块](#yum_repository模块)
    - [yum模块](#yum模块)
    - [service模块](#service模块)
    - [lvg模块](#lvg模块)
    - [lvol模块](#lvol模块)
    - [filesystem模块](#filesystem模块)
    - [mount模块](#mount模块)
  - [Playbook剧本](#playbook剧本)
    - [YAML-语法](#yaml-语法)
    - [配置vim适应yaml语法](#配置vim适应yaml语法)
    - [测试联通性playbook](#测试联通性playbook)
    - [测试文件创建上传playbook](#测试文件创建上传playbook)
    - [基础playbook](#基础playbook)
    - [playbook中`|`的使用](#playbook中的使用)
    - [playbook中`>`的使用](#playbook中的使用-1)
    - [playbook用户密码](#playbook用户密码)
    - [parted模块](#parted模块)
- [快捷键](#快捷键)
  - [快速使用历史命令](#快速使用历史命令)
  - [vim命令模式单个词替换](#vim命令模式单个词替换)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)


# 学习目标

常用模块

Playbook

YAML语言

# 课堂笔记（命令）



# 课堂笔记（文本）

## 常用模块

### yum_repository模块

> 用于配置yum
>
> 常用选项：
>
> file： 指定文件名
> 其他选项，请与文件内容对照

```shell
[root@pubserver ansible]#ansible all -m yum_repository -a "file=myrepo name='APP' description='My APP' baseurl=ftp://192.168.88.240/dvd/AppStream gpgcheck=no enabled=yes" # 若需添加baseos，将内容改为baseos即可，但前提文件名与app创建时的文件名一致

[root@web1 ~]# cat /etc/yum.repos.d/myrepo.repo 
[APP]
async = 1
baseurl = ftp://192.168.88.240/dvd/AppStream
enabled = 1
gpgcheck = 0
name = My APP
```

### yum模块

> 用于rpm软件包管理，如安装、升级、卸载
>
> 常用选项：
>
> name：包名
> state：状态。present表示安装，如果已安装则忽略；latest表示安装或升级到最新版本；absent表示卸载。

```shell
[root@pubserver ansible]# ansible webservers -m yum -a "name=bind,bind-chroot state=present" # 装包，支持一个或多个;present扎装包可有可无
[root@pubserver ansible]# ansible webservers -m yum -a "name=bind state=absent" # 卸载
```

### service模块

> 用于控制服务。启动、关闭、重启、开机自启。
>
> 常用选项：
>
> name：控制的服务名
> state：started表示启动；stopped表示关闭；restarted表示重启
> enabled：yes表示设置开机自启；no表示设置开机不要自启。

```shell
[root@pubserver ansible]# ansible webservers -m yum -a "name=nginx state=latest" #安装nginx
[root@pubserver ansible]# ansible webservers -m service -a "name=nginx state=started enabled=yes" # 启动nginx并设置为开
```

### lvg模块

> 创建、删除卷组，修改卷组大小
>
> 常用选项：
>
> vg：定义卷组名。vg：volume group
> pvs：由哪些物理卷构成。pvs：physical volumes

```shell
# 在web1上安装lvm2
[root@pubserver ansible]# ansible web1 -m yum -a "name=lvm2"
# 在web1上添加两开硬盘20G，划分为gpt分区格式，划分两个分区sdb1 sdb2 一个5G 一个15G
[root@web1 ~]# lsblk # 查看磁盘添加情况
[root@web1 ~]#fdisk # 划分分区

[root@pubserver ansible]#ansible web1 -m lvg -a "vg=myvg pvs=/dev/vdb1" # 创卷卷组
[root@web1 ~]# vgs
  VG   #PV #LV #SN Attr   VSize  VFree 
  myvg   1   0   0 wz--n- <5.00g <5.00g
[root@pubserver ansible]# ansible web1 -m lvg -a "vg=myvg pvs=/dev/vdb1,/dev/vdb2" # 扩展卷组
[root@web1 ~]# vgs
  VG   #PV #LV #SN Attr   VSize  VFree 
```

### lvol模块

> 创建、删除逻辑卷，修改逻辑卷大小
>
> 常用选项：
>
> vg：指定在哪个卷组上创建逻辑卷
> lv：创建的逻辑卷名。lv：logical volume
> size：逻辑卷的大小，不写单位，以M为单位

```shell
[root@pubserver ansible]# ansible web1 -m lvol -a "vg=myvg lv=mylv size=2G" # 创建逻辑卷为2G
[root@web1 ~]# lvs
  LV   VG   Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  mylv myvg -wi-a----- 2.00g 
  
[root@pubserver ansible]# ansible web1 -m lvol -a "vg=myvg lv=mylv size=4G" # 扩容逻辑卷到4G；仅限于未格式化时扩容才有效
[root@web1 ~]# lvs
  LV   VG   Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  mylv myvg -wi-a----- 4.00g 
  
# 格式化后扩容；
[root@master ansible]# ansible salve1 -m filesystem -a "dev=/dev/my_vg/my_lv fstype=ext4 resizefs=yes"
```

[注]：xfs已经挂载不支持扩容

### filesystem模块

> 用于格式化，也就是创建文件系统
>
> 常用选项：
>
> fstype：指定文件系统类型
> dev：指定要格式化的设备，可以是分区，可以是逻辑卷

```shell
#  在web1上，把/dev/myvg/mylv格式化为xfs
[root@pubserver ansible]# ansible web1 -m filesystem -a "fstype=xfs dev=/dev/myvg/mylv"

# 在web1上查看格式化结果
[root@web1 ~]# blkid /dev/myvg/mylv
/dev/myvg/mylv: UUID="46c0af72-e517-4b15-9e53-ec72fbe1d96e" TYPE="xfs"
```

### mount模块

> 用于挂载文件系统
>
> 常用选项：
>
> path：挂载点。如果挂载点不存在，自动创建。
> src：待挂载的设备
> fstype：文件系统类型
> state：mounted，表示永久挂载

```shell
# 在web1上，把/dev/myvg/mylv永久挂载到/data
[root@pubserver ansible]# ansible web1 -m mount -a "path=/data src=/dev/myvg/mylv state=mounted fstype=xfs" 

# 在web1上查看
[root@web1 ~]# tail -1 /etc/fstab 
/dev/myvg/mylv /data xfs defaults 0 0
[root@web1 ~]# df -h /data/
Filesystem             Size  Used Avail Use% Mounted on
/dev/mapper/myvg-mylv  4.0G   61M  4.0G   2% /data

# 在web1上，卸载/dev/myvg/mylv
[root@pubserver ansible]# ansible web1 -m mount -a "path=/data state=absent"

# 在web1上，强制删除/dev/myvg/mylv 
[root@pubserver ansible]# ansible web1 -m lvol -a "lv=mylv state=absent vg=myvg force=yes"   # force是强制

# 在web1上，删除myvg卷组
[root@pubserver ansible]# ansible web1 -m lvg -a "vg=myvg state=absent"

# 删除顺序同手动删除一致，先卸载umount，再删除逻辑卷，再删除卷组
```

## Playbook剧本

> 常用于复杂任务的管理，以及管理经常要完成的任务
> playbook也是通过模块和它的参数，在特定主机上执行任务
> playbook是一个文件，该文件中需要通过yaml格式进行书写

### YAML-语法

> yaml文件的文件名，一般以yml或yaml作为扩展名
> 文件一般以`---`作为第一行，不是必须的，但是常用
> 键值对使用冒号:表示，冒号后面必须有空格。
> 数组使用-表示，-后面必须有空格。
> 相同的层级必须有相同的缩进。如果缩进不对，则有语法错误。每一级缩进，建议2个空格。
> 全文不能使用tab，必须使用空格。

### 配置vim适应yaml语法

```shell
[root@pubserver ansible]# vim ~/.vimrc
set ai        # 设置自动缩进
set ts=2      # 设置按tab键，缩进2个空格
set et        # 将tab转换成相应个数的空格
set cursorcolumn  # 查看是否对齐
# 不要复制注释
```

### 测试联通性playbook

```shell
# 编写用于测试连通性的playbook，相当于执行ansible all -m ping
[root@pubserver ansible]# vim test.yml
---
- name: test remote hosts  # name可以省略
  hosts: all
  tasks:
    - name: Test passed the ping model test
      ping:

[root@pubserver ansible]# ansible-playbook test.yml # 执行剧本
```

### 测试文件创建上传playbook

```shell
[root@pubserver ansible]# vim test.yml
---      
- name: create dir and copy file
  hosts: db1,web1
  tasks: 
    - name: create file
      file:
        path: /tmp/demo
        state: directory
        mode: '0755'
    - name: copy file
      copy:
        src: /etc/hosts
        dest: /tmp/demo/
[root@pubserver ansible]# ansible-playbook test.yml # 执行剧本     
[root@web1 ~]# ll /tmp/demo/
total 4
-rw-r--r-- 1 root root 277 Dec 19 15:24 hosts
```

### 基础playbook

```shell
# 在webservers主机创建用户bob、附加组为adm；创建文件1.txt写入内容hello world
[root@pubserver ansible]# vim two.yml
---      
- name: create user group webservers
  hosts: webservers
  tasks: 
    - name: create user group
      user:
        name: bob
        groups: adm
- name: create file and write content
  hosts: db1
  tasks: 
    - name: crate file
      copy:
        dest: /tmp/hi.txt
        content: "Hello World\n"
```

### playbook中`|`的使用

> `|`：保留文本原有格式

```shell
[root@pubserver ansible]# vim fl.yml
---      
- name: play1
  hosts: web1
  tasks: 
    - name: write content
      copy:
        dest: /tmp/2.txt
        content: |
        	hello world
        	ni hao shijie
[root@web1 ~]# cat /tmp/2.txt
hello world
ni hao shijie
```

### playbook中`>`的使用

> `>`：不换行

```shell
[root@pubserver ansible]# vim fl.yml
---      
- name: play1
  hosts: web1
  tasks: 
    - name: write content
      copy:
        dest: /tmp/3.txt
        content: >
        	hello world
        	ni hao shijie
[root@web1 ~]# cat /tmp/2.txt
hello world ni hao shijie
```

### playbook用户密码

```shell
[root@pubserver ansible]# vim user_john.yml
---    
- name: create user uid group password
  hosts: webservers
  tasks:
   - name: user uid group password
     user:
      name: john
      uid: 1040
      group: daemon
      password: "{{'123'|password_hash('sha512')}}"
```

### parted模块

> 用于硬盘分区管理
>
> 常用选项：
>
> device：待分区的设备
> number：分区编号
> state：present表示创建，absent表示删除
> part_start：分区的起始位置，不写表示从开头
> part_end：表示分区的结束位置，不写表示到结尾

```shell
# 创建一个1gib分区
# 创建一个5gib分区
# 创建卷组
# 创建逻辑卷
# 格式化
# 挂载
[root@pubserver ansible]# vim disk.yml
---      
- name: disk manage
  hosts: web1
  tasks: 
    - name: create a partition
      parted:
        device: /dev/vdc
        number: 1
        state: present
        part_end: 1GiB
    - name: add a new partition
      parted:
        device: /dev/vdc
        number: 2
        state: present
        part_start: 1GiB
        part_end: 6GiB
    - name: create vg
      lvg:
        vg: my_vg
        pvs: /dev/vdc1,/dev/vdc2
    - name: create lv
      lvol:
        vg: my_vg
        lv: my_lv
        size: 1G
    - name: mkfs my_lv
      filesystem:
        dev: /dev/my_vg/my_lv
        fstype: ext4
    - name: mount data
      mount:
        path: /data
        src: /dev/my_vg/my_lv
        fstype: ext4
        state: mounted
        
[root@web1 ~]# df -Th /data
Filesystem              Type  Size  Used Avail Use% Mounted on
/dev/mapper/my_vg-my_lv ext4  976M  2.6M  907M   1% /data              	
```





# 快捷键

## 快速使用历史命令

```shell
[root@pubserver ansible]#  # 按住ctrl+r
(reverse-i-search)`':   # 搜索命令关键词，找到后按esc保留命令
```

## vim命令模式单个词替换

> 先按R 再按需要替换的内容，仅仅是单个内容

# 问题





# 补充



# 今日总结



# 昨日复习

配置ansible管理环境

1. 配置网络参数
2. 配置yum
3. 安装ansible
4. 配置名称解析 /etc/hosts
5. 配置ansible到管理主机的ssh免密
6. 创建工作目录
7. 进入工作目录，创建相关的配置文件 ansible.cfg inventory
8. 两种管理方法，adhoc、playbook
9. ping、command、shell、script、file、copy、fetch、lineinfile、replace、user、group
10. 命令模式、声明模式
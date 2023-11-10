- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
  - [物理卷（PV）](#物理卷pv)
  - [卷组（VG）](#卷组vg)
  - [逻辑卷（LV）](#逻辑卷lv)
  - [刷新逻辑卷](#刷新逻辑卷)
    - [xfs](#xfs)
    - [ext4](#ext4)
  - [pstree](#pstree)
  - [pgrep](#pgrep)
  - [ps](#ps)
  - [top](#top)
  - [sleep](#sleep)
  - [sudo提权](#sudo提权)
- [课堂笔记（文本）](#课堂笔记文本)
  - [什么是逻辑卷](#什么是逻辑卷)
  - [物理卷创建](#物理卷创建)
  - [卷组创建](#卷组创建)
  - [建立逻辑卷](#建立逻辑卷)
  - [使用逻辑卷](#使用逻辑卷)
  - [逻辑卷扩展](#逻辑卷扩展)
    - [卷组有足够空间](#卷组有足够空间)
    - [卷组没有足够空间](#卷组没有足够空间)
  - [控制进程](#控制进程)
  - [杀死进程](#杀死进程)
- [快捷键](#快捷键)
- [问题](#问题)
  - [卷组添加时问题](#卷组添加时问题)
  - [卷组默认单位问题](#卷组默认单位问题)
  - [逻辑卷缺点](#逻辑卷缺点)
- [总结](#总结)
- [补充](#补充)
  - [案例1.杀死属于指定用户的所有进程](#案例1杀死属于指定用户的所有进程)

# 学习目标

逻辑卷管理

VDO

RAID磁盘阵列

进程管理

用户提权

# 课堂笔记（命令）

## 物理卷（PV）

1. 查看系统物理卷

   ```linux
   ]#pvs
     PV         VG       Fmt  Attr PSize   PFree
     /dev/vda2  rl       lvm2 a--  <19.00g    0 
   ```

2. 创建物理卷（可忽略，直接创建卷组）

   格式：prcreate /dev/磁盘分区

   ```linux
   ]#pvcreat /dev/vdb[1-2]
   ```

3. 删除物理卷

   格式：pvremove /dev/磁盘

   ```linux
   ]#pvremove /dev/vdb{1,2,3,5,6}
   ```

   

## 卷组（VG）

1. 查看系统存在卷组

   ```linux
   ]#vgs
     VG       #PV #LV #SN Attr   VSize   VFree 
     rl         1   2   0 wz--n- <19.00g     0 
   ```

2. 创建卷组

   格式：vgcreate  卷组名字 /dev/磁盘分区

   ```linux
   ]#vgcreat systemvg /dev/vdb[1-2]
   ```

3. 扩大卷组大小

   格式：vgextend 卷组名  /dev/磁盘分区

   需求：逻辑卷需要扩展，但卷组空间不满足逻辑卷

   ```linux
   ]#vgextend systemvg /dev/vdb{3,5,6}
   ```

4. 修改卷组默认单位

   格式：vgchange -s 需设定的单位 卷组名

   ```linux
   ]#vgchange -s 1M systemvg  # 卷组已存在
   ]#vgdisplay systemvg # 查询当前卷组信息
   ```

   代码解释：将systemvg卷组大小更改为1;系统默认单位4M

   

   创建卷组时指定PE大小

   ```linux
   ]#vgcreate -s 1M systemvg /dev/vdb[1-2]
   ```

5. 删除卷组

   格式：vgremove 卷组名

   ```lnux
]#vgremove systemvg
    Volume group "systemvg" successfully removed
   ```

## 逻辑卷（LV）

1. 查询当前戏台存在的逻辑卷

   ```lniux
   ]#lvs
   ```

2. 创建逻辑卷

   格式：lvcreate -L 大小 -n 名字 卷组名

   ```linux
   ]#lvcreate -L 16G -n vo systemvg
   Logical volume "vo" created.
   ```

   创建时指定PE大小

   ```linux
   ]#lvcreate -l 98 -n lbvase systemvg
   ```

3. 逻辑卷名更改

   格式：lvrename 逻辑卷名 重命名后

   ```linux
   ]#lvrename vo vo1
   ```

4. 删除逻辑卷

   前提：被删除的逻辑卷已提前卸载（umout）

   ```linux
   ]#lvremove /dev/systemvg/vo
   Do you really want to remove active logical volume systemvg/vo? [y/n]: y
     Logical volume "vo" successfully removed
   ]#vim /etc/fstab  # 删除开机自动挂载
   ```


## 刷新逻辑卷

前提：逻辑卷扩展后的操作

### xfs

格式：xfs_grows  卷组所在地

```lnux
]#xfs_grow /dev/systemvg/vo
```

### ext4

格式：resize2fs 卷组所在地

```linux
]#resize2fs /dev/systemvg/vo
```

## pstree

[-p]：列出对应进程的pid编号

```linux
]# pstree # 显示正在运行的所有进程
]# pstree -p
systemd─┬─ModemManager───2*[{ModemManager}]
        ├─NetworkManager───2*[{NetworkManager}]
        ├─accounts-daemon───2*[{accounts-daemon}]
        ├─atd
]# pstree -p yyh  # 查询用户yyh运行的进程及pid 
bash(10027)───less(10309)
```

[-a]：显示完整的命令行

```linux
]# pstree -a 
  │       └─2*[{xdg-permission-}]
  ├─systemd-journal
  ├─systemd-logind
  ├─systemd-machine
  ├─systemd-udevd
  ├─tuned -Es /usr/sbin/tuned -l -P
  │   └─4*[{tuned}]
  ├─udisksd
  │   └─4*[{udisksd}]
  ├─upowerd
]# pastree -a yyh # 显示用户正在运行的进程完整命令
bash
  └─less /etc/passwd
```

## pgrep

作用：查询正在运行进程

格式：pgrep [选项] 查询条件

[-l]：输出进程名

```linux
]#pgrep -l sys  
1 systemd
721 systemd-journal
762 systemd-udevd
888 systemd-machine
994 systemd-logind
1048 rsyslogd
2080 systemd
```

[-u]：检索指定用户的进程

```linux
]# pgrep -u yyh # 类似命令pstree -p yyh
10027
```

[-x]：精确匹配完整的进程名

```linux
]# pgrep  -x   crond   #精确匹配完整的进程名
1057
]#pgrep -lx crond
1057 crond
```

代码解释：所有选项可以配和使用

## ps

格式：ps [选项]

常用命令选项：

aux：显示当前终端所有进程（a）、当前用户在所有终端下的进程（x）、以用户格式输出（u）

```linux
]# ps aux # 显示当前所有进程

]# ps aux | wc -l #
```

-elf：显示系统内所有进程（-e）、以长格式输出（-l）信息、包括最完整的进程信息（-f）

```linux
]# ps -elf # 列出所有进程加父进程
```

## top

作用：动态查看进程排名

[-d]：刷新秒

```linux
top -d 1
```

进入top后按P：按照CPU排序

进入top后按M：按照内存排序

按住q：退出

## sleep

``` sleep
]#sleep 2000  #命令行挂起2000s，ctrl+c退出
```

## sudo提权

作用：让普通用户具备root用户身份去执行某些操作

为用户增加权限，须在/etc/sudoers里面编写（不推荐vim /etc/sudoers）

例：创建bob用户，添加cat、mkdir权

```linux
visudo # 此方法可以检查语法错误（推荐）
bob        ALL=（root）       /usr/bin/cat,/usr/bin/mkdir 
普通用户 所有的主机=（变成的身份） 可以执行的命令
useradd bob # 创建用户bob
echo 123  | passwd --stdin bob
su - bob
[bob@nb ~]sudo -l # 查看此用户提权的命令
[sudo] bob 的密码：   #输入bob用户的密码
....
用户 bob 可以在 test 上运行以下命令：
    (root) /usr/bin/cat, /usr/bin/mkdir
[bob@nb ~]$ cat    /etc/gshadow
cat: /etc/gshadow: 权限不够
[bob@nb ~]$ sudo   cat   /etc/gshadow
[bob@nb ~]$ exit
```

例：取消提权密码验证（根据上方案例）

```                                            
visudo # 编写取消提权密码验证
bob    ALL=（root）  NOPASSWD:/usr/bin/cat,/usr/bin/mkdir 
su - bob
sudo cat /etc/shadow  
```

[注]：用户前加上%则是给组提权，无则是对用户

# 课堂笔记（文本）

## 什么是逻辑卷

作用：整合分散空间、空间支持扩大

逻辑卷制作过程：将众多的物理卷（PV）组建成卷组（VG），再从卷组中划分出逻辑卷（LV）

物理卷：磁盘分区后未进行格式化的磁盘分区（vdb1等）

卷组：多个磁盘分区（物理卷）的组成

## 物理卷创建

```linux
]#pvcreate /dev/vdb[1-2]
  Physical volume "/dev/vdb1" successfully created. 物理卷成功 
  Physical volume "/dev/vdb2" successfully created.
```

## 卷组创建

作用：将物理分区合成卷组

[注]：详细操作，需先创建物理卷，再创建卷组

详解： 使用vgcreat可以自动创建卷组，不用手动再次创建

```linux
]#vgcreat system /dev/vdb[1-2]
  Physical volume "/dev/vdb1" successfully created. 物理卷成功 
  Physical volume "/dev/vdb2" successfully created.
  Volume group "systemvg" successfully created 卷组创建成功
```

## 建立逻辑卷

格式：lvcreat -L 大小G  -n  逻辑卷名字 卷组名

1. 不使用PE

   ```linux
   ]#lvcreat -L 16G -n vo systemvg
    Wiping xfs signature on /dev/systemvg/vo.
     Logical volume "vo" created.
   ```

2. 使用PE

   ```linux
   lvcreate -l 98 -n vo1 systemvg
   ```

   代码解释：创建由98个PE组成的逻辑卷                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        

## 使用逻辑卷

格式化文件系统-挂载逻辑卷（开机自动挂载）

```linux
]# ls   /dev/systemvg/vo 
/dev/systemvg/vo
]# ls -l    /dev/systemvg/vo
lrwxrwxrwx. 1 root root 7 11月 10 11:07 /dev/systemvg/vo -> ../dm-2
]# mkfs.xfs    /dev/systemvg/vo    #格式化xfs文件系统
]# blkid   /dev/systemvg/vo   #查看文件系统类型
]# vim  /etc/fstab        
/dev/systemvg/vo   /mylv    xfs   defaults  0   0
]# mkdir    /mylv
]# mount   -a       #检测fstab文件内容书写是否正确
]# df    -h    /mylv    #查看查看正在挂载使用的设备
```

代码解释：进行blkid查看文件系统时，如果需要查看vo需要手动敲出systemvg，一般逻辑卷存放在/dev/mapper/xx-逻辑卷名

## 逻辑卷扩展

### 卷组有足够空间

```linux
]# df   -h   |   grep   vo # 查询
]# vgs
]# lvextend    -L   18G    /dev/systemvg/vo
]# vgs
]# lvs
]# xfs_growfs  /dev/systemvg/vo
]# df   -h   |   grep   vo
]# lvs
```

代码解释：扩展逻辑卷的大小有两种

1. +5G在原有的基础上增加5G
2. 15G硬盘扩展到15G（系统本有10G，需要扩展5G）

一般使用第2种

### 卷组没有足够空间

```linux
]#vgextend systemvg /dev/vdb{3,5,6}  # 4为扩展分区
  Physical volume "/dev/vdb3" successfully created.
  Physical volume "/dev/vdb5" successfully created.
  Physical volume "/dev/vdb6" successfully created.
  Volume group "systemvg" successfully extended
  
]#lvextend -L 25G /dev/systemvg/vo
  Size of logical volume systemvg/vo changed from 18.00 GiB (4608 extents) to 25.00 GiB (6400 extents).
  Logical volume systemvg/vo successfully resized.
  
]#xfs_growfs /dev/systemvg/vo  # 刷新
]#df -h | grep vo
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/systemvg-vo   25G  211M   25G    1% /mylv

]#lvs
  vo   systemvg -wi-ao----  25.00g   
```

代码解释：卷组空间不足，需要从物理卷中新增到卷组中，操作步骤

1. 新增物理卷存储到卷组中
2. 再使用逻辑卷增加所要添加的空间大小
3. 刷新逻辑卷
4. 查看是否成功

## 控制进程

作用：进程前后台的调度

•Ctrl + z 组合键：挂起当前进程（暂停并转入后台）

```linux
]# sleep 2000 # 挂起2000s
^Z                 #按Ctrl+z  暂停放入后台
```

•jobs 命令：查看后台任务列表

```linux
]#jobs
[1]+  已停止               sleep 2000
```

•&符号：正在运行的状态放入后台

```linux
]#sleep 2000 &  # 将命令行挂起2000s放入后台运行
```

•fg 命令：将后台任务恢复到前台运行

```linux
]#fg 1     #让后台编号为1 的进程恢复到前台
sleep 2000
^C               #按Ctrl+c   结束
```

•bg 命令：激活后台被挂起的任务在后台运行

```linux
]# bg 1     #让后台编号为1 的进程继续运行
[1]+ sleep 2000 &
```

## 杀死进程

ctrl+c组合键，中断当前命令程序

kill [-9] pid、kill [-9] %后台任务编号

```linux
]# sleep 8000 &
[1] 11874
]# kill -9 %1
]# jobs
[1]+  已杀死               sleep 8000
```

killall [-9] 进程名

```linux
]# killall -9 sleep
[1]-  已杀死               sleep 8000
[2]+  已杀死               sleep 8000
```



# 快捷键



# 问题

## 卷组添加时问题

进行卷组添加时，只能添加主分区与逻辑分区，扩展分区不能添加

## 卷组默认单位问题

默认单位PE，每个PE为4M

## 逻辑卷缺点

1. 卷组只能集结一台机器的空间

2. 物理卷或卷组任何一个地方出错，整个硬盘崩溃

3. 读写速度不太优秀

# 总结



# 补充

## 案例1.杀死属于指定用户的所有进程

pkill -9 -u 用户名

例：杀死用户yyh的所有进程

```linux
]#useradd yyh
]#su - yyh
]$sleep 1000 &
]$sleep 1000 &
]$sleep 1000 &
]$vim /1.txt &
]$jobs
[1]   运行中               sleep 1000 &
[2]   运行中               sleep 1000 &
[3]-  运行中               sleep 1000 &
[4]+  已停止               vim /1.txt
# 切换到root
]#pgrep -up yyh # 使用进程树展示
bash(38697)─┬─sleep(38731)
            ├─sleep(38732)
            ├─sleep(38733)
            └─vim(38734)
]#pkill -9 -u yyh # 强制杀死属于用户yyh的所有进程
]$ 已杀死
```

案例总结：杀死所有进程后，可以强制踢出用户


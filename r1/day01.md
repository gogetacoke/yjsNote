- [本周复习内容](#本周复习内容)
  - [Liunx系统简介](#liunx系统简介)
  - [云计算介绍](#云计算介绍)
  - [服务器](#服务器)
  - [Tcp/IP协议及配置](#tcpip协议及配置)
  - [命令行基础](#命令行基础)
  - [目录及文件管理](#目录及文件管理)
  - [文本内容操作](#文本内容操作)
  - [归档及压缩](#归档及压缩)
  - [重定向与管道操作](#重定向与管道操作)
  - [find精确查找](#find精确查找)
  - [vim高级使用](#vim高级使用)
- [复习问题总结](#复习问题总结)
- [预习下周内容](#预习下周内容)

# 本周复习内容

> 以下命令格式：命令 选项  参数（举例）

## Liunx系统简介

1. 什么是Liunx？

   Liunx是一种操作系统

   + 什么是操作系统？

     操作系统是一堆软件的集合，让计算机硬件正常的个工作

     软件：微信、qq等

     硬件：CPU、磁盘、内存、显卡

2. Unix/Liunx的发展史

   Unlx诞生：1970-1-1（规定所有linux系统时间都是从1970-1-1开始至今）

   创始人：Ken Thompson、Dennis Ritchie

   LInux诞生：1991

   创始人：Linus Torwalds

   1991-10：发布第一个公开内核版本

   内核作用：管理CPU/内存、驱动基本硬件、文件系统

3. Linux版本及应用

   + Red Hat Enterprise Linux 

     红帽LInux企业版（RHEL）

     Centos社区版（开始不维护开源）

     ​	Rock Linux 免费开源企业级操作系统

   + Rocky Linux 

   + Suse Liunx Enterprise

   + Debian Linux

   + Ubuntu Linux

   + ........

   基于Linux的企业服务器

   嵌入式系统

   高性能大型运算

   ........

## 云计算介绍

一堆计算机同时对外提供服务

## 服务器

含义：能够为其他计算机提供服务的更高级的电脑

分类：机架式、塔式、柜机式、刀片式

尺寸：1u=1.75英寸=44.45毫米=4.445厘米（高）

典型服务器模式：C/S架构

	- 由服务器提供资源或某种功能
	- 客户机使用资源或功能

## Tcp/IP协议及配置

简介：TCP/IP是最广泛的通信协议集合

​	包括大量的internet应用中的标准协议

​	支持网络架构、跨操作系统平台的通信

主机与主机之间通信的三要素

+ IP地址（IP address）

  作用：用来标识一个网络节点的网络**地址**

  IP地址的组成：

  ​	一共32个二进制数

  ​	1001100.01010101.111000.10101010

  ​	表示为4个十进制数，以点隔开

  ​	192.168.1.1

  ​	二进制的11111111 = 十进制的255

  IP地址的分类：

  ​	一般计算机网络

  ​		A类：1～127  网+主+主+主

  ​		B类：128～191 网+网+主+主

  ​		C类：192～223 网+网+网+主

  ​	组播及科研专用

  ​		D类：224～239 组播

  ​		E类：240～254 科研

  网络位与主机位：

  ​	网络位：类似身份证号的前6位，用于表示区域

  ​	主机位：用来表示在区域中编号

  ​	例如：192.168.1.1=来自192.168.1区域，在区域中编号为1的主机	

+ 子网掩码（subnet mask）

  为计算机标识IP地址的网络位与主机位，利用二进制的1表示网络位，0表示主机位

+ IP路由（IP router）

## 命令行基础

1. pwd 显示当前所在位置

2. cd 目录切换

3. ls 显示当前目录内容

   ls -l /etc/passwd 长格式显示

4. cat 查看文本文件内容，适合查看内容较少文件

   cat -n /etc/passwd 显示行号

5. less 功能同上，适合查看内容较多文件，按上下键进行滚动，按q键进行退出

6. hostname 查看主机名词

   临时修改主机名：hostname hh.xx.gg

7. lscpu 查看CPU信息

8. cat /proc/meminfo 查看内存信息

   free 同上，但内容易懂

9. ifconfig 查看ip地址

   临时设置ip地址：ifconfig eth0 192.168.4.1 （eth0为网卡名）

10. mkdir /opt/hh 创建单层目录

11. touch /opt/123.txt 创建文本文件

12. head -2 /etc/passwd 查看部分内容的前两行

13. tail -2 /etc/passwd 查看部分内容的后两行

14. grep root /etc/passwd 过滤passwd文件中包含root的行并输出

15. vim 文本编辑器

    三个模式：命令模式、插入模式、末行模式

    vim当文件不存在，会自动创建此文件

    vim不能创建目录

    打开文本文件后，按i进入插入模式、输入:进入末行模式

16. reboot 重启系统

17. poweroff 关闭系统

18. which hostname 查询命令对应的程序

19. 命令 --help 查询当前命令的帮助信息

20. mount 挂载，让目录成为设备的访问点

    mount /devcdrom /dvd

21. umount 挂载

    umount /dvd

## 目录及文件管理

1.  .当前目录

2.  ..父目录

3. ～表示用户的家目录

4. ls -h提供易读的单位（K、M等）

   ls -lh /etc

   ls -d 显示目录本身的属性，而不是内容

   ls -ld /etc/passwd

   ls -A 显示目录所内所有隐藏的内容

   ls -R 递归显示目录内容

5. *匹配多个任意字符，针对不确定的文档名称，以特殊字符表示

   ls /root/a*   列出root目录下所有a开头的文件

   ls /etc/*tab 列出etc目录下以tab结尾的文件

6. ？单个字符，必须要有一个

   ls /etc/?tab 例如etc下有atab、ctab文件，将会被匹配出来 

   ls /etc/tty? 同上

7. [a-z]多个字符中的一个，若无则忽略，数字最小为0最大为9

   ls /dev/tty[1-9] 列出dev目录下tty1-tty9

8. {a，min，xy}多组不同的字符串，全匹配

   ls /etc/{rw,fs}tab 列出etc目录下rwtab、fstab

9. alias 别名，临时别名

   alias hn='hostname'

   alias myls =‘ls -l’ 

10. unalias 删除别名

    unalias hn

11. mkdir-p 连同父目录一并创建

    mkdir -p /work/apps 在根目录下创建wokr目录且目录内创建app文件夹

12. rm 删除文件或目录

    [-r] 递归删除

    [-f] 强制删除

    rm -rf /work/apps/1.txt 删除/work/apps目录下1.txt文件

    rm -rf /work/apps 删除apps目录

    rm -rf /work/* 删除/work目录下所有内容

13. mv 移动，源数据会消失

    mv /work/apps/1.txt /opt 将1.txt文件移动到opt目录下

14. cp 复制，源数据不会消失

    [-r] 复制**目录**时必须选择此选项

    cp /etc/passwd /opt 将etc目录下passwd内容复制到opt目录下

    cp -r  /etc/cni /opt 将etc下cni目录复制到opt目录下

    \cp -r /etc/cni /opt 强制覆盖

    cp /etc/passwd /opt/1.txt 将passwd文件复制到opt目录下并改名为1.txt

    cp /etc/passwd /work/apps/1.txt /opt  复制可以支持两个以上的参数，永远把最后一个参数作为目标，其他的所有的参数都作为源数据

## 文本内容操作

grep 高级使用

-v 取反匹配

grep -v root /etc/passwd 输出passwd中不包含root的文本

-i 忽略大小写

grep -i ROOT /etc/passwd 输出passwd中包含root的文本忽略大小写

^ 以字符串开头

grep -i ^r /etc/passwd 输出passwd中r开头的文本且忽略大小写的R

$ 以字符串结尾

grep o$ /etc/passwd 输出passwd中o结尾的文本文件

^$ 表示空行

grep -v ^$ /etc/passwd 输出passwd中不为空行的所有文本

## 归档及压缩

> 制作tar包格式：tar 选项 /路径/压缩包名字 /源数据

-c：动作为创建

-f：指定压缩包的包名字（必须在所有选项最后）

以下是调用的格式工具（从快-慢）

-z：.gz

-j：.bz2

-J：.xz

将passwd文件和home目录打包到opt目录下使用.gz打包工具

tar -zcf /opt/testpackg.tar.gz /etc/passwd /home

> 释放tar包格式：tar 选项 /路径/压缩包名字 选项 /释放的位置

-x：释放归档

-f：制定释放压缩包名字，必须放到选项最后

-C：指定释放的路径

将opt下的testpackage包释放到mnt目录下

tar -xf /opt/testpackge.tar.gz -C /mnt

**`以上打包会将同级目录同时打包到压缩包里面`**

> tar高级打包格式：tar 选项 /路径/压缩包名字 选项 /路径/空格 源数据

将etc目录下passwd、shells、hosts、fstab同时打包到opt下不打包路径

tar -zcf /opt/test2.tar.gz -C /etc passwd shells hosts fstab

`查询压缩包内容`

tar -tf /opt/test2.tar.gz

## 重定向与管道操作

> 重定向：将前面命令的输出内容，作为内容，写入到后面的文件

`>`：覆盖重定向

将passwd文件前五行内容写入到2.txt文件中（2.txt存在即覆盖原先里面有的内容，不存在则新建该文件并写入内容）

head -5 /etc/passwd > /opt/2.txt

`>>`：追加重定向

将passwd文件的前两行内容追加到2.txt文件中

head -2 /etc/passwd >> /opt/2.txt

> echo命令：输出echo命令后的内容

控制台输出123

echo 123

echo 配合重定向使用

echo 123 > /opt/3.txt

echo 123 >> /opt/3.txt

**echo高级使用**

清空3.txt文件中的所有内容

`>`  /opt/3.txt

> `|`：管道操作，就爱嗯命令的输出，传递给后面的命令，作为后面命令的参数

查询passwd文件的前四行，只展示最后一行

head -4 /etc/passwd `|` tail -1

展示passwd 文件第七行到第十五行

head -15 /etc/passwd `|` tail -9

将passwd中第7到15行内容和行号写入3.txt文件

cat -n /et/passwd  `|` head -15 `|` tail -9  > /opt/3.txt

**grep高级使用**

显示login.defs中有效信息(去除#和空行为有效信息)

grep -v ^# /etc/login.defs `|` grep -v ^$

将以上有效信息写入opt下4.txt

grep -v ^# /etc/login.defs `|` grep -v ^$ > /opt/4.txt

**过滤命令输出**

展示ifconfig中所有inet内容

ifconfig `|` grep inet

## find精确查找

> find命令格式：find 目录  条件

**-type** 类型（f文本文件、d目录、l快捷方式）

查询etc下所有的文本文件

find /etc -type f

**-name** ‘文档名词’（iname忽略大小写）

查询etc下所有a开头的文件

find /etc -name 'a*' -type f

查找etc下隐藏数据（小数点开头文件为隐藏文件）

find /etc -name ‘.*’

计算etc目录下有多少个以.conf的文件

find -name ‘*.conf’ -type f  `|` wc -l

**多条件使用** 

-o 满足一个条件即可

展示etc下以.conf结尾的文件或目录

find /etc -name ‘*.conf’ -o -type l

**-size** +（大于）或-（小雨）文件大小（k、M、G）

找出boot目录下文件大于300k的文件

find /boot -size +300k

找出boot目录文件大于10M小于50M

find /boot -size +10M -50M

**-user**用户名（按照数据的使用者）

创建一个用户

useradd test1

在根目录下查询用户test1的所有数据

find / -user test1

**-mtime** 修改时间（所有的时间都是过去时间）

+90：90天之前的数据

-90：90天之内的数据

查询 /etc 三个月前被修改的数据

find /etc -mtime +90

> find高级使用
>
> 命令格式：find `[范围][条件]` -exec 处理命令 {} \;
>
> -exec 额外操作的开始
>
> {} 永远表示前面find查找的结果
>
> \;额外操作的结束

查找boot目录下大于10M的文本文件复制到opt目录下

find /boot -size +10M -type f -exec cp {} /opt \;

## vim高级使用

**vim末行模式**

1. 读取文件内容

   :r /opt/1.txt

2. 字符串替换

   替换1-10行所有的root为okk

   :1,10s/root/okk/g

   替换文件内所有的root为okk

   :%s/root/okk/g
   
3.  开关参数的控制

   :set nu 或 set nonu 显示/不显示行号

   :set ai 或 set no ai 启用/不启用自动缩进（光标自动与上一行对齐）

   以上功能是临时的，永久设置如下：

   vim /root/.vimrc

**vim命令模式**

1. 多文件同时操作

   vimdiff /opt/1.txt /opt/2.txt

   按下ctrl+w然后左右键移动光标

   保存所有文件:wqa

2. 光标跳转

   1G或两个gg跳转文件的行首

   G跳转到文件的末尾行

   6gg：光标跳转至第6行

3. 复制

   yy复制光标出的一行

   12yy复制光标在内向下的12行

4. 粘贴

   p：粘贴到光标之后

   P：粘贴到光标之后

5. 删除

   x或delete键删除光标处的单个字符

   dd：删除光标出的第一行

   5dd：删除光标在内的向下五行

6. 撤销

   u：撤销最近的一次操作

   U：撤销当前行的所有修改

   ctrl+r：取消前一次撤销操作

7. 保存

   ZZ：保存并退出

   wq：保存并退出

8. 查找

   /word:向当前光标后查找字符串‘word’，按n/N调至前/后一个结果
   
   
   
   

# 复习问题总结

1. Liunx系统的根目录，/dev目录的作用是什么

   /：Linux文件系统的起点，linux所有的文件都存放在其中

   /dev：存储系统设备文件

2. 硬盘的命名规则

   + IDE硬盘为hda
   + SCSI或STAT硬盘为sd（第一块硬盘sda）
   + nvme硬盘为nvme0n1
   + KVM虚拟机硬盘为vda
   
3. 目录为蓝色、文件为黑色、可执行的程序为绿色

4. 命令行执行依赖于解释器（系统默认解释器：/bin/bash）

5. ctrl+c结束正在运行的命令

6. esc+.粘贴上一个命令的参数

7. ctrl+ l清楚整个屏幕

8. ctrl+w 往回删除一个单词，以空格为界定

9. 去除以#开头的注释行和空行为有效信息

10. 使用useradd创建一个普用户

11. /proc中存放内存或不占硬盘的数据

12. 简述绝对路径、相对路径

    相对路径：以/开始的完整路径

    绝对路径：以当前工作目录为参照的路径

13. linux命令行常用的通配符有哪些，各自的作用是什么

    *：任意多个字符

    ?：单个字符

    [a-z]：多个字符或连续范围中的一个，若无则忽略

    {a，mai，b}：多足不同的字符串，全匹配

14. 重定向与管道的区别

    重定向：将前面命令的输出，写入到后面的文本文件中，能过连接命令与文件

    管道：将前面输出的结果作为后面命令的参数，能够连接命令与命令

# 预习下周内容


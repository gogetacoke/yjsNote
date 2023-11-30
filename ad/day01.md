-- [学习目标](#学习目标)
- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
- [快捷键](#快捷键)
- [问题](#问题)
- [总结](#总结)


# 学习目标

Linux系统简介

安装Linux系统

Linux基本操作

# 课堂笔记（命令）

pwd

> 查看当前所在位置

cd

> 进入

cd ..

> 进入上一层

cd ../..

> 进入上两层

ls

> 显示当前目录下内容

ls /root

> 查看指定目录内容

lsblk

> 查看磁盘信息

lscpu

> 查看cpu信息

cat

> 查看文本内容详情

cat /proc/meminfo

> 查看内存信息

cat /etc/redhat-release

> 查看当前系统版本

less

> 查看文本内容详情，可以使用上下键进行滚动
>
> 按q键进行退出

hostname

> 查看主机名

hostname xxx

> 临时更改主机名

hostnamectl set-hostname xxx

> 永久修改主机名

ping

> 测试网络情况

ifconfig

> 查看ip信息

ifconfig 网卡 192.168.0.1

> 给指定网卡设置临时IP

mkdir

> 创建文件夹

touch

> 创建文件

head -x /x

> 查看指定文本第几行前的所有内容

tail -x /x/x

> 查看制定文本后几行的所有内容

grep xx /x/x

> 根据xx过滤文本中含有xx的内容并显示

vim

> 文本编辑器
>
> 三个编辑模式：命令模式、插入模式、末行模式
>
> wq保存并退出
>
> q！强制不保存并退出
>
> w保存不退出

reboot

> 重启

poweroff

> 关机

# 课堂笔记（文本）

1. UNIX诞生时间

   1970-1-1

2. Lnux常用的系列

      Red Hat Enterprise Linux 6/7/8/9

      Rocky Linux 8/9

      Suse Linux Enterprise 12

      Debian Linux 7.8

      Ubuntu Linux 22.04

3. IP地址的作用

   标识一个网络节点的网络地址

4. 子网掩码的作用

   区分IP地址的网络位与主机位

5. 硬盘的命名规则是什么

   1）IDE硬盘为hda

   2）SCSI或SATA硬盘为sda

   3）nvme硬盘为nvme0n1

   4）KVM虚拟机硬盘为vda

# 快捷键

ctrl + l 清除终端

# 问题



# 总结

1. Linux目录中与Linux目录结构中"/"与"/dev"主要作用？

   / 表示根目录

   /dev 存储设备相关数据

2. Linux中查看文本文件内容命令是？

   cat

   less

3. Linux中查看目录内容的命令是？

   ls

4. Linux中查看主机名的命令是？

   hostname

5.  Linux中查看IP地址的命令是？

   ifconfig

6. Linux中切换到/dev目录的命令是？

   cd /dev

7. Linux中显示当前位置的命令是？

   pwd

8.  Linux中查看CPU信息命令是？

   lscpu

9.  Linux中查看内存信息命令是？

   free

   cat /proc/meminfo

10. Linux中查看/boot目录内容如何操作？

    ls /boot

11.  Linux中查看/etc/passwd文件前两行如何操作？

    head -2 /etc/passwd

12.  Linux中查看/etc/passwd中包含root的行，如何操作？

    grep root /etc/passwd








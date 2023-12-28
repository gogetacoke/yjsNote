- [今日计划](#今日计划)
- [本周复习内容](#本周复习内容)
  - [RPM软件包简介](#rpm软件包简介)
  - [RPM包一般安装的位置](#rpm包一般安装的位置)
  - [查询软件信息](#查询软件信息)
  - [软件包的依赖关系](#软件包的依赖关系)
  - [YUM仓库构建](#yum仓库构建)
  - [获取帮助命令](#获取帮助命令)
  - [历史命令](#历史命令)
  - [系统时间](#系统时间)
  - [统计磁盘大小](#统计磁盘大小)
  - [制作链接文件](#制作链接文件)
  - [zip归档工具](#zip归档工具)
  - [用户配置文件](#用户配置文件)
  - [用户的创建](#用户的创建)
  - [usermod命令](#usermod命令)
  - [设置用户密码](#设置用户密码)
  - [用户初始配置文件](#用户初始配置文件)
  - [删除用户](#删除用户)
  - [组账户管理](#组账户管理)
  - [删除组帐号](#删除组帐号)
  - [计划任务](#计划任务)
  - [基本权限与归属](#基本权限与归属)
  - [查看数据权限](#查看数据权限)
  - [修改权限](#修改权限)
  - [修改归属关系](#修改归属关系)
  - [ACL策略管理](#acl策略管理)
  - [附加权限(特殊权限)](#附加权限特殊权限)
  - [逻辑卷](#逻辑卷)
  - [制作逻辑卷](#制作逻辑卷)
  - [逻辑卷的扩展](#逻辑卷的扩展)
  - [逻辑卷的补充](#逻辑卷的补充)
  - [进程管理](#进程管理)
  - [查看进程信息](#查看进程信息)
  - [控制进程（进程前后台的调度）](#控制进程进程前后台的调度)
  - [干掉进程的不同方法](#干掉进程的不同方法)
  - [sudo提权](#sudo提权)
  - [总结补充内容](#总结补充内容)
- [复习问题总结](#复习问题总结)
  - [yum查询库中有哪些程序](#yum查询库中有哪些程序)
  - [Liunx中判断用户具备的权限](#liunx中判断用户具备的权限)
- [预习下周内容](#预习下周内容)

# 今日计划

- [x] 复习上周全部内容及练习题重新做一遍

- [x] 补充笔记

- [x] 复习下周一二内容

# 本周复习内容

## RPM软件包简介

RPM包文件名特征

firefox-91.9.0-1.el8_5.x86_64

软件名-软件版本-系统版本-系统架构

## RPM包一般安装的位置

两个字：分散

普通执行程序：/usr/bin/、/bin

服务器程序、管理工具：/usr/sbin/、/sbin/

配置文件：/etc/、/etc/软件名/

日志文件：/var/log/、/var/log/软件名/

程序文档、man帮助手册页：/usr/share/doc/、/usr/share/man

## 查询软件信息

**查询是否安装此软件包**

格式：rpm -q 软件包名

```linux
]#rpm -q firefox # 查询系统firefox是否安装
firefox-91.9.0-1.el8_5.x86_64
]#rpm -q httpd
未安装软件包 httpd 
]#rpm -qa # 列出系统安装的所有包
]#rpm -qa | wc -l #统计包数量
]#rpm -qa | grep firefox
firefox-91.9.0-1.el8_5.x86_64
```

**查询软件信息**

格式：

rpm -qi 软件包名

rpm -ql 软件包名

```linux
]#rpm -qi firefox # 查询firefox软件安装详情
Name        : firefox
Version     : 91.9.0
Release     : 1.el8_5
Architecture: x86_64
]#rpm -ql firexo # 列出软件安装清单
/usr/lib64/firefox/browser/extensions/langpack-ta@firefox.mozilla.org.xpi
/usr/lib64/firefox/browser/extensions/langpack-te@firefox.mozilla.org.xpi
/usr/lib64/firefox/browser/extensions/langpack-th@firefox.mozilla.org.xpi
]#rpm -ql firefox | wc -l
```

**查询某个目录/文件是那个rpm包带来的**

格式：rpm -qf 文件路径

[注]：即使文件删除也能查询

```linux
]#which vim # 查询对应命令的程序文件
/usr/bin/vim
]#rpm -qf /usr/bin/vim
vim-enhanced-8.0.1763-16.el8_5.13.x86_64
]#rpm -qf /usr/sbin/ifconfig
]#rpm -qf /usr/sbin/poweroff
```

**查询软件包**

作用：可查询未安装软件包的软件信息

前提：系统需要挂载

格式：

rpm -qpl 未安装的软件包

rpm -qpi 未安装的软件包

```
]#rpm -q vsftp
未安装软件包 vsftp 
]#rpm -qpl /mnt/AppStream/Packages/v/vsftpd-3.0.3-35.el8.x86_64.rpm 
/usr/share/doc/vsftpd/vsftpd.xinetd
/usr/share/man/man5/vsftpd.conf.5.gz
/usr/share/man/man8/vsftpd.8.gz
/var/ftp
/var/ftp/pub
]# rpm -qpi /mnt/AppStream/Packages/v/vsftpd-3.0.3-35.el8.x86_64.rpm
警告：/mnt/AppStream/Packages/v/vsftpd-3.0.3-35.el8.x86_64.rpm: 头V4 RSA/SHA256 Signature, 密钥 ID 6d745a60: NOKEY
Name        : vsftpd
Version     : 3.0.3
Release     : 35.el8
Architecture: x86_64
```

**安装软件包**

格式：rpm -i rpm软件包文件

-v 显示细节信息

-h 显示安装进度

```
]#rpm -ivh vsftpd-3.0.3-35.el8.x86_64.rpm 
Verifying...                          ################################# [100%]
准备中...                          ################################# [100%]
正在升级/安装...
# rpm -q vsftpd
vsftpd-3.0.3-35.el8.x86_64
```

**卸载软件**

格式：rpm -e 软件名

```
# rpm -e vsftpd 
# rpm -q vsftpd
未安装软件包 vsftpd 
```

**导入红帽签名信息**

作用：上方查询未安装软件包信息时有个警告信息，红帽的提示，安装红帽以外的软件包需要红帽密钥，将系统密钥导入，取消警告

```
]# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-rockyofficial #导入签名信息
]# rpm -qpi /mnt/AppStream/Packages/v/vsftpd-3.0.3-35.el8.x86_64.rpm # 验证是否成功
Name        : vsftpd
Version     : 3.0.3
Release     : 35.el8
Architecture: x86_64
Install Date: (not installed)
```

## 软件包的依赖关系

rm命令方式：不能自动解决依赖关系

yum命令方式：能自动解决依赖关系

## YUM仓库构建

**以下操作基于，光盘已挂载**

仓库配置文件：/etc/yum.repos.d/*.repo

[注]：错误的文件会影响正确的文件

提示：一般构建仓库时，将/etc/yum.repos.d/所有内容删除或则放在一个文件夹

**yum命令执行流程**

yum命令-》读取/etc/yum.repos.d/*.repo配置文件内容-》从而找到仓库的具体位置

**仓库文件配置内容**

[源名称]：自定义仓库名称，具有一定意义

name：本地软件仓库的描述

baseurl：指定yum服务端的url地址

enabled：是否启用

gpgcheck：是否验证待安装的rpm包

gpgkey：用于rpm软件包验证的密钥文件

示例：

```
[local]
name=locals
baseurl=file:///mnt/AppStream #指定仓库位置file://表示本地服务端
enabled=1 # 仓库启用，默认1启用，可以省略
gpgcheck=1 启用验证，默认1启用，不起用输入0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-rockyofficial #启用验证需要书写签名信息，不启用可不书写
```

**验证仓库**

作用：验证仓库配置信息是否正确

```
]# yum repoinfo # 查询仓库详情
cky Linux 8 - BaseOS                          528 kB/s | 7.2 MB     00:14    
Rocky Linux 8 - Extras                           14 kB/s |  14 kB     00:01    
locals                                           18 MB/s | 7.8 MB     00:00    
local2s                                         7.6 MB/s | 2.6 MB     00:00    
仓库ID            : appstream
仓库名称          : Rocky Linux 8 - AppStream
Repo-revision      : 8.8
Repo-distro-tags      : [cpe:/o:rocky:rocky:8]:  ,  , 8, L, R, c, i, k, n, o, u,
                      : x, y
Repo-updated       : 2023年11月09日 星期四 01时59分28秒
```

**安装软件包**

格式：

yum install 软件包名

yum -y install 软件包名（不用手动输入y）

```
]# yum -y install httpd
上次元数据过期检查：0:02:16 前，执行于 2023年11月11日 星期六 14时05分11秒。
依赖关系解决。
]# rpm -q httpd # 检测是否安装成功
httpd-2.4.37-56.module+el8.8.0+1456+d0a01c5e.7.x86_64
```

**卸载软件**

格式：

yum remove httpd

yum -y remove httpd（建议不使用）

```
]# yum remove httpd
依赖关系解决。
================================================================================
 软件包        架构   版本                                     仓库        大小
================================================================================
移除:
 httpd         x86_64 2.4.37-56.module+el8.8.0+1456+d0a01c5e.7 @appstream 4.3 M
 ]# rpm -q httpd # 检测是否卸载
未安装软件包 httpd 
```

**仓库查询**

```
]# yum list httpd # 查询仓库是否存在
上次元数据过期检查：0:03:01 前，执行于 2023年11月11日 星期六 14时07分50秒。
可安装的软件包
httpd.x86_64         2.4.37-56.module+el8.8.0+1456+d0a01c5e.7          appstream

]# yum search httpd # 包含就ftp就返回
上次元数据过期检查：0:03:51 前，执行于 2023年11月11日 星期六 14时07分50秒。
============================= 名称 精准匹配：httpd =============================
httpd.x86_64 : Apache HTTP Server
=========================== 名称 和 概况 匹配：httpd ===========================
keycloak-httpd-client-install.noarch : Tools to configure Apache HTTPD as

]# yum provides /usr/bin/hostname #查询仓库中那个软件包产生该文件
上次元数据过期检查：0:05:51 前，执行于 2023年11月11日 星期六 14时07分50秒。
hostname-3.20-6.el8.x86_64 : Utility to set/show the host name or domain name
仓库        ：@System
匹配来源：
文件名    ：/usr/bin/hostname
hostname-3.20-6.el8.x86_64 : Utility to set/show the host name or domain name
仓库        ：baseos
匹配来源：
文件名    ：/usr/bin/hostname
hostname-3.20-6.el8.x86_64 : Utility to set/show the host name or domain name
仓库        ：local2
匹配来源：
文件名    ：/usr/bin/hostname
```

**程序恢复**

作用：不小心删除安装软件的一些文件，可进行恢复

格式：yum -y reinstall 软件包

```
]# rm -rf /usr/bin/zip
]# ls /usr/bin/zip
ls: 无法访问'/usr/bin/zip': 没有那个文件或目录
]# yum provides /usr/bin/zip # 查询那个软件包能产生该文件
上次元数据过期检查：0:09:40 前，执行于 2023年11月11日 星期六 14时07分50秒。
zip-3.0-23.el8.x86_64 : A file compression and packaging utility compatible with
                      : PKZIP
仓库        ：@System
匹配来源：
文件名    ：/usr/bin/zip
zip-3.0-23.el8.x86_64 : A file compression and packaging utility compatible with
                     : PKZIP
仓库        ：baseos
匹配来源：
文件名    ：/usr/bin/zip
zip-3.0-23.el8.x86_64 : A file compression and packaging utility compatible with
                      : PKZIP
仓库        ：local2
匹配来源：
文件名    ：/usr/bin/zip
]# yum -y reinstall zip-3.0-23.el8.x86_64 # 覆盖安装
]# ls /usr/bin/zip
/usr/bin/zip
```

**清空缓存**

作用：缓解内存，第一次使用yum repoinfo 会将信息存储到内存

格式：yum clean all

```
]# yum clean all
38 文件已删除
```

## 获取帮助命令

**方式一help**

```
]# cat --help
```

**方式二man**

```
]# man cat # 键盘上下进行查看，可输入/examples 查看使用案例，按q退出
]# man 5 passwd #5  查看配置文件的帮助信息
```

## 历史命令

作用：管理/调用曾经执行过的命令

格式：history [选项]

```
]# history # 查看历史命令
]# !2 # 执行历史命令中的第2条命令
]# !hi # 执行最近一次以hi开头的历史命令
]# history -c # 清空开机到现在的历史命令
]# history -c && history -w# 清空所有历史命令

```

## 系统时间

作用：查看或修改系统时间

格式：date [选项]

```
]# date # 查询当前系统时间
]# date -s '2022-11-1 13:30:20' # 修改系统时间
2022年 11月 01日 星期二 13:30:20 CST
]# date +%Y # 显示年
2023
]# date +%m # 显示月
11
]# date +%d # 显示日
01
]# date +%H # 显示小时
14
]# date +%M # 显示分
33
]# date +%S # 显示秒
40
]# date +%F # 显示年与日
2023-11-01
]# date +%T # 显示时间
14:33:52
```

## 统计磁盘大小

格式：du [选项]

```
]# du -s /etc # 统计每个参数所占用的工具大小
31320   /etc
]# du -sh /etc # 提供易读单位
31M     /etc
```

## 制作链接文件

**软链接**

格式：ln -s /路径/源数据 /路径/快捷方式的名称

```
]# ln -s /etc/passwd /opt/test
]# ls -l /opt/test 
lrwxrwxrwx. 1 root root 11 11月  1 14:41 /opt/test -> /etc/passwd
```

软链接缺点：源数据消失，快捷方式失效

软链接优势：可以针对目录与文件制作快捷方式，支持跨分区

**硬链接**

格式：ln /路径/源数据 /路径/快捷方式的名称 #硬链接

```
]# ln /opt/a.txt /opt/b.txt 
]# ln -s /opt/a.txt /opt/c.txt # 软链接
]# ls -i /opt/ # 查看内存地址
36383146 a.txt  36383146 b.txt  36383147 c.txt  33856367 test
]# rm -rf /opt/a.txt  # 删除源文件
]# cat /opt/b.txt   # 硬链接 访问
123
]# cat /opt/c.txt  # 软链接访问
cat: /opt/c.txt: 没有那个文件或目录
```

硬链接优势：源数据消失，快捷方式仍然有效
硬链接缺点：只能针对文件制作快捷方式，不支持支持跨分区

## zip归档工具

作用：跨平台解压（Windows和Linux）

**压缩**

格式：zip [-r]：被压缩的数据有目录必须加上 压缩包 被归档的文档

```
]# zip -r /a.zip /etc/passwd /home
]# ls /
a.zip  boot/  etc/   lib/   media/ opt/   root/  sbin/  sys/   usr/   
bin/   dev/   home/  lib64/ mnt/   proc/  run/   srv/   tmp/   var/   
```

**解压**

格式：

unzip -l 压缩包  查看压缩包内容

unzip 压缩包 -d 解压路径

```
]# unzip -l /a.zip 	
]# unzip /a.zip -d /opt/
```

## 用户配置文件

作用：/etc/passwd：存放用户基本信息配置文件

牢记七个字段含义

用户名**:**密码占位符**:**UID**:**基本组GID**:**用户描述信息**:**家目录**:**解释器

## 用户的创建

格式：useradd [选项] 参数 用户名 

-d：指定家目录

-u：指定用户ID

-g：指定基本组

-G：指定附加组

-s：指定解释器

> LInux上两种解释器：
>
> 1. /bin/bash  默认
> 2. /sbin/nologin 禁止用户登录

## usermod命令

格式：usermod [选项] 参数 用户名

-l：修改用户帐号登录名称

```
]# usermod -l yh yyh
]# id yh
uid=1000(yh) gid=1000(yyh) 组=1000(yyh)
```

-u：修改UID

```
]# usermod -u 1001 yh
]# id yh
uid=1001(yh) gid=1000(yyh) 组=1000(yyh)
```

-G：修改附加组ID

-d：修改家目录

-s：修改解释器

```
]# usermod -s /sbin/nologin yh
```

## 设置用户密码

格式：passwd [选项] 用户名

`--`stdin：应用无交互修改密码

**交互修改密码**

```
]# passwd yh  # 管理员修改密码，可不遵循密码规则
更改用户 yh 的密码 。
新的 密码：
无效的密码： 密码是一个回文
重新输入新的 密码：
passwd：所有的身份验证令牌已经成功更新

]$ passwd # 普通用户修改密码
更改用户 yh 的密码 。
Current password: 
当前密码： 
```

**非交互修改密码**

只有管理员能操作

```
]# echo 123 | passwd --stdin yh
更改用户 yh 的密码 。
passwd：所有的身份验证令牌已经成功更新。
```

## 用户初始配置文件

/etc/shadow：存放用户密码配置信息文件

]# grep yh /etc/shadow
yh:$6$WtyG5h1WFi1D6kmD$QUsjM4MOGUZvSKsSXITpOC33nRz1smHUUcXgVQV3UP3FzqOdBngZKeeWVryMIybDbJswLygbZIuf3MbwSwekx1:19662:0:99999:7:::

上一次修改密码的时间：自1970-1-1到达上一次修改密码的时间，所经历的天数

字段1：用户帐号的名称

字段2：加密后的密码字符串

字段3：上次修改密码的时间

字段4：密码的最短有效天数，默认0

字段5：密码的最长有效天数，默认99999

字段6：密码过期前的警告天数，默认7

字段7：密码过期后多少天禁用此用户账号

字段8：帐号失效时间，默认值为空

## 删除用户

userdel  [选项] 用户名

-r：家目录/用户邮件也一并删除

```
]# ls -l /home
总用量 4
drwx------.  3 test test   78 11月  1 15:52 test
drwx------. 15 yh   yyh  4096 11月  1 11:33 yyh

]# id test
id: “test”：无此用户

]# userdel -r test
]# ls -l /home
总用量 4
drwx------. 15 yh yyh 4096 11月  1 11:33 yyh
```

## 组账户管理

/etc/group：存放组帐号基本信息

```
]# grep yyh /etc/group
yyh:x:1000:
组名：组密码占位符：组的GID：组成员列表
```

/etc/gshadow：组的管理信息配置文件

```
]# grep school /etc/gshadow
school:!:stu3:stu1,stu2
组名：组密码占位符：组管理员：组成员列表
```

gpasswd [选项]  用户名 组名

-a：添加组成员

```
]# useradd stu1
]# groupadd school
]# id stu1
uid=1002(stu1) gid=1002(stu1) 组=1002(stu1)

]# gpasswd -a stu1 school
正在将用户“stu1”加入到“school”组中

]# id stu1
uid=1002(stu1) gid=1002(stu1) 组=1002(stu1),1001(school)
```

-d：删除组成员

```
]# gpasswd -d stu1 school
正在将用户“stu1”从“school”组中删除

]# id stu1
uid=1002(stu1) gid=1002(stu1) 组=1002(stu1)
```

-M：定义（重置）组成员列表，可设置多个用户

原有组将被重置

```
]# useradd stu2
]# gpasswd -M 'stu1,stu2' school
]# id stu1
uid=1002(stu1) gid=1002(stu1) 组=1002(stu1),1001(school)

]# id stu2
uid=1003(stu2) gid=1003(stu2) 组=1003(stu2),1001(school)

]# grep school /etc/group
school:x:1001:stu1,stu2

]# gpasswd -M '' school # 删除组所有成员
]# grep school /etc/group
school:x:1001:
```

-A：定义组管理员

管理员可以不属于此组，可设置一个或多个，添加或删除参考-M选项

```
]# useradd stu3
]# id stu3
uid=1004(stu3) gid=1004(stu3) 组=1004(stu3)

]# gpasswd -M 'stu1,stu2' school
]# grep school /etc/group
school:x:1001:stu1,stu2

]# gpasswd -A stu3 school
]# grep school /etc/gshadow
school:!:stu3:stu1,stu2
```

## 删除组帐号

格式：groupdel 组名

```
]# groupdel school
```

[注]：删除组的时候，不可以删除基本组

## 计划任务

作用：按照设置的时间间隔，为用户反复执行某一项系统任务

特别写法

分时日月周

5 * * * * #每个小时的第五分钟执行一次

*/5 * * * * #每隔5分钟运行一次

38 */2 * * * #每隔2小时运行一次

*：匹配范围内任意时间

,：分隔多个不连续的时间点

-：指定连续时间范围

/n：指定时间频率，每n ...

格式：

编辑：crontab -e

```
]# crontab -e # 编写一个每2分钟记录系统时间任务保存到1.txt
*/2 * * * * date >> /opt/1.txt

]# cat /opt/1.txt 
2023年 11月 01日 星期三 16:22:01 CST
2023年 11月 01日 星期三 16:24:01 CST
```

查看：crontab -l

```
]# crontab -l
*/2 * * * * date >> /opt/1.txt
```

/var/spool/cron 任务存放的路径

/var/spool/cron/root 记录root用户计划任务

清除：crontab -r

```
]# crontab -r
```

## 基本权限与归属

**基本权限**

读取：允许查看文件         r（ls、cat、less）

写入：允许辑文件        w（删除、创建、编辑）

可执行：允许允许和切换  x（对于目录：能够cd切换到此目录）

**归属关系**

所有者：拥有此文件/目录的用户 u

所属组：拥有此文件/目录的组  g

其他用户：除所有者、所属组以外的用户 o

```
]# ls -l /opt/1.txt 
-rw-r--r--. 1 root root 86 11月  1 16:24 /opt/1.txt
1：root：所有者
2：root：所属组
```

## 查看数据权限

ls -l或ls-d

```
]# ls -l /opt/1.txt 
-rw-r--r--. 1 root root 86 11月  1 16:24 /opt/1.txt
- ：以-开头为文本文件，d开头为目录，l开头为快捷方式（软硬链接）
rw-：所有者拥有-》读与写
r--：所属组拥有-》读
r--：其他用户拥有-》读
```

## 修改权限

格式：chmod [选项] [ugoa] [+-=] [rwx] 文件

```
]# mkdir test
]# ls -ld test/  
drwxr-xr-x. 2 root root 6 11月  1 17:08 test/

]# chmod u-w test/  # 所有者去掉w权限
]# ls -ld test/
dr-xr-xr-x. 2 root root 6 11月  1 17:08 test/

]# chmod u-w test/  # 所有者加上w权限
]# ls -ld test/
rwxr-xr-x. 2 root root 6 11月  1 17:08 test/

]# chmod g+w test/ # 所属组加上w权限
]# ls -ld test/
drwxrwxr-x. 2 root root 6 11月  1 17:08 test/

]# chmod g=rw test/ # 所属组重新定义权限
[root@test ~]# ls -ld test/
drwxrw-r-x. 2 root root 6 11月  1 17:08 test/

]# chmod a=w test/ #重新定义权限，所有人都只有w权限
[root@test ~]# ls -ld test/
d-w--w--w-. 2 root root 6 11月  1 17:08 test/

```

案例：新建一个p目录权限设置成rwxr-x---

```
]# mkdir p
[root@test ~]# chmod u=rwx,g=rx,o=--- p
[root@test ~]# ls -ld p
drwxr-x---. 2 root root 6 11月  1 17:26 p
```

-R：递归修改权限

```
]#  mkdir -p  /work/app
]# ls -ld /work/app/
drwxr-xr-x. 3 root root 16 11月  1 17:21 /work/app/
]# chmod -R o+w /work/app/ # 递归给目录其他用户添加一个w权限
[root@test ~]# ls -ld /work/app/
drwxr-xrwx. 3 root root 16 11月  1 17:21 /work/app/
[root@test ~]# ls -ld /work/app/aa/
drwxr-xrwx. 3 root root 16 11月  1 17:21 /work/app/aa/

[root@test ~]# ls -ld /work/app/aa/bb/
drwxr-xrwx. 2 root root 6 11月  1 17:21 /work/app/aa/bb/
```

**权限利用数字方式表示**

r、w、x分别对于4、2、1,后三组分别求和

分组：所有者、所属组、其他用户

字符：rwx、r-x、---

数字：421、41、0

求和：7 5 0

```
]# chmod 750 p
[root@test ~]# ls -ld p
drwxr-x---. 2 root root 6 11月  1 17:26 p
```

## 修改归属关系

格式：

chown 属主 文件

```
]# mkdir /nsd15
[root@test ~]# groupadd tmooc
[root@test ~]# useradd lisi
[root@test ~]# ls -ld /nsd15/
drwxr-xr-x. 2 root root 6 11月  1 17:37 /nsd15/

]# ls -ld /nsd15/
drwxr-xr-x. 2 lisi root 6 11月  1 17:37 /nsd15/
```

chown 属主:属组 文件

```
]# ls -ld /nsd15/
drwxr-xr-x. 2 lisi tmooc 6 11月  1 17:37 /nsd15/
```

chown :属组 文件

```
]# chown :root /nsd15/
]# ls -ld /nsd15/
drwxr-xr-x. 2 lisi root 6 11月  1 17:37 /nsd15/
```

-R：递归修改归属关系

```
]# chown -R lisi /work/app/
[root@test ~]# ls -ld /work/app/
drwxr-xrwx. 3 lisi root 16 11月  1 17:21 /work/app/
```

## ACL策略管理

作用：能过对个别用户、个别组设置独立的权限，类似白黑名单

格式：

setfacl [选项] u:用户名:权限 文件

setfacl [选项] g:用户名:权限 文件

**查看文件是否拥有A权限**

```
]# getfacl /test/
getfacl: Removing leading '/' from absolute path names
# file: test/
# owner: root
# group: root
user::rwx
user:dc:rwx
group::r-x
mask::rwx
other::r-x
```

-m：修改ACL策略

```
]# mkdir /p
[root@test ~]# chmod 770 /p
[root@test ~]# ls -ld /p
drwxrwx---. 2 root root 6 11月  1 17:49 /p
[root@test ~]# su dc
[dc@test root]$ cd /p
bash: cd: /p: 权限不够
]$ exit
exit
[root@test ~]# setfacl -m u:dc:rx /p
[root@test ~]# su dc
[dc@test root]$ ls -ld /p
drwxrwx---+ 2 root root 6 11月  1 17:49 /p
```

-x：清除指定的ACL策略

```
]# getfacl /test/ # 查询到dc拥有a权
getfacl: Removing leading '/' from absolute path names
# file: test/
# owner: root
# group: root
user::rwx
user:dc:rwx
group::r-x
mask::rwx
other::r-x
]#setfacl -x u:dc /test
]# getfacl /test/
getfacl: Removing leading '/' from absolute path names
# file: test/
# owner: root
# group: root
user::rwx
group::r-x
mask::r-x
other::r-x
```

-b：清除所有已设置的ACL策略

```
]# setfacl   -b   /nsd22    #清除目录所有ACL策略
```

-R：递归设置ACL策略

```
]# setfacl -Rm    u:dc:rwx    /opt/aa
```

**ACL策略-黑名单**

```
]#mkdir /shares
]#chmod 777 /shares
]#setfacl -m u:dc:--- /shares #将dc用户设置为黑名单，就不给他任何权限
```

## 附加权限(特殊权限)

**粘滞位**

占用其他用户的x位

作用：设置了t权限的目录，即使用户拥有写入权限，也不能删除或改名其他用户文档，适用于目录，用来限制用户滥用写入权限

显示t或T，取决他人是否有x权限

```
]# mkdir /shares
]# chmod 777 /shares/
]# chmod o+t /shares/ # 给shares目录设置t权限
]# ls -ld /shares/
drwxrwxrwt. 2 root root 6 11月 17 07:30 /shares/

]# mkdir /shares/root1 # root用户在shares目录下创建root1目录
]# su - dc
]$ rm -rf /shares/root1/ #用户dc执行删除操作
rm: 无法删除'/shares/root1/': 不允许的操作
```

**SetGID权限**

作用：在一个具有SGID权限的目录下，新建的文件会自动继承父目录的属组身份

```
]# mkdir /nsd1
]# chown :open /nsd1/
 ls -ld /nsd1/
drwxr-xr-x. 2 root open 6 11月 17 07:43 /nsd1/

]# chmod g+s /nsd1/
]# ls -ld /nsd1/
drwxr-sr-x. 2 root open 6 11月 17 07:43 /nsd1/

]# mkdir /nsd1/abc1
]# ls -ld /nsd1/
drwxr-sr-x. 3 root open 18 11月 17 07:45 /nsd1/

]# touch /nsd1/1.txt
]# ls -ld /nsd1/1.txt 
-rw-r--r--. 1 root open 0 11月 17 07:45 /nsd1/1.txt
```

**SetUID权限**

作用：能够用来传递可执行程序所有者的身份及具备的权限

[注]：针对可执行程序文件、可执行程序所有者具备可执行权限，显示占用所有者的x位

```
[root@localhost ~]# which mkdir       //利用which找到mkdir命令的绝对路径
/bin/mkdir
[root@localhost ~]# cp /bin/mkdir /bin/mymd1    //复制并改名
[root@localhost ~]# ls -l /bin/mymd1             //查看是否生成mymd1
-rwxr-xr-x. 1 root root 49384 2月  27 10:34 /bin/mymd1
[root@localhost ~]# chmod u+s /bin/mymd1         //添加SUID权限
[root@localhost ~]# ls -l /bin/mymd1             //查看是否添加成功
-rwsr-xr-x. 1 root root 49384 2月  27 10:34 /bin/mymd1
```

```
[root@localhost ~]# id zhangsan             //查看zhangsan用户是否存在
uid=500(zhangsan) gid=500(zhangsan) 组=500(zhangsan)
[root@localhost ~]# su – zhangsan           //切换用户身份测试
[zhangsan@localhost ~]$ ls -l /bin/mkdir   //查看mkdir命令程序权限的划分
-rwxr-xr-x. 1 root root 49384 10月 17 2013 /bin/mkdir    //可以看到没有SUID
[zhangsan@localhost ~]$ mkdir snew01       //创建测试目录snew01
[zhangsan@localhost ~]$ ls -ld snew01/     //查看snew01权限及归属关系
drwxrwxr-x. 2 zhangsan zhangsan 4096 2月  27 10:40 snew01/  //属主与属组均是zhangsan
[zhangsan@localhost ~]$ ls -l /bin/mymd1   //查看mymd1命令程序权限的划分
-rwsr-xr-x. 1 root root 49384 2月  27 10:34 /bin/mymd1   //可以看到具备SUID
[zhangsan@localhost ~]$ mymd1 snew02        //创建测试目录snew02
[zhangsan@localhost ~]$ ls -ld snew02       //查看snew02权限及归属关系
drwxrwxr-x. 2 root zhangsan 4096 2月  27 10:47 snew02
```

## 逻辑卷

作用：整合分散空间，空间支持扩大

## 制作逻辑卷

前提：已加载一块80G硬盘，并且使用fdisk进行了分区

**建立卷组VG)**

卷组创建时自动创建物理卷

```
]# vgcreate systemvg /dev/vdb[1-3] # 将物理卷vdb1-3合并成一个卷组，取名为systemvg
  Physical volume "/dev/vdb1" successfully created.
  Physical volume "/dev/vdb2" successfully created.
  Physical volume "/dev/vdb3" successfully created.
  Volume group "systemvg" successfully created
```

**建立物理卷**

```
]# lvcreate -L 20G -n vo systemvg #创建一个20G的物理卷，起名为vo
  Logical volume "vo" created.
```

**使用物理卷(LV)**

```
]# mkfs.xfs /dev/systemvg/vo  # 格式化xfs文件系统
meta-data=/dev/systemvg/vo       isize=512    agcount=4, agsize=1310720 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=0 inobtcount=0
data     =                       bsize=4096   blocks=5242880, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
Discarding blocks...Done.

]# blkid /dev/systemvg/vo # 查看文件系统类型
/dev/systemvg/vo: UUID="6e5496c2-18ba-430d-bb07-536cb0b5b7b5" BLOCK_SIZE="512" TYPE="xfs"

]# vim /etc/fstab # 编写开机自动挂载
/dev/systemvg/vo        /a      xfs     defaults 0 0

]# mkdir /a # 创建挂载点
]# mount /dev/systemvg/vo /a # 挂载
]# df -h /a # 查询挂载情况
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/systemvg-vo   20G  175M   20G    1% /a
```

**总结：**

1. 添加硬盘
2. lsblk查询添加情况
3. fdisk 划分分区
4. vgcreate 创建卷组
5. lvcreate 创建逻辑卷
6. mkfs.xfs 格式逻辑卷
7. 挂载使用

## 逻辑卷的扩展

**卷组有充足空间**

xfs_growfs：xfs

resies2fs：ext4

```
]# vgs # 查询到卷组还有9.9G空余空间
  VG       #PV #LV #SN Attr   VSize   VFree 
  rl         1   2   0 wz--n- <19.00g     0 
  systemvg   3   1   0 wz--n- <29.99g <9.99g
  
]# vgs  # 查询当前逻辑卷
  vo   systemvg -wi-ao----  20.00g  
  
]# lvextend -L 25G /dev/systemvg/vo # 将逻辑卷扩展到25G
  Size of logical volume systemvg/vo changed from 20.00 GiB (5120 extents) to 25.00 GiB (6400 extents).
  Logical volume systemvg/vo successfully resized. 
 
]# vgs
  VG       #PV #LV #SN Attr   VSize   VFree 
  rl         1   2   0 wz--n- <19.00g     0 
  systemvg   3   1   0 wz--n- <29.99g <4.99g
  
]# lvs 
vo   systemvg -wi-ao----  25.00g 

]# df -h /a 
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/systemvg-vo   20G  175M   20G    1% /a

]# xfs_growfs /dev/systemvg/vo  # 刷新xfs文件系统
]# df -h /a
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/systemvg-vo   25G  211M   25G    1% /a
```

**卷组空间不足**

```
]# lsblk # 查询磁盘空间情况
vdb             252:16   0   80G  0 disk 
├─vdb1          252:17   0   10G  0 part 
│ └─systemvg-vo 253:2    0   25G  0 lvm  /a
├─vdb2          252:18   0   10G  0 part 
│ └─systemvg-vo 253:2    0   25G  0 lvm  /a
├─vdb3          252:19   0   10G  0 part 
│ └─systemvg-vo 253:2    0   25G  0 lvm  /a
├─vdb4          252:20   0    1K  0 part 
├─vdb5          252:21   0   15G  0 part 
├─vdb6          252:22   0   15G  0 part 
└─vdb7          252:23   0   20G  0 part 

]# vgs
  VG       #PV #LV #SN Attr   VSize   VFree 
  rl         1   2   0 wz--n- <19.00g     0 
  systemvg   3   1   0 wz--n- <29.99g <4.99g
  
]# vgextend systemvg /dev/vdb[5-7] # 扩展卷组空间，通过添加磁盘vdb还未格式分区的文件系统
  Physical volume "/dev/vdb5" successfully created.
  Physical volume "/dev/vdb6" successfully created.
  Physical volume "/dev/vdb7" successfully created.
  Volume group "systemvg" successfully extended
]# vgs
  VG       #PV #LV #SN Attr   VSize   VFree 
  rl         1   2   0 wz--n- <19.00g     0 
  systemvg   6   1   0 wz--n-  79.97g 54.97g
  
  扩展物理卷同上方卷组空间够的情况一样
```

代码解释：扩展逻辑卷是层层叠加，逻辑卷空间不足看卷组，卷组空间不足看物理卷，物理卷不够就关机添加

## 逻辑卷的补充

xfs文件系统不支持缩减

ext4文件系统支持缩减

卷组划分空间单位：PE，1PE=4M

```
]#vgdisplay systemvg
  PE Size               4.00 MiB
```

**创建卷组指定PE大小**

```
]#vgcrate -s 1M /dev/vdb[1-2]
```

默认创建卷组PE为4M

**修改存在卷组PE大小**

```
]# vgchange -s 1M systemvg
  Volume group "systemvg" successfully changed
  
]# vgdisplay systemvg
  PE Size               1.00 MiB
```

**根据PE大小创建逻辑卷**

```
]# lvcreate -l 98 -n  hat systemvg # 创建一个98PE大小名为hat的逻辑卷
Logical volume "hat" created.

]# lvs  
hat  systemvg -wi-a-----  98.00m 
```

**逻辑卷删除**

```
]# lvremove /dev/systemvg/vo # 删除正在挂载的vo卷
  Logical volume systemvg/vo contains a filesystem in use.

]# lvremove /dev/systemvg/hat #删除未挂载的hat卷
Do you really want to remove active logical volume systemvg/hat? [y/n]: y
  Logical volume "hat" successfully removed.
```

[注]：被挂载的逻辑卷不能删除，需卸载后才能删除，

**卷组删除**

```
]# vgremove systemvg # 删除卷组：前提基于卷组创建的物理卷需要全部删除
Do you really want to remove volume group "systemvg" containing 1 logical volumes? [y/n]: y
  Logical volume systemvg/vo contains a filesystem in use.
  
]# vgremove systemvg # 物理卷被删除后，执行删除卷组
  Volume group "systemvg" successfully removed
  
]#pvremove /dev/vdg{1,2,3,5,6,7} # 删除物理卷
```

查看进程信息

## 进程管理

程序：静态没有执行的代码，磁盘空间

进程：动态执行的代码，cpu与内存资源

父进程与子进程：树形结构

进程编号：PID

## 查看进程信息

**pstree**

```
systemd─┬─NetworkManager───2*[{NetworkManager}]
        ├─agetty
        ├─auditd───{auditd}
        ├─chronyd
        ├─crond
        ├─dbus-daemon───{dbus-daemon}
        ├─firewalld───{firewalld}
        ├─irqbalance───{irqbalance}
        ├─polkitd───5*[{polkitd}]
        ├─qemu-ga───{qemu-ga}
        ├─rsyslogd───2*[{rsyslogd}]
        ├─sshd───sshd───sshd─┬─bash───pstree
        │                    └─sftp-server
        ├─sssd─┬─sssd_be
        │      └─sssd_nss
        ├─systemd───(sd-pam)
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-udevd
        └─tuned───4*[{tuned}] 
```

代码解释：列出所有正在使用的进程，system为所有进程的父进程，pid永远为1

-p：列出对应进程的pid

```
]# pstree -p
systemd(1)─┬─NetworkManager(874)─┬─{NetworkManager}(879)
           │                     └─{NetworkManager}(880)
           ├─agetty(1487)
           ├─auditd(830)───{auditd}(831)
           ├─chronyd(862)
           ├─crond(1471)
           ├─dbus-daemon(855)───{dbus-daemon}(860)
           ├─firewalld(873)───{firewalld}(1099)
           ├─irqbalance(856)───{irqbalance}(859)
           ├─polkitd(853)─┬─{polkitd}(866)
]# pstree -p | grep sshd # 查询sshd的pid
           |-sshd(882)-+-sshd(1494)---sshd(1509)-+-bash(1510)---su(1846)---bash(1847)---vim(1876)
           |           `-sshd(1878)---sshd(1882)---bash(1883)-+-grep(1911)
           
]# pstree -p yyh # 查询用户yyh当前开启的进程
bash(1847)───vim(1876)           
```

-a：显示完整的命令行

```
]# pstree -a
systemd --switched-root --system --deserialize 17
  ├─NetworkManager --no-daemon
  │   └─2*[{NetworkManager}]
  ├─agetty -o -p -- \\u --noclear tty1 linux
  ├─auditd
  │   └─{auditd}
  ├─chronyd
  ├─crond -n
  ├─dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation--syslog-
  │   └─{dbus-daemon}
  ├─firewalld -s /usr/sbin/firewalld --nofork --nopid
  │   └─{firewalld}
  ├─irqbalance --foreground
  │   └─{irqbalance}
  ├─polkitd --no-debug
]# pstree -a yyh # 查询用户yyh正在操作的完整命令
bash
  └─vim /etc/1.txt  
  
]# pstree -pa yyh
bash,1847
  └─vim,1876 /etc/1.txt
```

**ps**

aux：显示进程详情信息

```
]# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  1.0 174800 13464 ?        Ss   20:28   0:01 /usr/lib/systemd/systemd --switc
root           2  0.0  0.0      0     0 ?        S    20:28   0:00 [kthreadd]
root           3  0.0  0.0      0     0 ?        I<   20:28   0:00 [rcu_gp]
root           4  0.0  0.0      0     0 ?        I<   20:28   0:00 [rcu_par_gp]
```

-elf：列出所有进程的父进程信息

PPID为父进程pid

```
]# ps -elf
F S UID          PID    PPID  C PRI  NI ADDR SZ WCHAN  STIME TTY          TIME CMD
4 S root           1       0  0  80   0 - 43700 do_epo 20:28 ?        00:00:01 /usr/lib/systemd/sys
1 S root           2       0  0  80   0 -     0 -      20:28 ?        00:00:00 [kthreadd]
1 I root           3       2  0  60 -20 -     0 -      20:28 ?        00:00:00 [rcu_gp]
```

**top**

动态查看工具

top [-d刷新秒数] [-U用户名]

```
]# top -d  1
```

按P进行CPU排序

按M进行内存排序

**pgrep**

格式：pgrep [选项] 查询条件

-l：输出进程名

```
]# pgrep -l ssh
882 sshd
```

-u：检索指定用户的进程

```
]# pgrep -u yyh
1945
1974
]# pgrep -lu yyh
1945 bash
1974 vim
```

-x：精确匹配完整的进程名（需输入准确的程序名）

```
]# pgrep -x ssh
1926

]# pgrep -l sh
575 kdmflush/253:0
584 kdmflush/253:1

]# pgrep -x sh
```

## 控制进程（进程前后台的调度）

&：正在运行的状态放入后台

```
]# sleep 100 &
[1] 2017

]# jobs
[1]+  运行中               sleep 100 &
```

ctrl+z：挂起当前进程，暂停放入后台

```
]# sleep 100
^Z
[2]+  已停止               sleep 10

]# jobs
[1]-  运行中               sleep 100 &
[2]+  已停止               sleep 100
```

jobs：查看后台任务列表

fg：将后台任务恢复到前台运行

```
]# fg 2
sleep 100

```

bg：激活后台被挂起的任务

```
]# jobs
[1]+  已停止               sleep 100

]# jobs
[1]+  已停止               sleep 100

[root@node2 ~]# bg 1
[1]+ sleep 100 &

[root@node2 ~]# jobs
[1]+  运行中               sleep 100 &
```

## 干掉进程的不同方法

ctal+c：中断当前命令程序

kill [-9] PID、kill [-9] %后台任务编号

```
]# pgrep -l vim
2022 vim

]# kill -9 2022

]# sleep 1000 &
[1] 2024

]# kill -9 %1
]# jobs
[1]+  已杀死               sleep 1000
```

killall [-9] 进程名

```
]# pgrep -l sleep
2025 sleep
2026 sleep
2027 sleep

]# killall -9 sleep
[1]+  已杀死               sleep 100
[2]   已杀死               sleep 1000
[3]-  已杀死               sleep 2000
```

pkill [-9] 查找条件 #包含就算

## sudo提权

作用：让普通用户具备root用户身份去执行某些操作

/etc/sudoers # 提权配置文件

```
]# su yyh
]$ mkdir /hh
mkdir: 无法创建目录 “/hh”: 权限不够

]#visudo # 给用户配置权限，此方法可以检查语法错误（100行下写入内容）
yyh     ALL=(root) /usr/bin/mkdir
用户	所有主机=(变成的身份) 可以执行的命令

]$ sudo -l
用户 yyh 可以在 node2 上运行以下命令：
    (root) /usr/bin/mkdir
   
]$ sudo mkdir /test
[sudo] yyh 的密码：
[yyh@node2 root]$ ls -ld /test
drwxr-xr-x. 2 root root 6 11月 17 22:25 /test
```

**取消密码验证**

```
]#visudo # 添加一个NOPASSWD：
yyh     ALL=(root) NOPASSWD:/usr/bin/mkdir

]$ sudo mkdir /test1
]$ ls -ld /test1
drwxr-xr-x. 2 root root 6 11月 17 22:28 /test1
```



## 总结补充内容



# 复习问题总结

## yum查询库中有哪些程序

查询库中是否有firefox

yum list | grep firefox

## Liunx中判断用户具备的权限

1. 查看用户，对于该数据据所处的身份，顺序从u>g>o，匹配及停止
2. 查看相应身份的权限

# 预习下周内容


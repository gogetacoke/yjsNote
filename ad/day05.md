- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
  - [用户与组管理](#用户与组管理)
    - [用户帐号简介](#用户帐号简介)
    - [组帐号](#组帐号)
    - [groupadd](#groupadd)
    - [/etc/passwd](#etcpasswd)
    - [useradd](#useradd)
    - [usermod](#usermod)
    - [/etc/shadow](#etcshadow)
    - [userdel](#userdel)
    - [/etc/group](#etcgroup)
    - [gpasswd](#gpasswd)
    - [/etc/gshdow](#etcgshdow)
    - [grouodel](#grouodel)
  - [id](#id)
  - [cron](#cron)
  - [查询帮助信息](#查询帮助信息)
  - [zip](#zip)
- [课堂笔记（文本）](#课堂笔记文本)
  - [全局配置文件](#全局配置文件)
  - [Linux系统执行useradd命令，会完成那些操作？](#linux系统执行useradd命令会完成那些操作)
  - [Linux系统执行userdel -r命令，会完成那些操作？](#linux系统执行userdel--r命令会完成那些操作)
- [快捷键](#快捷键)
- [问题](#问题)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)

# 学习目标

用户管理

组帐号管理

计划任务

# 课堂笔记（命令）

## 用户与组管理

### 用户帐号简介

> 用户帐号作用：可以登陆操作系统，不同用户具备不同的权限
>
> 唯一标识：UID（编号从0开始编号 ，默认最大60000）
>
> 管理员root的UID：0
>
> 普通用户的UID：默认从1000凯撒
>
> 系统用户的UID：1-999
>
### 组帐号
>
> 作用：方便管理用户
>
> 唯一标识：GID
>
> *原则：Linux一个用户必须至少属于一个组*
>
> 基本句：一般情况与用户同名，一个用户必须有基本组，基本组只能有一个
>
> 附加组：一个用户可以有多个附加组，也可以没有附加组
>
### groupadd

>
> groupadd 组名
>
### /etc/passwd
>
> /etc/passwd：存放用户基本信息配置文件
>
> passwd中一条举例：
>
> root**:**x:0:0:root:/root:/bin**/**bash 
>
> 用户名**:**密码占位符**:**UID**:**基本组GID**:**用户描述信息**:**家目录**:**解释器
>
### useradd
>**[-u]：指定uid**
>
>useradd -u 1314 test1
>
>**[-d]：指定用户家目录（宿主目录）**
>
>useradd -d /opt/home1 test2  创建一个用户test2，家目录指定/opt/home1（不能提前创建用户目录、且用户目录不能为多层级为创建目录）
>
>grep test2 /etc/passwd    查询用户配置信息
>
>**[-g]：指定基本组（了解）**
>
>**[-G]：指定所属附加组，前提需要有组，或已创建组**
>
>groupadd stugrp 创建一个stugrp组
>
>useradd -G stugrp test3    指定用户test3的附加组为stugrp
>
>**[-s]：指定用户的登陆解释器，默认为/bin/bash**
>
>`/sbin/nologin：禁止用户登陆`
>
>useradd -s /sbin/nologin test4    指定用户未不登陆解释器
>
>`passwd test4 `   设置秘密，屏幕没有显示，需要输入两次密码


### usermod
>usermod -l stu1 test1 将用户test1的用户名改为stu1
>
>usermod -u 1600 stu1 更改用户uid
>
>usermod -s /sbin/nologin stu1 更改用户解释器
>
>**移动家目录**
>
>usermod -md  /移动位置/起名 被移动用户
>
>usermod -md /opt/dc15 test5  将用户test5的家目录移动到opt下取名dc15
>
>**重置用户附加组**
>
>groupadd xiaohang 创建一个xiaohang组
>
>usermod -G xiaohang test5 将xiaohang组加入用户test5的附加组中
>
>[注]：test5本没有xiaohang附加组，执行上诉命令将直接加入附加组，默认附加组不变
>
>**交互式密码修改**
>
>passwd test3 管理员修改密码，可不遵循密码规则
>
>su - test3 临时切换用户
>
>passwd 普通用户修改密码，必须遵守密码规则
>
>exit 退出当前用户，退回到root
>
>**非交互式修改密码**
>
>格式：echo 新密码 `|` passwd --stdin 用户名
>
>echo 123 `|` passwd --stdin test1 将用户test1密码修改为123

### /etc/shadow

> etc/shadow  存放用户密码
>
> 每个用户记录一行，以：分割为9个字段
>
> grep test7 /etc/shadow
>
> test7:$6$NVe937Nd$B0n94XrpQ.LipQHTpYh0iV/M4jCLdccfHxzRLprdxDzwk8WDDh/TzdTfh8lA9y9WKJ.8Ls/l5.w/1W.nV6CFX/:18481:0:99999:7:::
>
> 上一次修改密码的时间：自1970-1-1到达上一次修改密码的时间，所经历的天数
>
> 字段1：用户帐号的名称
>
> 字段2：加密后的密码字符串
>
> 字段3：上次修改密码的时间
>
> 字段4：密码的最短有效天数，默认0
>
> 字段5：密码的最长有效天数，默认99999
>
> 字段6：密码过期前的警告天数，默认7
>
> 字段7：密码过期后多少天禁用此用户账号
>
> 字段8：帐号失效时间，默认值为空
>
> 字段9：保留字段（未使用）

### userdel

> 作用：删除用户信息
>
> 格式：userdel [-r] 用户名
>
> [-r]：家目录/用户邮件也一并删除
>
> userdel -r test2

### /etc/group

> /etc/group 存放组的基本信息

### gpasswd

> 格式：gpasswd -选项 用户名 组名
>
> [-a]：添加用户到附加组
>
> [-d]：从组删除用户
>
> [-M]：多个用户添加（覆盖添加）
>
> [-A]： 将用户设置为组管理员
>
> 例：将test1加入group1组中
>
> gpasswd -a test1 group1
>
> 查询添加情况：grep test1 /etc/group
>
> 例：删除group1中的test1用户
>
> gpasswd -d test1 tarena
>
> 例：删除组中所有成员
>
> gpasswd -M '' group1
>
> 例：将用户test1设置为组管理员（除了root只有管理员才能修改添加成员）
>
> gpasswd -A test1 tarena



### /etc/gshdow

> 作用：组管理信息配置文件
>
> /etc/gshadow
>
> tarena**:!:**nb**:** 
>
> 组名**:**密码加密字符串**:**组的管理员列表**:**组成员列表
>
> 例：设置多个用户管理员
>
> gpasswd -A ‘hh，xx’ tarena 
>
> 例：删除所有管理员
>
> gpasswd -A ‘ tarena’

### grouodel

> groupdel 组名  （不能删除基本组）

## id 

> id test1 查询用户信息

## cron

> 作用：按照设置的时间间隔，为用户反复执行某一项固定的系统任务
>
> crontab 选项 用户名
>
> [-e]：编辑
>
> [-l]：查看
>
> [-r]：清除
>
> 计划任务书书写格式
>
> 分 时 日 月 周 任务命令行（绝对路径）
>
> `*` `*` `*` `*` `*` `*`  每分钟执行一次
>
> 30 8 `*` `*` `*` `*` 每天早上8：30执行一次
>
> 例：编写一个每分钟记录当前系统时间，写入到/opt/time.txt
>
> `*` `*` `*` `*` `*` `*`   date >> /opt/time.txt
>
> 任务文件存放路径：/var/spool/cron
>
> 查询记录root用户计划任务：cat /var/spool/cron/root

## 查询帮助信息

> 1. 命令 --help
> 2. man 命令
> 3. man 5 文件

## zip

> **压缩包**
>
> 格式：zip 选项  指定压缩路径 压缩文件
>
> [-r]：压缩
>
> 例如：将etc下hosts和passwd压缩到opt下
>
> zip -r /opt/test.zip /opt
>
> **解压缩**
>
> 格式：unzip 选项  压缩文件 -d 解压缩路径
>
> 例如将test.zip解压缩到opt下
>
> unzip /opt/test.zip -d /opt
>
> **查看压缩包文件**
>
> unzip -l /opt/test.zip

# 课堂笔记（文本）

## 全局配置文件

> **永久修改别名**
>
> /etc/bashrc （全局配置文件，修改影响全体用户）
>
> 例：配置一个别名，使全体用户都能使用
>
> vim /etc/bashrc
>
> alias my=‘echo 123’
>
> 保存并退出，打开新终端
>
> 输入 my

## Linux系统执行useradd命令，会完成那些操作？

1.会在/etc/passwd增加一行信息

2.会在/etc/shadow增加一行信息

3.会在/home新增用户家目录

4.会在/var/spool/mail增加用户邮件文件

5.会在/etc/group增加一行组信息

6.会在/etc/gshadow增加一行组的管理信息

## Linux系统执行userdel -r命令，会完成那些操作？

1.会在/etc/passwd删除一行信息

2.会在/etc/shadow删除一行信息

3.会在/home删除用户家目录

4.会在/var/spool/mail删除用户邮件文件

5.会在/etc/group删除一行组信息

6.会在/etc/gshadow删除一行组的管理信息

# 快捷键



# 问题



# 今日总结



# 昨日复习

1. 统计/etc目录大小，请写出命令

   du -sh /etc

2. LInux系统命令执行!vim作用是什么？

   执行历史命令中最近一条执行vim开头的历史命令

3. 如何查看系统时间

   date

4. 针对/opt目录制作一个链接文件，放在/root目录下

   ln -s /opt /root

5. 查询/usr/sbin/ifconfig程序是由那个软件安装产生

   rpm -qf /usr/sbin/ifconfig

6. 利用rpm命令检测firefox是否安装

   rpm -q firefox

7. 查询bash软件的安装清单

   rpm -ql bash

8. 利用yum命令重新安装软件hostname

   yum -y reinstall hostname

9. 利用yum安装httpd软件

   yum -y httpd

10. yum仓库配置文件放在什么路径下，具体的字段有哪些

    /etc/yum.repos.d/*.repo

    []：仓库名

    name：仓库描述信息

    baseurl：软件包地址

    enabled：是否启用

    gpgcheck：是否开启红帽安装认证

    gpgkey：红帽签名认证密钥
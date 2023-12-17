\- [学习目标](#学习目标)

\- [课堂笔记（命令）](#课堂笔记命令)

 \- [umask](#umask)

 \- [chown](#chown)

 \- [mkdir](#mkdir)

\- [课堂笔记（文本）](#课堂笔记文本)

 \- [ls-l显示详情](#ls-l显示详情)

 \- [基本权限](#基本权限)

  \- [文本文件](#文本文件)

  \- [目录](#目录)

 \- [归属关系](#归属关系)

 \- [数据权限](#数据权限)

 \- [修改权限](#修改权限)

 \- [数字表示权限](#数字表示权限)

 \- [修改归属关系](#修改归属关系)

 \- [Linux中判断用户具备的权限](#linux中判断用户具备的权限)

 \- [ACL策略](#acl策略)

 \- [ACL策略设置黑名单](#acl策略设置黑名单)

 \- [附加权限](#附加权限)

  \- [粘滞位 StickyBIt权限](#粘滞位-stickybit权限)

  \- [Set GID权限](#set-gid权限)

  \- [Set UID权限（了解）](#set-uid权限了解)

  \- [附加权-数字权限](#附加权-数字权限)

\- [快捷键](#快捷键)

\- [问题](#问题)

 \- [实现lisi用户可以读取/etc/shadow文件内容，您有几种办法？（至少写出三种以上）](#实现lisi用户可以读取etcshadow文件内容您有几种办法至少写出三种以上)

\- [总结](#总结)

# 学习目标

基本权限和归属

附加权限

ACL策略管理

# 课堂笔记（命令）

## umask

作用：决定用户创建目录的初始权限

参数：

[-S] 显示新建目录的默认权限

```linux
umask # 查询当前默认权限
0755
umask -S 
u=rwx,g=rx,o=rx
```

新建目录默认权限为：0755

新建文件目录权限：0644

举例：设置文件创建的默认权限（了解即可）

```linux
umask 0722
umask
0722
mkdir -S
u=,g=rx,o=rx
mkdri zz
ls -ld zz
d---r-xr-x. 2 root root 6 11月  8 17:58 zz 
```

补充：权限计算公式最高权限0777-umsk设置的权限，才是新建文件或目录的权限

## chown

详情查看文本笔**修改归属关系**

```linux
chown lisi:tmooc /nsd14
```

## mkdir

[-m]：新建文件时指定权限

```linux
mkdir -m 777 /nsd1
```

# 课堂笔记（文本）

## ls-l显示详情

使用ls -l查看文件权限关系

例：drwxr-xr-x. 140 root root 8192 11月  8 08:25 /etc/

设置了ACL策略例子：drwxr-xr-x+. 140 root root 8192 11月  8 08:25 /etc/abc

拆解以上显示内容：

> d: 当前查看的是目录 （/etc）
>
> 140：当前文件或目录大小（etc目录的大小）
>
> rwx：所属主拥有读写执行权限（第一个root）
>
> r-x：所属组拥有读执行权限无写权限 （第二个root）
>
> r-x：其他用户拥有读执行权限（需要我们新建一个用户来操作/etc）
>
> +： 当前目录或用户设置了ACL策略
>
> 补充：
>
> + 第一个-表示数据权限
> + 后面所有-每三个为一组，(属主,属组,所有者)
> + 三个都为-表示没有任何权限

## 基本权限

### 文本文件

+ 读取：允许查看内容 -read  **r**

  cat、less、head、tail、grep

+ 写入：允许修改诶日 -write **w**

  vim、>、>>

+ 可执行：允许运行和切换-excute **x**

  Shell、python

### 目录

r读取权限：ls命令查看目录内容

w写入权限：能够创建、删除、修改等目录的内容

x 执行权限：能够cd切换到此目录下（进入此目录）

## 归属关系

+ 所有者（属主）：拥有此文件/目录的用户 -user u
+ 所属组（属组）：拥有此文件/目录的组 -group  g
+ 其他用户：出所有者、所属组以外的用户 -other o
+ 所有：以上所有用户 -a

例如：zhangsan（所有者） zhangsan(所属组) 1.txt

## 数据权限

参考ls -l显示详情

`-`：文本文件

`d`：目录

`l`：快捷方式

## 修改权限

格式：chmod [ugoa] [+-=] [rwx] 文件 ...

常用命令选项

```linux
mkdir  /opt/abc # 先创建目录，以便操作
ls -ld /opt/abc # 查询当前创建目录的权限
drwxr-xr-x. 2 root root 6 11月 8 11：50 /opt/abc
chmod o=rx /opt/abc #设置目录abc其余用户权限只有读和执行权
chmod g=x /opt/abc #设置组用户访问目录abc读与执行权
chmod g+r /opt/abc #给组用户添加一个读取权限
chmod a=rwx #给所有人设置权限
chmod o=--- /opt/abc # 删除其余用户访问目录abc所有的权限
```

[-R] 递归修改权限

```linux
mkdir -P /opt/a/b/c 
chmod -R o=--- /opt/a
ls -ld /opt/a
drwxr-x---. 3 root root 15 11月  8 19:02 /opt/a
ls -ld /opt/a/b
ls: 无法访问'/opt/b': 没有那个文件或目录
ls -ld /opt/a/b/c
ls: 无法访问'/opt/c': 没有那个文件或目录
```

代码解释：将目录/opt/a的所有者权限以递归设置为---（没有任何权限）其a目录下的所有文件或目录都不能进行访问，再次使用mkdir -p /opt/a/b/c/d时d目录同样继承上诉权限

## 数字表示权限

权限位的8进制数表示

– r、w、x分别对应4、2、1，后3组分别求和

分组： User权限 Group权限 Other权限

字符： r w x    r - x         r - x

数字： 4 2 1   4 0 1       4 0 1

求和： 7 5 5

```linux
mkdir /test1
ls -ld /test1
drwxr-xr-x. 2 root root 6 11月  8 19:09 /test1/
chmod 700 /test1
ls -ld /test1
drwx------. 2 root root 6 11月  8 19:09 /test1/
```

代码解释：将目录test1的权限设置为7（主4+2+1）0（组）0（其他用户）

## 修改归属关系

格式：

+ chown 属主 文件
+ chown 属主:属组 文件
+ chown :属组 文件

```linux
mkdir /test2
ls -ld /test2
drwxr-xr-x. 2 root(属主) root(属组) 6 11月  8 19:09 /test2/ 
groupadd stu # 创建stu组
useradd yyh # 创建用户yyh
chown stu:yyh /test2 #修改test2的所有者与所属组
ls -ld /test2
drwxr-xr-x. 2 yyh stu 6 11月  8 19:21 /test2
useradd yyh1
groupadd stu1
chown yyh1 /test2 # 仅修改所有者
ls -ld /test2
drwxr-xr-x. 2 yyh1 stu 6 11月  8 19:21 /test2
chown :stu1 /test2 # 仅修改所属组
drwxr-xr-x. 2 yyh1 stu1 6 11月  8 19:21 /test2
```

代码解释：一并修改了目录test2的所有者与所属组，单独修改了主与组

[-R]：递归修改归属关系

```linux
mkdir -p /opt/aa/bb/cc
ls -ld /opt/aa
drwxr-xr-x. 3 root root 16 11月  8 19:29 /opt/aa
chown -R yyh:stu /opt/aa
ls -ld /opt/aa
drwxr-xr-x. 3 yyh stu 16 11月  8 19:29 /opt/aa
ls -ld /opt/bb
drwxr-xr-x. 3 yyh stu 16 11月  8 19:29 /opt/aa/bb/
```

代码解释：参考递归修该数据权限

## Linux中判断用户具备的权限

查看用户，对于该数据所处的身份，顺序所有者>所属组>其他人，原则是匹配及停止

例如：

​	192.168.1.1可以进入

​	所有客户端不可以进入

举例：

```linux
useradd test3
groupadd test
mkdir /testd
chown test3:test /testd # 设置testd目录的所有者与所属组为
ls -ld /testd
drwxr-xr-x. 3 test3 test 16 11月  8 19:29 /testd
chmod o= /testd #将testd目录的其余用户访问设置为空（没有rwx）
useradd test4
su - test4 # 切换test4用户
cd /testd
bash: cd: /testd: 权限不够
exit # 退出到上级用户
gpasswd -a test4 test # 将用户test4加入到test组中
su  - test4
cd testd
exit
gpasswd -a test3 test # 将当前所有者加入到test组中
chmod 450 /test # 设置权限读 读，执行 无权限
su - test3 #切换test3用户
cd /testd
bash: cd: /testd: 权限不够
```

代码解释：将目录testd所有者与所属组更改为test3:test 将其其他权限设置为空

用户test4访问testd则显示权限不够，将test4加入到组test中，这样test4将以组内成员进行访问，由ls -ld /testd 可知组用户拥有读与执行权，再次su test4 进入testd目录可正常访问，将testd目录的所有者test3加入到testd的组中，将权限更该为450,虽然所属组的权限5中包含读与执行权，但liunx权限的权限管具备关系，匹配及停止，他首先匹配他是主用户，主用户只有读权限，将直接停止访问下一级，报出权限不足。

## ACL策略

介绍：文档归属的局限性，属主、属组、其他人

作用：能够对个别用户，个别组设置独立的权限

格式：

setfacl [选项] u:用户名:权限 文件

setfacl [选项] g:组名:权限 文件

以下举例前提：

```linux
mkdir /nsd1

mkdri /nsd2

useradd dc

useradd ztt

useradd xtt

chmod 770 /nsd1 # 将其他人权限设置为空（不能正常访问）
```

常用命令选项：

[-m]：修改ACL策略

```linux
su - dc
cd /nsd1
-bash: cd: /nsd1: 权限不够
exit
setfacl -m u:dc:rx /nsd1 # 单独赋予dc用户权限
getfacl /nsd1 # 查询nsd1目录有那些acl策略
su - dc
cd /nsd1
exit
```

[-x]：清除指定ACL策略

```linux
getfacl /nsd1
setfacl -x u:dc /nsd1 # 删除指定用户ACL
```

[-b]：清除所有已设置的ACL策略

```linux
setfacl -b /nsd1
```

[-R]：递归设置ACL策略

```linux
setfacl -Rm u:dc:rwx /opt/aa
```

代码解释：将/opt/aa目录下所有者权限设置为rwx且用户为dc

[注]：针对已存在数据

## ACL策略设置黑名单

作用：单独拒绝某些用户

```linux
mkdir /home/public
chmod 777 /home/public
setfacl -m u:dc:--- /home/public
```

**代码解释：**将用户的家目录的所有者权限设置为空

## 附加权限

### 粘滞位 StickyBIt权限

作用及解释：

> + 占用其他人的x（执行权）位
>
> + 显示t或T，取决于其他人是否有x权限，有则t无则T
>
> + 适用于目录，用来限制用户滥用写入权
>
> + 在设置了t权限的目录下，即使用户有写入权限，也不能删除或改名其他用户文档

举例：

```linux
mkdir /nsd1 # 创建一个文件夹
chmod 777 /nsd1 # 通过数字设置文件夹为公共目录
ls -ld /nsd1 # 查看目录nsd1
drwxrwxrwx. 2 yyh yu1 19 11月  8 11:49 /nsd1
chmod o+t /nsd1 # 给nsd1目录设置t权限
drwxrwxrwt. 2 yyh yu1 19 11月  8 11:49 /nsd1
```

**代码解释：**将nsd1设置为公共目录，所有用户都可以在里面进行查看删除文件，设置一个t权限后，每个用户在里面只能对自己创建的文件进行修改删除，对于其余用户的文件只能查看

### Set GID权限

作用及解释：

+ 占用组的x位

+ 显示为s或S，取决于属组是否有可执行（x）权限

+ 对目录有效
+ 在一个具有SGID权限的目录下，新建的文档会自动集成副目录的属组身份

举例：

```linux
mkdir /nsd2
chmod g+s /nsd2 # 给目录设置sg权限
drwxrws---+ 2 root root 21 11月  8 14:56 /nsd2
mkdir /nsd2/abc
ls -ld /nsd2/abc
drwxr-sr-x. 2 root root 6 11月  8 17:13 /nsd2/abc
```

### Set UID权限（了解）

作用：

+ 占用属主（User）的 x 位

+ 显示为 s 或 S，取决于属主是否有 x 权限

+ 仅对可执行的程序有意义

+ 当其他用户执行带SUID标记的程序时，具有此程序属主的身份和相应权限

```linux
which mkdir
/usr/bin/mkdir
/usr/bin/mkdir   /opt/abc01  # 使用程序文件进行创建
/usr/bin/hahadir    /opt/abc02 # 将mkdir程序复制到opt目录下
```



### 附加权-数字权限

Set UID	4  (s)   

Set GID	2 （s）

StickyBIt	1 （t）

```linux
mkdir /nsd1
chmod 3755 /nsd1 # 3：sg权限+t权限 7：rwx 5：rx
```



# 快捷键



# 问题

## 实现lisi用户可以读取/etc/shadow文件内容，您有几种办法？（至少写出三种以上）

1. 利用其他人身份

   chmod o+r /etc/shadow

2. 利用所属组身份

   chown :lisi /etc/shadow

   chmod g+r /etc/shadow

3. 利用所有者身份

   chown lisi /etc/shadow

   chmod u+r /etc/shadow

4. 利用ACL策略

   setfacl -m u:lisi:r /etc/shadow



# 今日总结

1. 新建目录默认权限

   0755

# 昨日复习

1. 创建用户的命令是什么？

   useradd

2. 修改已存在用户的属性命令是什么？

   usermod

3. 非交互式lisi设置密码为123 请写出该命令

   echo 123 | passwd `--`stdin lisi

4. 请写出 存放用户基本信息 的配置文件

   /etc/passwd

5. 请写出 禁止用户登录系统的解释器程序

   /sbin/nologin

6. 将harry用户加入tedu组，请写出该命令

   gpasswd -a  harry tedu

7. 请写出 组账号基本信息 配置文件

   /etc/group

8. 如何判断一个用户是否存在？

   id 用户名

   查询passwd、shadow中是否有用户信息

9. 如何判断一个组是否存在？

   查询/etc/group文件

10. 如何判断一个用户是否在users组中？

    id查看用户的组

    查看/etc/group文件

11.  /etc/passwd文件第六字段与第七字段，分别表示的含义？

    家目录  解释器

12. 请写出您熟悉的Linux命令？（至少10条）

    cd、pwd、ls、du -sh、mkdir、cat、less、vim、useradd、usermod、groupadd、userdel、groupdel、gpasswd、tar、crontab、echo、man、history、find、touch
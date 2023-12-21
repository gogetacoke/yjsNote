- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [Ansible管理-环境准备](#ansible管理-环境准备)
    - [前提准备](#前提准备)
    - [装包](#装包)
    - [配置名称解析](#配置名称解析)
    - [配置ssh免密码](#配置ssh免密码)
  - [Ansible管理环境配置](#ansible管理环境配置)
    - [创建工作目录](#创建工作目录)
    - [创建配置文件](#创建配置文件)
    - [创建主机清单文件](#创建主机清单文件)
    - [测试](#测试)
    - [查看所有模块](#查看所有模块)
    - [创建用户测试](#创建用户测试)
    - [远程管理方法](#远程管理方法)
  - [adhoc临时命令](#adhoc临时命令)
    - [语法](#语法)
    - [执行后颜色](#执行后颜色)
    - [测试主机连通信](#测试主机连通信)
    - [command模块](#command模块)
    - [shell模块](#shell模块)
    - [script模块](#script模块)
    - [以上三个命令总结](#以上三个命令总结)
    - [file模块](#file模块)
      - [文件创建](#文件创建)
      - [目录创建](#目录创建)
      - [权限修改](#权限修改)
      - [文件/目录删除](#文件目录删除)
      - [创建软链接](#创建软链接)
    - [copy模块](#copy模块)
    - [fetch模块](#fetch模块)
    - [lineinfile模块](#lineinfile模块)
    - [replace模块](#replace模块)
  - [user模块](#user模块)
  - [group模块](#group模块)
- [快捷键](#快捷键)
  - [vim中如何快速补齐同字母内容](#vim中如何快速补齐同字母内容)
  - [快速复制](#快速复制)
  - [快速移动到指定位置](#快速移动到指定位置)
  - [更改上条命令快速执行](#更改上条命令快速执行)
  - [快速退出用户](#快速退出用户)
- [问题](#问题)
  - [touch创建文件](#touch创建文件)
  - [ansible管理远程主机有哪些方式](#ansible管理远程主机有哪些方式)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)


# 学习目标

Ansible基础

掌握Ansible ad-hoc命令

Ansible模块应用

# 课堂笔记（命令）



# 课堂笔记（文本）

## 读文前提

> 文中有\{\{\}\}的都加上了斜线进行转移，为了在github上显示，实际代码操作不用添加

## Ansible管理-环境准备

### 前提准备

> 准备四台主机：pubserver：88.240、web1：88.11、web2：88.12 、db1：88.13
>
> ansible包
>
> 四台yum搭建好，可以240机器使用网络yum-ftp

### 装包

```shell
yum -y install ansible
```

### 配置名称解析

```shell
[root@pubserver ~]# echo -e "192.168.88.240\tpubserver" >> /etc/hosts
[root@pubserver ~]# for i in 1 2
> do
> echo -e "192.168.88.1$i\tweb$i" >> /etc/hosts
> done
[root@pubserver ~]# echo -e "192.168.88.13\tdb1" >> /etc/hosts
[root@pubserver ~]# tail -4 /etc/hosts
192.168.88.240   pubserver
192.168.88.11    web1
192.168.88.12    web2
192.168.88.13    db1
```

### 配置ssh免密码

```shell
[root@pubserver /]# ssh-keygen
[root@pubserver /]# for i in web{1..2} db1;do ssh-copy-id $i;done # 为其余3个主机配置免密码登陆
```

## Ansible管理环境配置

### 创建工作目录

```shell
[root@pubserver /]mkdir ansible
[root@pubserver /]cd ansible
[root@pubserver /]touch ansible.cfg inventory
```

### 创建配置文件

```shell
[root@pubserver /]vim /ansible/ansible.cfg
[defaults]
inventory = inventory # 定义主机清单文件，为当前目录下的inventory
host_key_checking = false # 不检查SSH主机密钥的有效性
```

### 创建主机清单文件

```shell
[root@pubserver ansible]# vim inventory
# 创建主机清单文件。写在[]里的是组名，[]下面的是组内的主机名
[webservers]
web[1:2]   # web1和web2的简化写法，表示从1到2

[dbs]
db1

# cluster是组名，自定义的；:children是固定写法，表示下面的组名是cluster的子组。
[cluster:children]
webservers
dbs
```

### 测试

```shell
[root@pubserver ansible]# ansible all --list-hosts # 查看所有管理的主机
  hosts (3):
    web1
    web2
    db1
[root@pubserver ansible]# ansible webservers --list-hosts # 查看某一个组的主机
  hosts (2):
    web1
    web2
```

[注]：需要进入到ansible文件夹中执行

### 查看所有模块

```shell
ansible-doc -l
```

### 创建用户测试

```shell
[root@pubserver ansible]# ansible all -a "useradd zhangsan" # 为所有主机创建一个zhangsan用户

[root@web1 ~]# id zhangsan
uid=1000(zhangsan) gid=1000(zhangsan) groups=1000(zhangsan)
```

### 远程管理方法

> ansible进行远程管理的两个方法：
>
> + adhoc临时命令。就是在命令行上执行管理命令。
> + playbook剧本。把管理任务用特定格式写到文件中。
> 无论哪种方式，都是通过模块加参数进行管理。

## adhoc临时命令

### 语法

```shell
ansible 主机或组列表 -m 模块 -a "参数"
```

### 执行后颜色

> 绿色：表示已经成功完成的任务或操作。
> 黄色：表示需要注意或需要修改的内容。
> 蓝色：表示调试信息或执行进度。
> 红色：表示错误或失败的任务或操作。
> 紫色：表示模块正在进行异步操作。

### 测试主机连通信

> ping模块测试远程主机的连通性。

```shell
[root@pubserver ansible]#ansible all -m ping # 全为绿色的SUCCES表示成功
```

### command模块

> 默认模块，用于在远程主机上执行任意命令
>
> command不支持shell特性，如管道、重定向

```shell
#在所有主机上创建目录/tmp/demo
[root@pubserver ansible]#ansible all -m command -a "mkdir /tmp/demo"

[root@web1 ~]# ll /tmp/
demo systemd-private-294b4c963af04a579cd906cc81c04809-chronyd.service-SkROAg
```

### shell模块

> 与command模块类似，但是支持shell特性，如管道、重定向

```shell
# 查看所有主机IP前10行信息
[root@pubserver ansible]#ansible all -m shell -a "ip a s | head"
```

### script模块

> 用于在远程主机上执行脚本

```shell
# 为远程主机创建5个用户，且密码都设置为123456
[root@pubserver ansible]#vim test.sh
for i in user{1..5}
do
	useradd $i
	echo "123456" | passwd --stdin $i
done
[root@pubserver ansible]#ansible webservers -m script -a "test.sh"
```

代码解释：

> 以上编写一个脚本创建用户user1-5，并为每个用户设置密码123456
>
> 通过ansbile script模块将脚步分别在webservers(web1、web2)组下的所有主机分别执行该脚本
>
> 原理：主机执行该模块后，将test.sh脚本分别复制到web1,web2上使用bash test.sh执行脚本，执行完毕后自动删除脚本

### 以上三个命令总结

> 不建议使用，因为没有幂等性

### file模块

> - 可以创建文件、目录、链接等，还可以修改权限、属性等
> - 常用的选项：
>   - path：指定文件路径
>   - owner：设置文件所有者
>   - group：设置文件所属组
>   - state：状态。touch表示创建文件，directory表示创建目录，link表示创建软链接，absent表示删除
>   - mode：设置权限
>   - src：source的简写，源
>   - dest：destination的简写，目标

#### 文件创建

```shell
[root@pubserver ansible]#ansible webservers -m file -a "path=/tmp/file.txt state=touch" # 如果文件不存在，则创建；如果存在则改变它的时间戳
```

#### 目录创建

```shell
[root@pubserver ansible]#ansible webservers -m file -a "path=/tmp/demo state=directory" 
```

#### 权限修改

```shell
[root@pubserver ansible]# ansible webservers -m file -a "path=/tmp/file.txt owner=sshd group=adm mode='0777'" # 修改文件设置属主、属组，为文件夹设置权限
```

#### 文件/目录删除

```shell
[root@pubserver ansible]# ansible webservers -m file -a "path=/tmp/file.txt state=absent"#删除文件
```

#### 创建软链接

```shell
[root@pubserver ansible]# ansible webservers -m file -a "src=/etc/hosts dest=/tmp/hosts.txt state=link"
```

### copy模块

> - 用于将文件从控制端拷贝到被控端
> - 常用选项：
>   - src：源。控制端的文件路径
>   - dest：目标。被控制端的文件路径
>   - content：内容。需要写到文件中的内容
> - copy选项也支持权限属组修改

```shell
[root@pubserver ansible]# ansible webservers -m copy -a "src=/etc/passwd dest=/root/" # 将本机的/etc/passwd文件拷贝到所有主机的root目录下，空目录不可以

[root@pubserver ansible]# ansible webservers -m copy -a "dest=/tmp/mytest.txt content='Hello World\n'" # 在远程主机创建一个文件，并写入内容，文件可以存在或不存在
```

### fetch模块

> - 与copy模块相反，copy是上传，fetch是下载
> - 常用选项：
>   - src：源。被控制端的文件路径
>   - dest：目标。控制端的文件路径
>   - flat：扁平。bool；下载文件，且没有文件层级
> - 不能应用于目录

```shell
[root@pubserver ansible]# ansible webservers -m fetch -a "src=/etc/hostname dest=/root" #将远程主机的hostname下载到本机的root目录下
[root@pubserver ansible]# ls /root
web1 web2
```

### lineinfile模块

> - 用于确保存目标文件中有某一行内容
> - 常用选项：
>   - path：待修改的文件路径
>   - line：写入文件的一行内容
>   - regexp：正则表达式，用于查找文件中的内容
> - 匹配修改最后一个

```shell
[root@pubserver ansible]# ansible webservers -m lineinfile -a "path=/etc/issue line='hello word\n'" # 为目标主机/etc/issue文件添加两行内容
[root@web1 ~]#cat /etc/issue
\S
Kernel \r on an \m

hello word

[root@pubserver ansible]# ansible webservers -m lineinfile -a "path=/etc/issue line='chi le ma' regexp='hello'" # 查找hello行，将hello内容改为chi le ma；整行内容都将被替换成chi le ma
[root@web1 ~]#cat /etc/issue
\S
Kernel \r on an \m

chi le ma
```

### replace模块

> - lineinfile会替换一行，replace可以替换关键词
> - 常用选项：
>   - path：待修改的文件路径
>   - replace：将正则表达式查到的内容，替换成replace的内容
>   - regexp：正则表达式，用于查找文件中的内容
> - 匹配修改所有包含内容

```shell
[root@pubserver ansible]# ansible webservers -m replace -a "path=/etc/issue regexp='chi' replace='he'"
[root@web1 ~]#cat /etc/issue
\S
Kernel \r on an \m

he le ma
```

### user模块

> - 实现linux用户管理
> - 常用选项：
>   - name：待创建的用户名
>   - uid：用户ID
>   - group：设置主组
>   - groups：设置附加组
>   - home：设置家目录
>   - password：设置用户密码
>   - state：状态。present表示创建，它是默认选项。absent表示删除
>   - remove：删除家目录、邮箱等。值为yes或true都可以。

```linux
[root@pubserver ansible]# ansible webservers -m user -a "name=lisi uid=1010 groups=daemon,root home=/home/lisi" # 创建用户，指定所属组，附加组，家目录，uid

[root@pubserver ansible]# ansible webservers -m user -a "name=lisi password=\{\{'123456'|password_hash('sha512')\}\}" # 为用户创建密码
# \{\{\}\}是固定格式，表示执行命令。password_hash是函数，sha512是加密算法，则password_hash函数将会把123456通过sha512加密变成zhangsan的密码

# 删除zhangsan用户，不删除家目录
[root@pubserver ansible]# ansible webservers -m user -a "name=zhangsan state=absent"

# 删除lisi用户，同时删除家目录
[root@pubserver ansible]# ansible webservers -m user -a "name=lisi state=absent remove=yes"
```

### group模块

> - 创建、删除组
> - 常用选项：
>   - name：待创建的组名
>   - gid：组的ID号
>   - state：present表示创建，它是默认选项。absent表示删除

```shell
# 在webservers组中的主机上创建名为devops的组
[root@pubserver ansible]# ansible webservers -m group -a "name=devops"

# 在webservers组中的主机上删除名为devops的组
[root@pubserver ansible]# ansible webservers -m group -a "name=devops state=absent"
```



# 快捷键

## vim中如何快速补齐同字母内容

```shell
vim 1.txt
[AppStream]
name=A  # 按住键盘ctrl+n，将匹配文章中A开头的所有字母，可上下键盘选择
```

## 快速复制

> 双击选中需要复制的命令，鼠标滚轮按以下即可复制

## 快速移动到指定位置

> ctrl + 左右箭头，按照一个单词一个单词进行切换

## 更改上条命令快速执行

```shell
]#scp  /etc/yum.repos.d/mydvd.repo root@192.168.88.12:/etc/yum.repos.d/mydvd.repo  # 上条命令
]#^12^18 # 将替换上条命令12为18，并且执行
```

[注]：只是针对上一条命令，匹配的第一个

## 快速退出用户

> ctrl+d

# 问题

## touch创建文件

> 创建的文件不存在，就创建
>
> 创建的文件存在，就改变创建的时间

## ansible管理远程主机有哪些方式

> adhoc
>
> playbook

# 补充



# 今日总结

ansible两种管理方法：abhoc、playbook

adhoc方法格式：ansible 组或主机 -m 模块 -a 参数

模块：ping、command、shell、script、file、copy、fetch、lineinfile、replace、user、group

# 昨日复习
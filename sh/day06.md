- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
  - [tr](#tr)
- [课堂笔记（文本）](#课堂笔记文本)
  - [案例](#案例)
    - [监控脚本(简单)](#监控脚本简单)
    - [系统初始化](#系统初始化)
    - [通过文档批量创建帐号并配置密码](#通过文档批量创建帐号并配置密码)
    - [可视化安装软件脚本](#可视化安装软件脚本)
    - [编写每天备份文件脚本](#编写每天备份文件脚本)
  - [输入重定向](#输入重定向)
      - [`<`](#)
      - [`<<`](#-1)
      - [read](#read)
  - [脚本内部调用外部文件](#脚本内部调用外部文件)
- [快捷键](#快捷键)
- [问题](#问题)
  - [1 在.bash\_profile文件中定义的TMOUT变量有什么作用？](#1-在bash_profile文件中定义的tmout变量有什么作用)
  - [2 简述`<`、`<<`各自的作用。](#2-简述各自的作用)
  - [3 如何使用read同时定义多个变量？](#3-如何使用read同时定义多个变量)
  - [4 /dev/urandom文件有什么用处？可以在什么场合使用？](#4-devurandom文件有什么用处可以在什么场合使用)
  - [5 使用lftp时，如何上传文件？](#5-使用lftp时如何上传文件)
  - [6 打tar包时如果要排除所有txt格式的文件如何实现？](#6-打tar包时如果要排除所有txt格式的文件如何实现)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)




# 学习目标



# 课堂笔记（命令）

## tr

tr -cd命令是一个Unix和Linux系统上的文本处理工具，用于删除文本中不属于指定字符集的字符。这个命令通常与管道一起使用，例如：

```
shell复制代码echo "Hello, 123!" | tr -cd '0-9'
123
```

上面的命令将输出"123"，因为它使用tr -cd命令删除了输入字符串中不属于'0-9'范围内的字符。

tr表示translate（翻译）

-c表示取补集（complement）

-d表示删除（delete）

所以tr -cd命令的含义就是删除不属于指定字符集的字符。

# 课堂笔记（文本）

## 案例

### 监控脚本(简单)

```
#!/bin/bash
while :
do
    clear
    df -h| awk '/\/$/{print"根分区剩余容量是"$4}'
    ifconfig eth0|awk '/TX p/{print "网卡发送数据量是"$5"字节"}'
    free -h|awk '/^Mem/{print "内存剩余容量是"$4}'
    uptime | awk '{print "每15分钟cpu的平均负载是"$NF}'
    awk 'END{print "服务器账户总数是"$NR"个"}' /etc/passwd
    who | awk 'END{print NR"人正在使用服务器"}'
    rpm -qa | awk 'END{print "安装的软件包总数是"NR"个"}'
    sleep 3
done
```

### 系统初始化

> 编写一个脚本可以匹配不同系统的服务器(包含7版本与8版本的系统)实现以下需求：
>
> 1，所有服务器永久关闭防火墙服务和SELinux
>
> 2，关闭7版本系统的命令历史记录，修改8版本的命令历史记录最多保存2000条并加上时间戳
>
> 3，关闭8版本系统的交换分区
>
> 4，定义root远程登录系统后的ssh保持时间为300秒
>
> 5，设置时间同步，ntp服务器地址是192.168.88.240

```
#!/bin/bash
#脚本执行完后，用ssh远程登录测试
#可以先手工备份/etc/fstab和/etc/profile
#1)判断当前账户身份，并关闭防火墙与selinux
[ $UID -ne 0 ] && echo "请使用管理员操作" && exit
systemctl stop firewalld                 
systemctl disable firewalld
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
#2）根据不同版本的系统执行各自的任务
egrep -q "\s+8\.[0-9]" /etc/redhat-release                    #判断系统版本
if [ $? -ne 0 ];then
    sed -ri 's/HISTSIZE=[0-9]+/HISTSIZE=0/' /etc/profile        #关闭历史命令
else
    sed -ri 's/HISTSIZE=[0-9]+/HISTSIZE=2000/' /etc/profile        #历史命令2000条
    sed -i '/^export /i HISTTIMEFORMAT="%F %T "' /etc/profile    #历史命令时间戳
    sed -i '/^export /s/$/ HISTTIMEFORMAT/' /etc/profile        #找到export开头那行，在最后加 HISTTIMEFORMAT，可以在子进程中也显示时间戳
    swap=$(swapon | awk 'NR!=1{print $1}')                #找到交换分区
    for i in $swap
    do
        swapoff $i                                        #循环关闭交换分区
    done
    sed -i '/swap/s/^/#/' /etc/fstab                    #关闭交换分区自动挂载
fi
#3)最后所有机器设置ssh超时时间与时间同步
echo "export TMOUT=300"  >> ~/.bash_profile                #定义ssh超时退出时间
yum -y install chrony
systemctl enable chronyd
sed  -ri  '/^(pool|server).*iburst/s/^/#/'   /etc/chrony.conf
sed  -i  '1i server 192.168.88.240 iburst'   /etc/chrony.conf
systemctl restart chronyd
```

### 通过文档批量创建帐号并配置密码

```
tr -cd '_a-zA-Z0-9' < /dev/urandom | head -c 10    
#-c是取反 -d是删除，对_a-zA-Z0-9取反删除，剩下就只是_a-zA-Z0-9这个范围内的字符串，head -c 10 可以得到10位字符
```

```
#!/bin/bash
x=$(awk '/^[a-zA-Z0-9]/&&!/已创建/{print NR}' user.txt)
if [ -z "$x" ];then
    echo "没有新用户需要创建"
    column -t user.txt  # 格式化输出文档
    exit
fi
for i in $x
do
    pass=$(strings /dev/urandom | tr -cd '_a-zA-Z0-9' | head -c 10)
    sed -i "${i}s/$/\t$pass/"  user.txt
    read name pass << EOF  
    $(sed -n "${i}p" user.txt)
EOF
    useradd $name
    echo $pass | passwd --stdin $name
    sed -i "${i}s/$/\t已创建/"  user.txt
done
column -t user.txt
```

### 可视化安装软件脚本

```
#vim menu                #创建菜单函数文件
x=1                    #高亮行号，默认为1即可
y=0                    #第几行
menu (){                #循环显示菜单的函数
    clear
    for i in 1,安装ftp服务 2,开关ftp服务 3,退出
    do
        echo "----------------"
        let y++
        [ $x -eq $y ] && echo -e "\033[43;93m$i\033[0m" && continue
    done
    y=0
    echo "----------------"
}
```

```
ftp安装函数
ftp_install(){
    if rpm -q vsftpd &> /dev/null ;then        #ftp如果已经安装则条件算成功
        msg="vsftpd已安装"                    #定义信息变量msg
    else
        yum -y install vsftpd &> /dev/null
        [ $? -eq 0 ] && msg="vsftpd安装成功" || msg="vsftpd安装失败"
    fi
}
```

```
#vim ser_manager            #创建服务管理函数文件
ser_manager(){
    if ! rpm -q vsftpd &>/dev/null ;then    #ftp如果未安装则条件算成功,！是取反
        msg="未安装vsftpd软件包"
        return                                #结束函数任务，跳出函数
    fi
    case $1 in
    start)                                #调用本函数后追加start就是把服务开启
        systemctl start vsftpd
        msg="ftp服务已经启动"
        ser_manager=start;;
    stop)                                    #调用本函数后追加stop就是把服务关闭
        systemctl stop vsftpd
        msg="ftp服务已经关闭"
        ser_manager=stop;;
    esac
}
```

```
#!/bin/bash
. menu            #读入菜单函数
. ftp_install        #读入安装函数
. ser_manager        #读入服务管理函数
while :
do
    menu
    echo "$msg"
    read -n 3 c    #-n 3是输入足够3个字符就自动进行下一步，c是存储字符的变量
    if [ "$c" == $'\033[A' ];then        #如果按了 "上" 键
        [ $x -eq 1 ] && continue        #根据变量x定义高亮行，在第1行就没变化
        let x--                        #如果不在第1行，就把x-1
    elif [ "$c" == $'\033[B' ];then    #如果按了 "下" 键
        [ $x -eq 3 ] && continue        #如果在第3行，没变化
        let x++                        #如果不在第3行，就把x+1
    elif [ -z $c ] && [ $x -eq 1 ];then        #如果在第1行回车就执行下列任务
        msg="ftp服务安装中。。。"
        echo "$msg"
        ftp_install                        #执行ftp_install函数的任务
    elif [ -z $c ] && [ $x -eq 2 ];then        #如果在第2行回车就执行下列任务
        [ "$ser_manager" != "start" ] && ser_manager start || ser_manager stop
    elif [ -z $c ] && [ $x -eq 3 ];then        #如果在第3行回车就执行下列任务
        exit
    fi
done
```

### 编写每天备份文件脚本

```
# 根据需求打包文件夹以外的内容，打包除.tmp结尾的所有内容
]#touch a b
]#touch c.tmp
]#ls
a b b.tmp
]#tar -zcf test.tar.gz --exclude=*.tmp .
```

```
#!/bin/bash
sou_path=/var/www/html
tar_path=/opt/backup_date
# date=$(date +%Y-%m-%d) # 年月日
date=$(date +%T) # 时分秒
ex_file=*.tmp
tar -zcf ${tar_path}/web_file_${date}.tar.gz --exclude=$ex_file ${sou_path} &> /dev/null
file_total=$(ls ${tar_path}|wc -l)
echo "${date}的文件打包放入${tar_path}当前文件总数是${file_total}个"
if [ $file_total -ge 5 ];then
    lftp 192.168.88.2 << EOF
    cd pub
    mput -q ${tar_path}/*
    quit
EOF
    if [ $? -ne 0 ];then
        echo "上传失败！"
    else
        echo "上传成功！"
        rm -rf ${tar_path}/web_file*
    fi
fi  
```

## 输入重定向

#### `<`

后面需要根文件名，不再从键盘读取数据，而是从文件中读取数据

```
]#wc -l < /etc/passwd
53
```

#### `<<`

代表你要的内容在这里，某指令导入字符串时使用，则无需文件

```
]# mail -s test root << EOF
hello
test mail~
EOF
```

#### read

使用reda指令配合输入重定向可同时定义多个变量

```
vim abc.txt
a b
c d
while read a b;do echo $a;done < abc.txt
a
c
while read a b;do echo $a $b;done < abc.txt
a b
c d
```

## 脚本内部调用外部文件

```
. 文件名
#!/bin/bash
. menu
. ftp_install
. ser_manager
while :
do

使用.命令来引入名为menu的脚本文件。这个语法会导致脚本中的内容被加载并执行，类似于将menu文件中的内容嵌入到当前脚本中。
```



# 快捷键

# 问题

## 1 在.bash_profile文件中定义的TMOUT变量有什么作用？

> 限制ssh连接时长，单位是秒

## 2 简述`<`、`<<`各自的作用。

> `<` 输入重定向，后面必须跟文件名，让程序不再从键盘读取数据，而是从文件中读取数据
>
> `<<`通过某指令导入字符串时使用，无需文件

## 3 如何使用read同时定义多个变量？

> 仅定义一次，可以使用read a b 命令，然后输入两个字符串中间加空格
>
> 如果定义多次，可以结合while循环导入文件
>
> ```
> whlie read a b
> do
> 	echo $a $b
> done < abc.txt
> ```

## 4 /dev/urandom文件有什么用处？可以在什么场合使用？

> 生成随机字符
>
> 生成密码，或需要一串字符串

## 5 使用lftp时，如何上传文件？

> 使用mput命令

## 6 打tar包时如果要排除所有txt格式的文件如何实现？

> tar -zcf test.tar.gz --exclude=*.txt /test

# 补充



# 今日总结



# 昨日复习
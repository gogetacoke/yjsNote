- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
- [快捷键](#快捷键)
- [问题](#问题)
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
#!/bin/bash
. menu
. ftp_install
. ser_manager
while :
do
        menu
        echo "$msg"
        read -n 3 c
        if [ "$c" == $'\033[A' ];then
                [ $x -eq 1 ] && continue
                let x--
        elif [ "$c" == $'\033[B' ];then
                [ $x -eq 3 ] && continue
                let x++
        elif [ $x -eq 1 ] && [ -z "$c" ];then
                msg="ftp正在安装中..."
                echo "$msg"
                ftp_install
        elif [ $x -eq 2 ] && [ -z "$c" ];then
                [ "$ser_manager" != "start" ] && ser_manager start || ser_manager stop
        elif [ $x -eq 3 ] && [ -z "$c" ];then
                exit
        fi
done
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
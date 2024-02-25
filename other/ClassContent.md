scp /etc/passwd  root@192.168.66.11:/x

> scp /目录/迁移文件 目的ip:存放的路径（不指定则使用和被发送文件所在的位置）

ssh 192.168.66.66

> 远程连接

ss -ntulp | grep 80

> 查询端口是否占用

bash-completion

> 代码拓展包，方便tab键联想代码

wingftp

> windows上安装ftp

--nodeps 

> rpm包跳过当前包的依赖

安装nginx所需要的依赖

> pcre-devel
>
> openssl-devel

centos7输入法

```
yum -y install ibus ibus-libpinyin
# 配置环境变量 root用户；.bashrc
export GTK_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
export QT_IM_MODULE=ibus

# reboot
```

linux不支持ntfs

```
yum -y install ntfs-3g
```

查询系统最大打开文件次数

```
ulimt -n # 查看
ulimit -n  10000 # 临时修改
vim /etc/security/limits.conf 
*               soft    nofile            100000 # 限制值，每次使用
*               hard    nofile             100000 # 软件限制值，最大
```

查询文件所在

```
whereis  文件名
```

yum也可以装RPM包

```
yum -y install phpredis-5.1.0-1.x86_64.rpm
```

开启Linux内核

```
modprobe ip_gre
lsmod | grep gre #查询开启情况
```

服务与语言

```
静态页面：httpd、nginx(性能最好)、tomcat
动态：
php：nginx+php-fpm
python：nginx+uwsgi
java：tomcat
```

arp -n

> 查看IP与mac对照表

自定义退出码

> exit 10
>
> 退出码最多为二进制的8位，多出的将被丢弃，基本范围0～255
>
> 例如：
>
> 十进制：520
>
> 二进制：1000001000
>
> 520超出了8位将保留0001000，丢弃10，所以最终为8

设置时区

> timedatectl set-timezone Asia/Shanghai

开机暂时跳过自动挂载

```sh
/dev/sdb1  /mnt/data  ext4  defaults,noauto  0  0
# 使用noauto选项来避免系统在启动时尝试挂载该设备
```

who -b

> 系统上次登陆服务器时间

yum -y install dos2unix

> 用于在linux上查看编成源码，避免出现^M

```sh
dos2unix xxx.py
```

linux支持中文

```sh
]#yum -y install langpacks-zh_CN.noarch
]#vim /etc/locale.conf
LANG="zh_CN.UTF-8"
]#reboot
```

maven打包时的注意项

```perl
maven打包时会去执行带有test文件夹的测试代码，如果代码不通过，将打包不通过，加上如下参数可以跳过test
-Dmaven.test.skip=true
```

jps

```perl
# java-xxx-openjdk-devel 中提供的命令
[root@Backend ~]# jps # 用于查询当前启动的相关java进程PID
20098 jar
20238 Jps
```

xargs

```perl
# 命令同find中的 exec类似，将前面的命名运行结果交给{}
jps | grep jar | awk '{print $1}' | xargs kill  # 将|前的结果交给 xgrgs执行kill
```

2>&1

> 1> 标准正确输出
>
> 2> 标准错误输出
>
> 2>&1  将标准错误输出转换成标准输出

gz格式解压

```sh
gzip -d file.gz # 这样解压压缩包将被覆盖
gzip -dc file.gz > newfile # 将解压内容输出并重定向到名为newfile的文件中
```

man -w

> 用于查询man帮助手册所处路径

```sh
man -w ls # 查找ls命令的man帮助手册所在地方
/usr/share/man/man1/ls.1.gz
```

tee命令

> `tee` 会读取来自管道、重定向或者其他命令的输出，并将其内容既输出到屏幕，又写入指定的文件中 
>
> - `-a` 或 `--append`：追加模式，将内容追加到文件末尾，而不是覆盖文件内容。
> - `-i` 或 `--ignore-interrupts`：忽略中断信号（Ctrl+C）的影响，继续执行写入操作。

```sh
]# echo "Hello, World!" | tee output.txt # 将echo内容保存到output.txt文件中
# 同时写入到两个文件中
]#ls -l | tee file1.txt file2.txt
```

sed中配合&符号使用

> 通常用于在配置文件中注释匹配到的行

```sh
echo "Hello, World!" | sed 's/World/Friend/'

"
这个命令会将"World"替换为"Friend"，输出结果是："Hello, Friend!"
但是，如果你想要保留"World"并添加一些文本，可以这样用`&`
"
echo "Hello, World!" | sed 's/World/& is my friend/'
"
此时输出结果是："Hello, World is my friend!"，这里的`&`就代表了之前匹配到的"World"。
"
```

$@

> 代表所有参数的和会保留参数的意义

shell参数中默认值

```sh
# 未设置值为默认值123
[root@docker-0002 ~]#echo ${aa-123}
123
# 设置aa为456
[root@docker-0002 ~]#aa=456
[root@docker-0002 ~]#echo ${aa-123}
456
[root@docker-0002 ~]#unset aa
[root@docker-0002 ~]#echo ${aa-123}
123
```


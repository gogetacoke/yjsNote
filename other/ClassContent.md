e scp /etc/passwd  root@192.168.66.11:/x

> scp /目录/迁移文件 目的ip:存放的路径（不指定则使用和被发送文件所在的位置）

vm clone 虚拟机名称

vm setip 虚拟机 192.168.66.66

ssh 192.168.66.66

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
yum -y install ibus ibus-pinyin
# 配置环境变量 root用户；.bashrc
export GTK_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
export QT_IM_MODULE=ibus
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


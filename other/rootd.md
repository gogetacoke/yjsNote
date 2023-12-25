# `～`

~/.vimrc

> autocmd filetype sh set ai ts=4
>
> 编写shell脚本时会自动执行上方指令，设置自动缩进4
>

~/.ssh

> 显示ssh远程的机器、公私钥

# `mnt`

/mnt

>用于用户设置的挂载的访问点

# `dev`

/dev

 >存储设备文件数据

/dev/null

> 简称黑洞，存放在里面的消息，将消失，一般使用脚本输出信息重定向到黑洞

# `var`

var

> 存放经常变化的数据

/var/log/dmesg

> 记录系统启动过程种的各种消息

/var/log/cron

>  记录cron计划任务相关的消息

/var/log/message

> 记录内核消息、各种服务公共消息

/var/log/maillog

> 记录邮件收发相关的消息

/var/log/secure

> 记录与访问限制相关的安全消息

# `usr`

/usr/local

> 源码包编译存放地
>
> 不指定源码包编译位置，默认将编译存放在local下哦啊

# `opt`

/opt

> 用户自定义下载数据存储

# `root`

/root

> 管理员的家目录

/root/.ssh/know_hosts

> 记录当前主机ssh过那些机器

# `home`

/home

>  普通用户的家目录

# `tmp`

> 存放临时数据的地发

# `etc`

/etc/skel

> 每次新建家目录的模板目录

/etc/fstab

> 设置磁盘开机自动挂载

/etc/httpd/conf/httpd.conf

> httpd主配置文件

/etc/exports

> nfs设置共享的配置文件

/etc/shells

> 记录当期机器上有哪些解释器可以使用

/etc/profile

> 存储系统重要的变量，修改时，先备份

/etc/resolv.conf

> 指定dns的配置文件

/etc/services

> 服务与端口的对应关系 

/etc/motd

> 用于定义ssh链接后的提示信息

/etc/rc.d/rc/local

> 用于设置开机后自动运行的命令
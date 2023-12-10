scp /etc/passwd  192.168.66.11

> scp /目录/迁移文件 目的ip

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


> 修改配置文件后，需要source 刷新配置

# 历史命令

## 修改历史命令数

```
/etc/profile
HISTSIZE=2000
```

## 添加时间戳

```
/etc/profile
HISTTIMEFORMAT="%F %T" # 添加一条
```

# ssh

## 修改针对root用户ssh会话时长

```
~/.bash_profile
export TMOUT=100  # 添加一条，以秒为单位
```

# ntp

## 添加本地时间服务器

```
/etc/chrony.conf
server 192.168.88.240  iburst # 修改poor
```

# FTP

## 匿名访问

```
/etc/vsftpd/vsftpd.conf
anonymous_enable=NO
anonymous_enable=YES
```

## 修改上传权限

```
/etc/vsftpd/vsftpd.conf
#anon_upload_enable=YES
anon_upload_enable=YES
```


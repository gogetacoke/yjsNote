# 查看系统版本

```
cat /etc/redhat-release
Rocky Linux release 8.6 (Green Obsidian)

uname -a
inux localhost.localdomain 4.18.0-372.9.1.el8.x86_64 #1 SMP Tue May 10 14:48:47 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux

-r
4.18.0-372.9.1.el8.x86_64
```

# 查询对应进程

```
ps -ef|grep ssh  
```

# 查询端口

```
ss -ntulp|grep 80

netstat -ntlp | grep 80
```

# 查看文件详细信息

```
ls -l /

stat /
```

# 提取文件名

```
basename /etc/passwd
passwd

basename -a /etc/passwd /etc/shadow
passwd
shadow
```

# 提取完整目录名

```
dirname /etc/sysconfig/network-scripts/ifcfg-ens33 

/etc/sysconfig/network-scripts

```

# 查看文件类型

```
file abc01/
abc01/: directory
```

# 定位二进制程序、源代码文件

```
whereis ls
ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz /usr/share/man/man1p/ls.1p.gz
显示程序文件和man帮助所在地
```

# 更新linux数据库

```
有些时候使用find查询不到刚删除的文件，因为linux数据库没及时更新
updatedb
```

# 快速查找文件或目录

```
# 比find要快一点，find是去磁盘找，这个命令是从数据库查找
locate file.txt
locate /opt/a #查找opt目录下所有a开头的文件或目录
-i  忽略大小写
```

# grep

```
grep -c root /etc/passwd  # 统计root出现的次数
grep -n root /etc/passwd #查询包含root的出现在哪一行
1:root:x:0:0:root:/root:/bin/bash
10:operator:x:11:0:operator:/root:/sbin/nolog
```

# cat

```
cat -n  # 将内容以行号输出
cat -b # 有内容才进行编号，没有内容，不编号
```

# 内容翻页

```
less /etc/passwd

more /etc/passwd
more -c -5 /etc/passwd # 每次显示五行内容
```

# tac

```
tac /etc/passwd  # 反向显示内容，与cat相反
```

# wc 

```
wc test.txt # 统计总共有多少行，多少个字，以及字节数
wc -m # 统计文本字符数
```

# 创建一个指定大小的文件

```
dd if=源设备 of=目标设备 bs=块大小 count=次数
dd if=/dev/zero of=/opt/test bs=1M count=2048 
```


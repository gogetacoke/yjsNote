- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
- [快捷键](#快捷键)
- [问题](#问题)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)


# 学习目标

RPM软件包管理

Yun软件包仓库

命令的补充

# 课堂笔记（命令）

rpm

> rpm -q firefox 查询系统是否已安装firefox
>
> rpm -qa 查询系统已经安装的软件包
>
> rpm -qa | wc -l 统计系统已安装包的数量
>
> rpm -ql firefox 查询软件安装了那些内容
>
> rpm -qi firefox 查询软件安装信息
>
> rpm -qf /etc/passwd 查询对应程序文件由那个软件包产生
>
> rpm -qpl xxx 查询未安装程序安装后所产生的内容（前提须有该软件包在系统上）
>
> rpm -qpi xxx 查询未安装软件的软件信息（前提须有该软件包在系统上）
>
> rpm --import /etc/pki/rpm-gpg/RPMxx 导入红帽签名信息（不导入，则查询未安装包信息时会有个警告）
>
> **软件安装**
>
> rpm -ivh 软件包名
>
> yum -y install xxx.rpm
>
> **卸载软件包**
>
> rpm -e 软件包名

yum

> **仓库构建**
>
> 将 /etc/yum.repos.d/下的所有内容新创建一个文件夹将内容mv进去，前提系统已经挂载镜像
>
> 编写仓库文件配置内容
>
> vim /etc/yum.repos.d/hh.repo
>
> 编写内容：
>
> ```linux
> [xixi] # 仓库名称标识
> name=hah xixi # 仓库描述信息
> baseurl=file:///mnt/AppStream # 仓库位置，这是本地仓库
> enabled=1 # 是否启用本地仓库 0不启用 1启用
> gpgcheck=0 # 是否检测软件红帽信息 0不启用 1启用
> # gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-rockyofficial #启用检测红帽信息才有这一条
> ```
>
> 多个仓库则在后追加上诉相同内容，其仓库位置改为新仓库地址
>
> yum repoinfo 识别本地仓库是否搭建成功
>
> yum repolist 列出仓库数量
>
> yum makecache 刷新仓库缓存
>
> **安装软件**
>
> yum -y install httpd 
>
> 检测软件安装是否成功
>
> rpm -q httpd
>
> 
>
> -y：默认同意安装
>
> **卸载软件**
>
> yum remove httpd（谨慎使用-y）
>
> **仓库查询**
>
> yum list `ftp `查询仓库是否有ftp软件
>
> yum search `ftp` 包含ftp就匹配（相比rpm -q 更加敏感）
>
> yum provides `guestmount `查询仓库中那个软件可以在任意路径下产生该程序（相比rpm -qf 更加敏感）
>
> **覆盖安装**
>
> yum -y reinstall xx  修复安装（案例：不小心删除文件后的操作）
>
> **清空缓存**
>
> yum clean all 清空yum的缓存

date

> 查看调整系统日期时间
>
> date 查看系统时间
>
> date -s ‘2023-11-6 16:39’ 修改系统时间
>
> date +%Y     #显示年
>
> date +%m    #显示月
>
> date +%d     #显示日期
>
> date +%H    #显示时
>
> date +%M    #显示分
>
> date +%S     #显示秒
>
> date +%F    #显示年**-**月-日
>
> date +%T    #显示时:分:秒

history

> histoty 显示历史命令列表
>
> history -c 清空历史命令（只是清空开机后所有命令）
>
> history -w清空历史命令为空（清空全部历史命令）
>
> ！144 指定历史命令第144条
>
> ！cat  指定最近一条以cat开头的历史命令

du -sh 文件名或文件所在路径

> 统计目录文件大小

ln -s

> **软链接**
>
> 创建快捷方式（链接文件、目录）
>
> 格式：ln -s /源文件、目录 /快捷方式名称
>
> ln -s /etc/hosts /1.txt 快捷文件
>
> ln -s /opt /xiix 快捷目录（切勿在xiix后面加上/否则将删除opt目录里面所有内容）
>
> 缺点：源文件删除后，快捷方式不能使用
>
> 
>
> **硬链接**
>
> 格式：ln /源文件 /快捷方式名称
>
> [注]：硬链接不能链接目录
>
> ln /etc/passwd /opt/pw.txt
>
> 优点：源文件删除后，快捷方式还在

ls -i

> 查询内存地址
>
> ls -i /opt

# 课堂笔记（文本）

1. windos软件安装两大特点

   可以制定安装路径，安装到同一个目录

2. 红颜色代表软件包

3. 红帽签名所在地

   /etc/pki/rpm-gpg/

4. 软件仓库配置文件所在地

   /etc/yum.repos.d/xxx.repo

# 快捷键



# 问题

1. linux安装的程序一般放在那个位置

   普通执行程序：/usr/bin、/bin

   服务器程序、管理工具：/user/sbin、/sbin

# 今日总结



# 昨日复习

1. 将root/backup.tar.gz压缩包，释放到/opt目录下

   tar -xf /root/backup.tar.gz -C /opt

2. 利用find查找/etc目录下，所有以‘.conf’结尾的数据

   find /etc -name ‘*.conf’

3. 利用find查找/etc目录下，chichi目录即可

   find /etc -type d

4. 如何将/boot目录大于10M的数据，快速拷贝到/opt目录下

   find /boot -size +10M -exec cp {} /opt \;

5. 统计/var目录三个月之前数据的个数，如何操作?

   find /var -mtime +90 `| `wc -l

6. 显示/etc/login.defs文件有效信息，请写出该命令

   grep -v ^# /etc/login.defs `|`grep -v ^$

7. vim编辑器，如何实现，复制光标所在的一行？

   yy

8. vim编辑器，将全文的man替换为MAN如何操作？

   %s/man/MAN/g

9. vim编辑器中如何开启行号功能？

   set nu

10. 将"hello world"快速写入到，/opt/cbd.txt文件的最后一行，如何操作？

    echo hello word >> /opt/cdb.txt

11. 显示ifconfig命令输出的前两行，如何操作？

    ifconfig `|` head -2 
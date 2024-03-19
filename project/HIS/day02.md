- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [GitLab服务器搭建](#gitlab服务器搭建)
    - [GitLab概述](#gitlab概述)
    - [镜像源](#镜像源)
    - [安装](#安装)
    - [初始化](#初始化)
    - [访问](#访问)
  - [代码托管](#代码托管)
    - [推送远程仓库](#推送远程仓库)
    - [HTTP免密登陆](#http免密登陆)
    - [HIS项目代码托管](#his项目代码托管)
  - [CI/CD](#cicd)
  - [JenKins部署](#jenkins部署)
    - [Jenkins初始化](#jenkins初始化)
    - [离线部署插件](#离线部署插件)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)


# 学习目标

Git服务器

GitLab搭建使用

CI/CD概述

Jenkins部署

Jenkins工程构建

# 课堂笔记（命令）

# 课堂笔记（文本）

## GitLab服务器搭建

### GitLab概述

![](../../pic/pro/d2-4.png)

### 镜像源

> 下载gitlab[点击清华镜像源下载](https://mirrors.tuna.tsinghua.edu.cn/)

### 安装

>  gitlab-ce-12.4.6-ce.0.el7.x86_64.rpm 

```perl
[root@gitlab ~]# yum clean all;yum repoinfo
# 缺少依赖无法安装，当前为rocky8系统，当前下载的包兼容el7版本，会出现包无法自动解决依赖，根据提示手动解决依赖
[root@gitlab ~]# yum -y install ./gitlab-ce-12.4.6-ce.0.el7.x86_64.rpm  
Last metadata expiration check: 0:00:01 ago on Wed 24 Jan 2024 09:46:59 AM CST.
Error: 
 Problem: conflicting requests
  - nothing provides policycoreutils-python needed by gitlab-ce-12.4.6-ce.0.el7.x86_64
(try to add '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages)
# 解决依赖
[root@gitlab ~]#yum list | grep -i polic | grep python
[root@gitlab ~]#yum -y install policycoreutils-python-utils.noarch
[root@gitlab ~]#rpm -ivh --nodeps --force gitlab-ce-12.4.6-ce.0.el7.x86_64.rpm 
```

### 初始化

```perl
[root@gitlab ~]#gitlab-ctl reconfigure
#重启GitLab相关服务
[root@GitLab ~]# gitlab-ctl restart
ok: run: alertmanager: (pid 1975) 0s        //报警服务
ok: run: gitaly: (pid 1986) 1s              //Git后台服务
ok: run: gitlab-exporter: (pid 2014) 0s     //Prometheus数据采集器
ok: run: gitlab-workhorse: (pid 2020) 1s    //反向代理服务器
ok: run: grafana: (pid 2117) 0s             //数据可视化服务
ok: run: logrotate: (pid 2129) 0s           //日志文件管理服务
ok: run: nginx: (pid 2135) 1s               //静态WEB服务
ok: run: node-exporter: (pid 2142) 0s       //Prometheus数据采集器
ok: run: postgres-exporter: (pid 2148) 1s   //Prometheus数据采集器
ok: run: postgresql: (pid 2159) 0s          //数据库服务
ok: run: prometheus: (pid 2168) 0s          //Prometheus监控服务
ok: run: redis: (pid 2178) 1s               //缓存数据库服务
ok: run: redis-exporter: (pid 2183) 0s      //Prometheus数据采集器
ok: run: sidekiq: (pid 2192) 0s             //异步执行队列服务
ok: run: unicorn: (pid 2203) 0s             //Rails托管WEB服务 

#GitLab主机确认80端口被监听
[root@GitLab ~]# ss -anptul | grep :80
tcp   LISTEN 0      511          0.0.0.0:80        0.0.0.0:*    users:(("nginx",pid=11166,fd=7),("nginx",pid=11165,fd=7),("nginx",pid=11164,fd=7))  
[root@GitLab ~]# 
```

### 访问

> 浏览器访问部署机器的IP地址即可
>
> http://192.168.88.20
>
> 根据页面进行自行创建用户、组、仓库

## 代码托管

### 推送远程仓库

```perl
#参考提示命令"推送已有Git仓库部分"
[root@Programer ~]# cd myproject/
[root@Programer myproject]# git remote -v                   #查看远程仓库关联信息
[root@Programer myproject]# git remote add origin           http://192.168.88.20/devops/myproject.git    关联远程项目
[root@Programer myproject]# git remote -v                   #查看远程想看关联信息
origin  http://192.168.88.20/devops/myproject.git (fetch)
origin  http://192.168.88.20/devops/myproject.git (push)
[root@Programer myproject]#git add . # 添加所有文件到暂缓区
[root@Programer myproject]#git commit -m "add" # 提交文件到本地仓库
[root@Programer myproject]# git push -u origin --all        #推送本地仓库所有分支;r若有标签还需单独推送 git push -u origin --tags；首次推送需要指定远程仓库别名origin
Username for 'http://192.168.88.20': mark       #用户名
Password for 'http://mark@192.168.88.20':       #密码
枚举对象中: 29, 完成.
对象计数中: 100% (29/29), 完成.
使用 2 个线程进行压缩
压缩对象中: 100% (19/19), 完成.
写入对象中: 100% (29/29), 2.21 KiB | 2.21 MiB/s, 完成.
总共 29（差异 5），复用 0（差异 0），包复用 0
 * [new branch]      feature -> feature
 * [new branch]      hotfix -> hotfix
 * [new branch]      master -> master
分支 'feature' 设置为跟踪来自 'origin' 的远程分支 'feature'。
分支 'hotfix' 设置为跟踪来自 'origin' 的远程分支 'hotfix'。
分支 'master' 设置为跟踪来自 'origin' 的远程分支 'master'。

[root@Programer myproject]# git push -u origin --tags       #推送本地仓库所有标
Username for 'http://192.168.88.20': mark       #用户名
Password for 'http://mark@192.168.88.20':       #密码
总共 0（差异 0），复用 0（差异 0），包复用 0
To http://192.168.88.20/devops/myproject.git
 * [new tag]         v1 -> v1
```

> git remote add origin  http://192.168.88.20/devops/myproject.git    
>
> 关联远程仓库地址写错可通过如下进行删除，重新上传
>
> git remote remove origin  

### HTTP免密登陆

```sh
#每次推送都需要认证，可以设置免密推送
    #store：永久存储
    #cache：默认缓存15分钟，期间无需认证，可通过cache --timeout=3600设置超时时间
    #git remote add origin http://name:pass@192.168.88.20/devops/myproject.git
[root@Programer myproject]# git config --global credential.helper store #持久保存
[root@Programer myproject]# git push   #推送代码（本次需要认证）
Username for 'http://192.168.88.20': mark   #用户名
Password for 'http://mark@192.168.88.20':   #密码
Everything up-to-date
[root@Programer myproject]# git push     #再次推送测试免密
Everything up-to-date
[root@Programer myproject]# git config --list
user.name=mark
user.email=mark@tedu.cn
init.defaultbranch=master
credential.helper=store
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
remote.origin.url=http://192.168.88.20/devops/myproject.git
remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
branch.feature.remote=origin
branch.feature.merge=refs/heads/feature
branch.hotfix.remote=origin
branch.hotfix.merge=refs/heads/hotfix
branch.master.remote=origin
branch.master.merge=refs/heads/master
[root@Programer myproject]# cat ~/.git-credentials  # 存放账密文件
http://mark:1234.com@192.168.88.20
```

### HIS项目代码托管

> [点击下载代码](https://gogetacoke.lanpw.com/ibMDk1m5bwli)

```perl
# 将代码拷贝到20机器安装uzip解压
[root@Programer ~]# unzip HIS.zip                           #解压项目代码压缩包
[root@Programer ~]# ls HIS
HIS-BACKEND                 #HIS项目后端代码，SpringBoot框架编写
HIS-CONFIG                  #HIS项目后端程序运行配置文件
HIS-FONTEND                 #HIS项目前端代码（已编译）
HIS-FONTEND-DEPENDENCY      #HIS项目前端代码编译依赖库（npm库）
HIS-FONTEND-SOURCECODE      #HIS项目前端代码（未编译，VUE框架编写）
HIS-SQL                     #HIS项目数据库SQL文件
```

> + 在GitLab服务端网页上创建Project名为HIS-BACKEND、HIS-FONTEND

```perl
[root@Programer ~]# cd HIS/HIS-FONTEND
[root@Programer HIS-FONTEND]# git init                          #初始化仓库
已初始化空的 Git 仓库于 /root/HIS/HIS-FONTEND/.git/
[root@Programer HIS-FONTEND]# git remote add origin             http://192.168.88.20/devops/HIS-FONTEND.git                     #关联本地与远程
[root@Programer HIS-FONTEND]# git remote -v
origin  http://192.168.88.20/devops/HIS-FONTEND.git (fetch)
origin  http://192.168.88.20/devops/HIS-FONTEND.git (push)
[root@Programer HIS-FONTEND]# git add ./                        #添加文件到暂存区
[root@Programer HIS-FONTEND]# git commit -m "Init Fontend Code" #提交到本地版本库
[root@Programer HIS-FONTEND]#git tags v1
[root@Programer HIS-FONTEND]# git push -u origin --all          #推送所有分支
[root@Programer HIS-FONTEND]# git push -u origin --tags # 推送所有标签   
```

## CI/CD

![](../../pic/pro/d2-5.png)

## JenKins部署

```perl
#Jenkins主机安装OpenJDK环境
[root@Jenkins ~]# yum clean all; yum repolist -v
...
Total packages: 8,265
[root@Jenkins ~]# yum -y install java-11-openjdk-devel.x86_64   #安装OpenJDK11
[root@Jenkins ~]# ln -s /usr/lib/jvm/java-11-openjdk-11.0.15.0.9-2.el8_5.x86_64/ /usr/lib/jvm/jdk                                                #创建JDK环境软链接
[root@Jenkins ~]# vim /etc/bashrc 
[root@Jenkins ~]# tail -2 /etc/bashrc                           #声明JAVA_HOME环境变量
export JAVA_HOME="/usr/lib/jvm/jdk/"
export PATH=${JAVA_HOME}/bin/:$PATH
[root@Jenkins ~]# source /etc/bashrc                            #刷新当前bash环境
[root@Jenkins ~]# echo ${JAVA_HOME}                             #查看JAVA_HOME变量
/usr/lib/jvm/jdk/
[root@Jenkins ~]# which java
/usr/lib/jvm/jdk/bin/java
[root@Jenkins ~]# java -version
openjdk version "11.0.15" 2022-04-19 LTS
OpenJDK Runtime Environment 18.9 (build 11.0.15+9-LTS)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.15+9-LTS, mixed mode, sharing)
[root@Jenkins ~]# 

#安装工具相关软件
[root@Jenkins ~]# yum -y install git postfix    #Git用于拉取代码、postfix用于发邮件

#安装Jenkins(提前下载jenkis)
[root@Jenkins ~]# ls jenkins-2.361.4-1.1.noarch.rpm 
jenkins-2.361.4-1.1.noarch.rpm
[root@Jenkins ~]# yum -y localinstall ./jenkins-2.361.4-1.1.noarch.rpm  #安装Jenkins

#启动Jenkins服务
[root@Jenkins ~]# systemctl enable jenkins.service              #设置Jenkins开机自启动
[root@Jenkins ~]# systemctl start jenkins.service               #启动Jenkins服务器
[root@Jenkins ~]# ss -antpul | grep java                        #确认8080端口被监听
tcp   LISTEN 0      50     *:8080     *:*    users:(("java",pid=13602,fd=8))
```

### Jenkins初始化

```perl
#获取Jenkins初始密码
[root@Jenkins ~]# cat /var/lib/jenkins/secrets/initialAdminPassword 
1aaf609acf3843a8aaf17e8b1f22e8ea
[root@Jenkins ~]# 

#浏览器访问： http://192.168.88.30:8080/
```

### 离线部署插件

```perl
#离线部署Jenkins插件
[root@Jenkins ~]# ls /var/lib/jenkins/plugins/              #空目录，没有任何插件
[root@Jenkins ~]# ls jenkins_plugins.tar.gz 
jenkins_plugins.tar.gz
[root@Jenkins ~]# tar -xPpf jenkins_plugins.tar.gz -C /     #解压Jenkins插件，保留文件归属
[root@Jenkins ~]# ls -l /var/lib/jenkins/plugins/               #确认该目录下文件归属

[root@Jenkins ~]# systemctl restart jenkins.service         #重启Jenkins加载插件
[root@Jenkins ~]# ss -antpul | grep java
tcp   LISTEN 0      50       *:8080    *:*    users:(("java",pid=13917,fd=8))
```

> `tar -xPpf` 命令拓展：
>
> - `-x`: 表示要进行解压操作。
> - `-P`: 表示保持原始文件路径，即在解压缩时不会改变文件的存储路径。
> - `-p`: 保留文件的权限和属性。
> - `-f`: 指定要解压缩的 tar 归档文件



# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习
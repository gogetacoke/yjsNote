- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)

# 学习目标

Dockerfile详解

容器镜像制作

私有仓库

# 课堂笔记（命令）

# 课堂笔记（文本）

## 镜像编排

### Dockerfile概述

> Dockerfile是一种更强大的镜像制作方式
>
> 编写类似脚本的Dockerfile文件，通过该文件制作镜像
>
> 制作镜像：docker build -t 镜像名称:标签 Dockerfile所在目录

### Dockerfile参数详解

| **指令**   | **说明**                                                 |
| ---------- | -------------------------------------------------------- |
| FROM       | 指定基础镜像（唯一）                                     |
| RUN        | 在容器内执行命令，可以写多条                             |
| ADD        | 把文件拷贝到容器内，如果文件是 tar.xx 格式，会自动解压   |
| COPY       | 把文件拷贝到容器内，如果文件是 tar.xx 格式，不会自动解压 |
| ENV        | 设置启动容器的环境变量                                   |
| WORKDIR    | 设置启动容器的默认工作目录（唯一）                       |
| CMD        | 容器默认的启动参数（唯一）                               |
| ENTRYPOINT | 容器默认的启动命令（唯一）                               |
| USER       | 启动容器使用的用户（唯一）                               |
| EXPOSE     | 使用镜像创建的容器默认监听使用的端口号/协议              |

#### 语法案例1

> 使用FROM与CMD创建简单镜像

```shell
# 编写Dockerfile文件
[root@docker-0002 ~]#mkdir myimg
[root@docker-0002 ~]#vim myimg/Dockerfile
FROM mylinux:latest
CMD ["/bin/ls","-l"]
# 制作镜像
[root@docker-0002 ~]#docker build -t img1:latest /root/myimg
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM mylinux:latest
 ---> 8744fedcf7c8
Step 2/2 : CMD ["/bin/ls","-l"]
 ---> Running in 2d090fbb5245
Removing intermediate container 2d090fbb5245
 ---> 76aa69bc14f4
Successfully built 76aa69bc14f4
Successfully tagged img1:latest
# 测试镜像
[root@docker-0002 ~]#docker run -it --rm img1:latest
total 48
lrwxrwxrwx   1 root root    7 Oct 11  2021 bin -> usr/bin
drwxr-xr-x   5 root root  360 Feb 19 01:18 dev
drwxr-xr-x   1 root root 4096 Feb 19 01:18 etc
drwxr-xr-x   2 root root 4096 Oct 11  2021 home
lrwxrwxrwx   1 root root    7 Oct 11  2021 lib -> usr/lib
lrwxrwxrwx   1 root root    9 Oct 11  2021 lib64 -> usr/lib64
drwx------   2 root root 4096 Nov 14  2021 lost+found
drwxr-xr-x   2 root root 4096 Oct 11  2021 media
drwxr-xr-x   2 root root 4096 Oct 11  2021 mnt
drwxr-xr-x   2 root root 4096 Oct 11  2021 opt
dr-xr-xr-x 102 root root    0 Feb 19 01:18 proc
dr-xr-x---   1 root root 4096 Feb 18 08:00 root
drwxr-xr-x  11 root root 4096 Nov 14  2021 run
lrwxrwxrwx   1 root root    8 Oct 11  2021 sbin -> usr/sbin
drwxr-xr-x   2 root root 4096 Oct 11  2021 srv
dr-xr-xr-x  13 root root    0 Feb 19 01:18 sys
drwxrwxrwt   1 root root 4096 Feb 18 07:59 tmp
drwxr-xr-x   1 root root 4096 Nov 14  2021 usr
drwxr-xr-x   1 root root 4096 Nov 14  2021 var
```

#### 语法案例2

> ENTRYPOINT与CMD的区别

```shell
# ENTRYPOINT 与 CMD 执行方式为 ${ENTRYPOINT} ${CMD}
[root@docker-0002 ~]# vim myimg/Dockerfile 
FROM mylinux:latest
ENTRYPOINT ["echo"]
CMD  ["/bin/ls", "-l"]
# 创建镜像
[root@docker-0002 ~]# docker build -t img2:latest myimg
......
Successfully tagged img2:latest

# 不指定在容器内的执行的命令，则CMD 做为参数传递，在容器内执行了 echo '/bin/ls -l'  
[root@docker-0002 ~]# docker run -it --rm img2:latest 
/bin/ls -l

# 指定在容器内执行的命令(id)，则CMD 被替换，相当于在容器内执行了 echo id
[root@docker-0002 ~]# docker run -it --rm img2:latest id
id

# 查询环境变量PATH
[root@docker-0002 ~]# docker run -it --rm img2:latest $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
# 查询echo的帮助信息
[root@docker-0002 ~]# docker run -it --rm img2:latest --help
Usage: echo [SHORT-OPTION]... [STRING]...
  or:  echo LONG-OPTION
"
ENTRYPOINT指定的参数被作为了默认参数，只有在创建Dockerfile文件时才能指定，在使用该镜像作为容器时CMD的参数作为额外参数传递给ENTRYPOINT；
以上方案例可知，固定参数为echo 后面根符合的参数均可
"
```

> ENTRYPOINT与CMD什么时候使用：
>
> + 启动镜像默认参数不想被更改或被覆盖就用ENTRYPOINT
> + 其余都使用CMD
>
> 总结：
>
> + ENTRYPOINT 主要用来为镜像设定一个固定不变的启动指令，而 CMD 则提供了对这个指令进行定制或添加默认参数的灵活性。

#### 语法案例3

> 使用COPY与ADD参数来拷贝文件/var/tmp与/tmp目录
>
> COPY：直接拷贝
>
> ADD：拷贝并解压
>
> [注]：文件格式为tar.xx格式才会自动·解压：拷贝的文件是拷贝的当前Dockerfile所在目录的文件

```shell
# 制作tar包
[root@docker-0002 ~]# tar -cf myimg/myfile.tar -C /etc/ hosts issue
[root@docker-0002 ~]# ls myimg/
Dockerfile  myfile.tar
# 编写Dockerfile文件
[root@docker-0002 ~]# vim myimg/Dockerfile 
FROM mylinux:latest
COPY myfile.tar /var/tmp/
ADD  myfile.tar /tmp/
CMD  ["/bin/bash"]
# 制作镜像
[root@docker-0002 ~]#docker build -t img3:latest myimg/
# 使用容器查看内容
[root@docker-0002 ~]#dcoker run -it --rm img3:latest
[root@c6f9f4dbc573 ~]#ls /var/tmp/
myfile.tar
[root@c6f9f4dbc573 ~]#ls /tmp
hosts  issue  ks-script-jr03uzns  ks-script-pu9ezlau
```

#### 语法案例4

> + 获取当前用户id并在tmp目录下创建file1文件
> + 切换为nobody用户，获取用户id，并在tmp目录下创建file2
> + 设置环境变量mymsg，并调用该变量
> + 切换工作目录为tmp目录

```shell
# 编辑Dockerfile
[root@docker-0002 ~]# vim myimg/Dockerfile 
FROM mylinux:latest
RUN  id && touch /tmp/file1
USER nobody
RUN  id && touch /tmp/file2
ENV  mymsg="Hello World"
WORKDIR /tmp
CMD  ["/bin/bash"]

# 创建镜像
[root@docker-0002 ~]# docker build -t img4:latest myimg
......
Successfully built eb8b669cbe7c
Successfully tagged img3:latest

# 创建并进入容器
[root@docker-0002 ~]#docker run -it --rm img4:latest

# USER 指令设置使用 nobody 用户运行容器
bash-4.4$ id
uid=65534(nobody) gid=65534(nobody) groups=65534(nobody)

# USER 指令前创建的文件是 root 权限，之后是 USER 用户权限
bash-4.4$ ls -l /tmp/file? 
-rw-r--r-- 1 root   root   0 Feb  5 05:25 /tmp/file1
-rw-r--r-- 1 nobody nobody 0 Feb  5 05:25 /tmp/file2

# ENV环境变量可以直接调用
bash-4.4$ echo ${mymsg}  
Hello World

# WORKDIR 把工作目录设置到 /tmp
bash-4.4$ pwd
/tmp
```

### Appache镜像

> 制作方法：开启两个终端边实操边编写Dockerfile文件

#### 准备配置文件

```shell
[root@docker-0002 ~]#mkdir httpd
# 设置测试页面
[root@docker-0002 ~]#echo 'Welcom To The Apache' > httpd/index.html
[root@docker-0002 ~]#echo '<?php> phpinfo(); </php>' > httpd/info.php
[root@docker-0002 ~]#tar -cf httpd/myweb.tar.gz -C httpd/ index.html info.php
# 获取配置文件
[root@docker-0002 ~]#docker run -itd --name myweb mylinux:latest
[root@docker-0002 ~]#docker exec -it myweb dnf -y install httpd
[root@docker-0002 ~]#docker exec -it c6f9f4dbc573 /bin/bash
[root@c6f9f4dbc573 ~]#vim /etc/httpd/conf.modules.d/00-mpm.conf
11 LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
23 #LoadModule mpm_event_module modules/mod_mpm_event.so
[root@c6f9f4dbc573 ~]#exit
[root@docker-0002 ~]#docker cp myweb:/etc/httpd/conf.modules.d/00-mpm.conf httpd/
[root@docker-0002 ~]#docker rm -f myweb
```

#### 验证镜像

```shell
# 编写Dockerfile文件
[root@docker-0002 ~]#vim httpd/Dockerfile
FROM mylinux:latest
RUN dnf -y install httpd php && dnf clean all
ADD myweb.tar.gz /var/www/html/
COPY 00-mpm.conf /etc/httpd/conf.modules.d/00-mpm.conf 
ENV LANG=C
WORKDIR /var/www/html/
EXPOSE 80/tcp
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]

# 制作镜像
[root@docker-0002 ~]#docker build -t httpd:latest httpd/
# 创建容器
[root@docker-0002 ~]#docker run -itd --name web1 httpd:latest
# 验证服务
[root@docker-0002 ~]#docker inspect web2 | grep -i ipaddr
[root@docker-0002 ~]#curl 172.17.0.2
Welcom To The Apache
```

### Nginx镜像

#### 编译软件包

> 准备nginx的源码包
>
> nginx-1.22.1.tar.gz 

```shell
# 安装编译工具和依赖软件包
[root@docker-0002 ~]# dnf install -y openssl-devel pcre-devel gcc make 
# 编译安装
[root@docker-0002 ~]# tar zxf nginx-1.22.1.tar.gz 
[root@docker-0002 ~]# cd nginx-1.22.1/
[root@docker nginx-1.22.1]# ./configure --prefix=/usr/local/nginx --with-pcre --with-http_ssl_module
[root@docker nginx-1.22.1]# make && make install
# 设置默认首页
[root@docker nginx-1.22.1]# echo 'Nginx is running !' >/usr/local/nginx/html/index.html
```

#### 制作镜像

```shell
[root@docker-0002 ~]# mkdir nginx
# 将编译好的 nginx 打包，这里必须使用相对路径
[root@docker-0002 ~]# tar czf nginx/nginx.tar.gz -C /usr/local nginx
# 编写dockerfile文件
[root@docker-0002 ~]# vim nginx/Dockerfile 
FROM mylinux:latest
RUN  dnf install -y pcre openssl && dnf clean all
ADD  nginx.tar.gz /usr/local/
ENV  PATH=${PATH}:/usr/local/nginx/sbin
WORKDIR /usr/local/nginx/html
EXPOSE 80/tcp
CMD  ["nginx", "-g", "daemon off;"]  # 放入前台运行

[root@docker-0002 ~]# docker build -t nginx:latest nginx
Successfully tagged nginx:latest
```

#### 验证镜像

```shell
# 查看镜像并创建容器
[root@docker-0002 ~]# docker images nginx:latest
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
nginx        latest    645dd2d9a8ec   3 minutes ago   274MB
[root@docker-0002 ~]# docker run -itd --name web3 nginx:latest
e440b53a860a93cc2b82ad0367172c344c7207def94c4c438027c60859e94883

# 查看容器地址并访问验证
[root@docker-0002 ~]# docker inspect web3 |grep -i IPAddress
[root@docker-0002 ~]# curl http://172.17.0.3/
Nginx is running !
```

#### 多阶段镜像

> 将上方编译程序也通过容器来实现
>
> 概括来说：将多个dockerfile融为一体

```shell
# 准备前期工作
[root@docker-0002 ~]#docker rm -f $(docker ps -qa)
[root@docker-0002 ~]#docker rmi nginx
[root@docker-0002 ~]#mv nginx-1.22.1.tar.gz nginx/

# 编写Dockerfile
[root@docker nginx]# vim nginx/Dockerfile
# 第一阶段编译程序
FROM mylinux:latest as builder # 给该镜像取别名
ADD  nginx-1.22.1.tar.gz /
WORKDIR /nginx-1.22.1
RUN  dnf install -y openssl-devel pcre-devel gcc make
RUN  ./configure --prefix=/usr/local/nginx --with-pcre --with-http_ssl_module
RUN  make && make install
RUN  echo 'Nginx is running !' >/usr/local/nginx/html/index.html

# 第二阶段最终镜像
FROM mylinux:latest
RUN  dnf install -y pcre openssl && dnf clean all
COPY --from=builder /usr/local/nginx /usr/local/nginx # 从builder镜像中拷贝编译好的文件到该镜像中
ENV  PATH=${PATH}:/usr/local/nginx/sbin
WORKDIR /usr/local/nginx/html
EXPOSE 80/tcp
CMD  ["nginx", "-g", "daemon off;"]

[root@docker-0002 ~]#docker build -t nginx:latest nginx
[root@docker-0002 ~]#docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
nginx        latest    cdffc4d6fe07   15 seconds ago   274MB
<none>       <none>    851c0af701fd   31 seconds ago   477MB
httpd        latest    578dec7d10bb   3 hours ago      299MB
mylinux      latest    8744fedcf7c8   23 hours ago     250MB
rockylinux   8.5       210996f98b85   2 years ago      205MB
[root@docker-0002 ~]#docker rmi 851c0af701fd # 该镜像为第一阶段编译程序的镜像，删除即可
[root@docker-0002 ~]#docker run -itd --name web1 nginx:latest
[root@docker-0002 ~]#docker inspect web1 | grep -i ipaddr
[root@docker-0002 ~]#curl 172.17.0.2
Nginx is running !
```

#### 总结

> 采用多阶段镜像制作的好处：
>
> 1. 减小最终镜像大小
>    + 多阶段构建允许将构建过程划分为多个阶段，比如一个用于编译和构建应用的阶段，另一个仅包含运行时所需的最小化组件。
>    + 编译或构建阶段可以包含大量工具链、依赖包和临时文件，这些在生成可执行文件后就不需要了。通过仅从构建阶段复制编译后的应用程序到新的运行时镜像，可以极大地减少生产环境镜像的大小。
> 2. 提升安全性
>    + 构建过程中使用的工具和依赖不包含在最终镜像中，降低了攻击面，因为不需要在运行时环境中维护那些只在构建时用到的软件。
> 3. 优化资源利用
>    + 由于不同的构建阶段可以复用层，且不需要在最终镜像中包含所有历史构建步骤，这节省了存储空间并加快了镜像拉取速度。
> 4. 简化构建流程
>    + 在单个Dockerfile中就能完成整个构建和清理过程，使得构建逻辑更加清晰，也更便于CI/CD集成。
>    + 可以方便地在一个阶段中构建项目，然后在下一个阶段中只提取必要的部分，避免手动清理工作。
> 5. 保持镜像整洁
>    + 避免了在生产镜像中出现开发工具、源代码、构建缓存等不必要的内容，从而创建出更为精简、专注的容器镜像。
> 6. 增强灵活性
>    + 不同构建阶段可以使用不同基础镜像，可以根据目标平台或架构定制不同阶段的构建环境和运行环境。

### php-fpm镜像

#### 手工部署

```shell
[root@docker-0002 ~]# docker run -it --name myphp mylinux:latest
[root@cbcf3d90c02e /]# dnf install -y php-fpm
# 修改配置文件
[root@cbcf3d90c02e /]# vim /etc/php-fpm.d/www.conf
38:  listen = 127.0.0.1:9000
# 创建目录，并授权
[root@cbcf3d90c02e /]# mkdir -p /run/php-fpm
[root@cbcf3d90c02e /]# chown -R nobody.nobody /var/log/php-fpm /run/php-fpm
# 使用 sudo 且换用户
[root@cbcf3d90c02e /]# dnf install -y sudo
[root@cbcf3d90c02e /]# sudo -u nobody /bin/bash
# 使用 nobody 启动服务
bash-4.4$ /usr/sbin/php-fpm --nodaemonize
[04-Sep-2023 09:58:01] NOTICE: [pool www] 'user' directive is ignored when FPM is not running as root
[04-Sep-2023 09:58:01] NOTICE: [pool www] 'group' directive is ignored when FPM is not running as root
[04-Sep-2023 09:58:01] NOTICE: fpm is running, pid 1
[04-Sep-2023 09:58:01] NOTICE: ready to handle connections
[04-Sep-2023 09:58:01] NOTICE: systemd monitor interval set to 10000ms
```

#### 制作镜像

```shell
# 开启一个新终端拷贝刚刚手动部署的php-fpm文件
[root@docker-0002 ~]#docker cp php:/etc/php-fpm.d/www.conf php/
[root@docker-0002 ~]#docker rm -f php
# 编写 dockerfile 文件
[root@docker-0002 ~]# vim php/Dockerfile
FROM mylinux:latest
RUN  dnf install -y php-fpm && dnf clean all && \
     mkdir -p /run/php-fpm && \
     chown -R nobody.nobody /run/php-fpm /var/log/php-fpm
COPY www.conf /etc/php-fpm.d/www.conf
USER nobody
EXPOSE 9000/tcp
CMD ["/usr/sbin/php-fpm", "--nodaemonize"]

[root@docker-0002 ~]# docker build -t php-fpm:latest php
Successfully tagged php-fpm:latest
```

> 多个命令需要同时使用多个RUN时，尽量用一条命令去执行

#### 验证镜像

```shell
# 查看镜像并创建容器
[root@docker-0002 ~]# docker images php-fpm:latest
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
php-fpm      latest    b2404bd119b0   48 seconds ago   275MB
[root@docker-0002 ~]# docker run -itd --name myphp php-fpm:latest
6eeff6af4a6469c298944b2bdd2ba69f32ebcbc6cb683a0a05af4eefbf90e8c1

# 验证服务
[root@docker-0002 ~]# docker exec -it php /bin/bash
# 验证用户
bash-4.4$ id
uid=65534(nobody) gid=65534(nobody) groups=65534(nobody)
# 我们无法直接调用 php 服务，可以通过查看进程验证服务
bash-4.4$ ps -ef
UID          PID    PPID  C  STIME  CMD
nobody         1       0  0  16:13  php-fpm: master process (/etc/php-fpm.conf)
nobody         7       1  0  16:13  php-fpm: pool www
nobody         8       1  0  16:13  php-fpm: pool www
nobody        17       0  0  16:13  /bin/bash
nobody        19      17  0  16:13  ps -ef
bash-4.4$ exit
# 测试完毕删除
[root@docker-0002 ~]# docker rm -f php
```

## Docker私有仓库

### 主机购买

| 主机名   | ip地址       | 最低配置    |
| -------- | ------------ | ----------- |
| registry | 192.168.1.35 | 2CPU,4G内存 |

### 安装私有仓库

```shell
# 在 registry 上安装私有仓库
[root@registry ~]# dnf install -y docker-distribution
# 启动私有仓库，并设置开机自启动
[root@registry ~]# systemctl enable --now docker-distribution
# 查询是否监听5000端口
[root@registry ~]# ss -lnt
State  Recv-Q Send-Q  Local Address:Port   Peer Address:Port Process 
LISTEN 0      128           0.0.0.0:22          0.0.0.0:*            
LISTEN 0      1024                *:5000
```

### 客户端配置

```sh
# 添加一行
[root@docker-0002 ~]#vim /etc/hosts
192.168.1.35 registry
# 修改配置文件
[root@docker-0002 ~]#vim /etc/docker/daemon.json
{
    "registry-mirrors": ["http://registry:5000"],
    "insecure-registries":["registry:5000"] # 由于镜像仓库只能信任https安全的链接，所以在这里添加指定使用不安全的连接地址
}
[root@docker-0002 ~]#systemctl restart docker
[root@docker-0002 ~]#docker info
Insecure Registries:
  registry:5000
  127.0.0.0/8
 Registry Mirrors:
  http://registry:5000/
```

### 上传镜像

```shell
# 给镜像设置标签
[root@docker-0002 ~]#docker tag nginx:latest registry:5000/img/web:nginx
# 上传nginx镜像
[root@docker-0002 ~]#docker push registry:5000/img/web:nginx
The push refers to repository [registry:5000/img/web]
。。。。
nginx: digest: sha256:1bba4afe4fdf4a92b36149f0e0f6d8ce14e871941bfddd16bed12e12d2e40b6b size: 1163
# 上传httpd镜像
[root@docker-0002 ~]#docker tag httpd:latest registry:5000/img/web:httpd
[root@docker-0002 ~]#docker push  registry:5000/img/web:httpd

# 上传到library（默认镜像）
[root@docker-0002 ~]#docker tag mylinux:latest registry:5000/library/mylinux:latest
[root@docker-0002 ~]#docker push registry:5000/library/mylinux:latest
```

### 验证测试

> 查看镜像名称： curl http://仓库IP:5000/v2/_catalog
> 查看镜像标签： curl http://仓库IP:5000/v2/镜像路径/tags/list
> 使用易读格式： python3 -m json.tool

```shell
# 查询仓库镜像
[root@docker-0002 ~]#curl http://registry:5000/v2/_catelog
{"repositories":["img/web"]}
[root@docker-0002 ~]#curl http://registry:5000/v2/img/web/tags/list
{"name":"img/web","tags":["httpd","nginx"]}
# 使用json格式化
[root@docker-0002 ~]#curl -s http://registry:5000/v2/img/web/tags/list | python3 -m json.tool
{
    "name": "img/web",
    "tags": [
        "httpd",
        "nginx"
    ]
}

# 查询默认镜像仓库信息
[root@docker-0002 ~]# curl -s http://registry:5000/v2/library/mylinux/tags/list | python3 -m json.tool
{
    "name": "library/mylinux",
    "tags": [
        "latest"
    ]
}
```

### 创建容器

```shell
# 删除现机器所有容器与镜像
[root@docker-0002 ~]#docker rm -f $(docker ps -qa)
[root@docker-0002 ~]#docker rmi -f $(docker images -qa)
# 拉去远程仓库镜像
root@docker-0002 ~]# docker pull registry:5000/img/web:nginx
nginx: Pulling from img/web
................
registry:5000/img/web:nginx
# 拉去默认仓库镜像
[root@docker-0002 ~]#docker pull mylinux:latest


# 删除方便测试
[root@docker-0002 ~]#docker rm -f $(docker ps -qa)
[root@docker-0002 ~]#docker rmi -f $(docker images -qa)
# 直接使用仓库镜像创建容器
[root@docker-0002 ~]#docker run -itd --name web1 registry:5000/img/web:nginx
[root@docker-0002 ~]#docker ps
CONTAINER ID   IMAGE                         COMMAND                  CREATED              STATUS              PORTS     NAMES
754c58827002   registry:5000/img/web:nginx   "nginx -g 'daemon of…"   About a minute ago   Up About a minute   80/tcp    web1
[root@docker-0002 ~]#docker images
REPOSITORY              TAG       IMAGE ID       CREATED       SIZE
registry:5000/img/web   nginx     cdffc4d6fe07   3 hours ago   274MB
# 直接使用默认仓库镜像创建容器
[root@docker-0002 ~]# docker run -itd --name test mylinux:latest
[root@docker-0002 ~]# docker ps
CONTAINER ID   IMAGE                         COMMAND                  CREATED         STATUS         PORTS     NAMES
dbaa63f818fb   mylinux:latest                "/bin/bash"              6 seconds ago   Up 5 seconds             test
754c58827002   registry:5000/img/web:nginx   "nginx -g 'daemon of…"   2 minutes ago   Up 2 minutes   80/tcp    web1
[root@docker-0002 ~]# docker images
REPOSITORY              TAG       IMAGE ID       CREATED        SIZE
registry:5000/img/web   nginx     cdffc4d6fe07   3 hours ago    274MB
mylinux                 latest    8744fedcf7c8   26 hours ago   250MB
```

> 经过测试，直接使用仓库镜像创建容器原理：先下载镜像到本地再制作成容器
>
> 

# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习
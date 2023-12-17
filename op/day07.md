- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
	- [python动态网站部署](#python动态网站部署)
		- [装包、依赖](#装包依赖)
		- [测试](#测试)
		- [安装uwsgi](#安装uwsgi)
		- [编写配置](#编写配置)
		- [nginx配置](#nginx配置)
		- [测试访问](#测试访问)
		- [配置进程](#配置进程)
		- [测试](#测试-1)
		- [配置静态网站](#配置静态网站)
		- [测试](#测试-2)
	- [灰度发布-基于IP](#灰度发布-基于ip)
		- [概念](#概念)
		- [编写配置](#编写配置-1)
		- [测试](#测试-3)
		- [弊端](#弊端)
	- [灰度发布-基于用户名](#灰度发布-基于用户名)
		- [概念](#概念-1)
		- [编写配置](#编写配置-2)
		- [测试](#测试-4)
	- [网站限流](#网站限流)
		- [基于全局限速](#基于全局限速)
		- [基于主机限速](#基于主机限速)
		- [基于目录限速](#基于目录限速)
		- [弊端](#弊端-1)
		- [限制用户下载](#限制用户下载)
	- [防盗链](#防盗链)
		- [作用](#作用)
		- [配置](#配置)
		- [测试](#测试-5)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)

# 学习目标

Nginx+uWSGI

灰度发布

访问限制

# 课堂笔记（命令）



# 课堂笔记（文本）

## python动态网站部署

### 装包、依赖

```
yum -y install gcc make python3 python3-devel
# down文件链接下载
pip3 install pytz-2022.6-py2.py3-none-any.whl  
pip3 install Django-1.11.8-py2.py3-none-any.whl 
pip3 install django-bootstrap3-11.0.0.tar.gz 
tar -xf python-project-demo.tar.gz # 测试项目 
```

### 测试

```
cd python-project-demo
python3 runserver 0.0.0.0:8000 # 使当前主机所有IP访问该端口；使用runserver效率低，需要使用uwsgi来部署较好
http://192.168.99.5:8000
```

### 安装uwsgi

```
yum -y install uWSGI-2.0.21.tar.gz 
```

### 编写配置

```
vim myproject.ini

[uwsgi]
socket=127.0.0.1:8000 # 与web服务{nginx}通信的接口，类似php-fpm中的网络或进程通信
chdir=/root/python/python-project-demo # 指定python项目绝对路径
wsgi-file=learning_log/wsgi.py # 指定usgi与python项目连接的媒介
daemonize=/var/log/uwsgi.log            #指定日志文件位置

uwsgi --ini myproject.ini #读取myproject.ini运行uwsgi
```

### nginx配置

```
vim /usr/local/nginx/conf/nginx.conf
server{
	....
	location / {
            root   html;
            index  index.html index.htm;
            uwsgi_pass 127.0.0.1:8000;        #动态页面交给uWSGI
            include uwsgi_params;            #调用uWSGI配置文件
}
	....
}

cd  /usr/local/nginx/
sbin/nginx # 启动nginx
```

### 测试访问

```
浏览器访问
http://192.168.99.5
```

### 配置进程 

```
vim myproject.ini

[uwsgi]
socket=127.0.0.1:8000 # 与web服务{nginx}通信的接口，类似php-fpm中的网络或进程通信
chdir=/root/python/python-project-demo # 指定python项目绝对路径
wsgi-file=learning_log/wsgi.py # 指定usgi与python项目连接的媒介
daemonize=/var/log/uwsgi.log            #指定日志文件位置
process=5 # 开启五个进程，类似php-fpm中的 pm.max_children 
master=true # 开启主进程，开启了5个进程实际时6个因为有个主进程
```

### 测试

```
killall uwsgi
uwsgi --ini myproject.ini
ss -ntulp|grep uwsgi 
```

### 配置静态网站

```
# python动静分离配置
location /static {
    root html;
}
location / {
    root   html;
    index  index.html index.htm;
    uwsgi_pass 127.0.0.1:8000;
    include uwsgi_params;
}
```

### 测试

```
cd /usr/local/nginx
sbin/nginx -s reload
mkdir html/static
echo this is static > html/static/index.html

# 浏览器访问
192.168.99.5/static #静态
192.168.99.5/ #动态
```

## 灰度发布-基于IP

### 概念

> 含义：
>
> 使用比较平稳的过渡方式升级或者替换产品项目的方法统称
>
> 作用：
>
> 及时发现项目问题
>
> 尽早获取用户反馈信息，以改进产品
>
> 如果项目产生问题，可以将问题影响控制到最小范围

### 编写配置

> 8001、8002 为需新更新的业务
>
> 80 老业务

```
# proxy 99.5机器nginx配置-配置集群

upstream s8001{  # 定义集群1：新业务
	server 192.168.99.100:8001;
}   
upstream s8002{  # 定义集群2：新业务
	server 192.168.99.200:8002;
}
upstream default{ # 定义集群：老业务
	server 192.168.99.100:80;
	server 192.168.99.200:80;
}
 server {
        listen       80;
        server_name  localhost;
        set $group "default"  # 定义一个变量为group，默认值default
        if ($remote_addr ~ "192.168.99.1"){ # $remote_addr，nginx内置变量，用户访问时的IP；匹配客户的IP为192.168.99.1开头时将变量$group赋值为s8001，及访问集群s8001；这里的包含指的是客户的IP为192.168.99.1开头时即99.100、99.199都会访问集群1
                set $group s8001;
        }
        if ($remote_addr ~ "192.168.99.2"){ # 包含2时访问集群2，即99.2、99.254等包含2的IP
                set $group s8002;
        }
        # 上述两种规则都不匹配时，即IP既没有包含1也没有包含2(99.5、99.72等)时，就访问默认集群default
        location / {
            root   html;
            index  index.html index.htm;
            proxy_pass http://$group; # 通过变量调用集群
        }
        }
```

```
# web1 99.100 nginx配置-配置网站
    server{  # 添加一个虚拟主机，监听8001端口
        listen 8001;
        server_name localhost;
        root html8001;
        index index.html;
}
mkdir html8001
echo web1-8001 > html8001/index.html
echo web1-80 > html/index.html
 
# web2 99.200 nginx配置同上-配置网站
    server{  # 添加一个虚拟主机，监听8002端口
        listen 8002;
        server_name localhost;
        root html8002;
        index index.html;
}
mkdir html8001
echo web2-8002 > html8002/index.html
echo web2-80 > html/index.html
```

### 测试

```
# 99.5主机访问
curl 192.168.99.5
web1-80
web2-80
# 99.100主机访问
curl 192.168.99.100
web1-8001
# 99.254主机访问
curl 192.168.99.254
web28002
```

### 弊端

> 1. P地址不稳定：IP地址可能会经常变化，尤其是在移动网络和云服务中。如果用户的IP地址发生变化，可能会导致他们在不同的灰度发布组中，从而破坏了预期的控制流量。
>
> 2. IP地址共享：在某些情况下，多个用户可能共享同一个IP地址，特别是在使用代理服务器、NAT（网络地址转换）或共享网络连接时。这样的情况下，无法准确将用户分配到特定的灰度发布组，可能会导致错误的流量控制。
>
> 3. 不支持多地域部署：当应用程序跨多个地理位置进行部署时，使用IP地址进行灰度发布可能会有问题。由于IP地址在不同的地理位置具有不同的分配方式，可能无法精确地将用户引导到目标灰度发布组。
>
> 3. 缺乏细粒度控制：基于IP的灰度发布通常只能实现相对简单的流量控制，例如按比例将用户引导到不同的灰度发布组。如果需要更细粒度的控制，例如按照用户属性或行为进行灰度发布，则IP地址可能无法提供足够的信息。

## 灰度发布-基于用户名

### 概念

> 根据用户名进行不同内容推送

### 编写配置

```
# ngixn配置
vim /usr/local/nginx/conf/nginx.conf
location ~ \.php$ {
    root           html;
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    include        fastcgi.conf;
}
cd /usr/local/nginx
tar -xf ~/lnmp_soft/php_scripts/php-session-demo.tar.gz 
mv php-session-demo/* html/
sbin/nginx
```

```
# php-fpm 使用网络通信
vim /etc/php-fpm.d/www.conf
	listen = 127.0.0.1:9000
systemcl start php-fpm
```

```
# 页面配置，在html/home.php页面修改
Welcome :  <?php
if (preg_match( "/^abc/",$_SESSION['login_user'])){
echo "<a href='http://192.168.99.100'>Begin</a>";
}
else
{
echo "<a href='http://192.168.99.200'>Begin</a>";
}
?>
<br>
# 通过判断登陆的用户名，来推送不同页面；以上为用户为abc开头推送99.100页面，反之推送99.200页面
```

### 测试

```
#浏览器
http://192.168.99.5/index.php 
#以abc用户登陆|bca用户登陆
```

## 网站限流

### 基于全局限速

```
# 修改nginx配置文件
vim conf/nginx.conf
http{
...
	limit_rate 100k;  # 限速为100k
	server{
	.......
	location / {
	....
	....
	}
	}
....
}

cd /usr/local/nginx
# 生成指定大小测试文件
dd if=/dev/zero of=html/test.img bs=100M count=1 # 生成一个100M的文件
# linux安装软件测试
yum -y install wget
wget 192.168.99.100/test.img
# 浏览器测试
http://192.168.99.100/test.img  #看到下载速度在100k上下
```

### 基于主机限速

```
# 修改nginx配置文件
vim conf/nginx.conf
http{
...
	limit_rate 100k;  # 限速为100k
	server{
	.......
	location / {
	....
	....
	}
	}
....
}
```

### 基于目录限速

```
# 修改nginx配置文件
vim conf/nginx.conf
http{
...
	limit_rate 100k;  # 限速为100k
	server{
	.......
	location /file_a {
		limit_rate 300k; #用户访问file_a目录时限速为300k
	}
	location /file_b {
		limit_rate 0k; #用户访问file_a目录时不限速
	}
	location / {
	....
	....
	}
	}
....
}
# 离的越近，就匹配谁的limit_rate

web1
wget 192.168.99.100/file_a/test.img # 限速为300k
wget 192.168.99.100/file_b/test.img # 不限速
```

### 弊端

```
当用户多开进行下载时，将会把服务器带宽占满
例如：服务器上总共有5个文件，且限速为100k，用户如果开启多个客户端进行下载，同时下载5个文件，此时用户相当于占用带宽500k，那如果1000个客户端下载，那服务器带宽直接拉满，宕机。
```

### 限制用户下载

```
# 修改nginx配置文件
vim conf/nginx.conf
http{
...
	limit_conn_zone $binary_remote_addr zone=addr:10m;
	limit_rate 100k;  # 限速为100k
	server{
	.......
	location /app{
		 limit_rate 30k;
         limit_conn addr 1;
	}
	location /file_a {
		limit_rate 300k; #用户访问file_a目录时限速为300k
	}
	location /file_b {
		limit_rate 0k; #用户访问file_a目录时不限速
	}
	location / {
	....
	....
	}
	}
....
}

web1
wget 192.168.99.100/app/test.img # 可正常下载
# 在开启一个终端
wget 192.168.99.100/app/test.img #提示有个连接正在下载
```

## 防盗链

### 作用

> 防止用户盗用链接，保护资源不被未授权的网站户用户直接引用，通过判断用户请求头中的referer值来判定

### 配置

```
vim conf/nginx.conf
server{
...
	valid_referers none 192.168.99.100; # 判断客户端请求头中的referer字段中是否为空或含有192.168.99.100；none:空
        if ($invalid_referer){ # 判断上述测试无效则成立
                return 403; # 返回错误提示
        }
...
}

cd /usr/local/nginx
sbin/nginx -s reload
```

```
# web1
vim html/index.html
web1
<a href="http://192.168.99.100/a.html">click/a>

echo succes > html/a.html


# web2
cd /usr/local/nginx
vim html/index.html
web2
<a href="http://192.168.99.100/a.html">click</a>

```

### 测试

```
# 浏览器访问192.168.99.100点击click正常跳转到a.html页面；浏览器访问192.168.99.200点击click报错为403
```

# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习

nginx+tomcat

rpmbuild

VPN：gre、L2TP
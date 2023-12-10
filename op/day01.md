- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [构建Nginx服务器](#构建nginx服务器)
    - [安装包及依赖](#安装包及依赖)
    - [指定编译信息](#指定编译信息)
    - [二进制及主程序安装](#二进制及主程序安装)
    - [目录文件信息](#目录文件信息)
    - [创建用户](#创建用户)
    - [启动](#启动)
    - [查看端口](#查看端口)
    - [停止](#停止)
    - [查询配置信息](#查询配置信息)
    - [重新加载配置文件](#重新加载配置文件)
    - [编写测试页面](#编写测试页面)
  - [修改配置文件](#修改配置文件)
    - [添加用户认证](#添加用户认证)
    - [添加用户及密码](#添加用户及密码)
    - [加载配置配置文件](#加载配置配置文件)
    - [访问测试](#访问测试)
    - [恢复配置文件](#恢复配置文件)
  - [虚拟主机](#虚拟主机)
    - [参数详解](#参数详解)
    - [配置虚拟主机](#配置虚拟主机)
  - [网站加密](#网站加密)
    - [加密分类](#加密分类)
    - [网站配置https](#网站配置https)
    - [生成公私钥](#生成公私钥)
    - [创建文件夹及访问](#创建文件夹及访问)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)

# 学习目标

Nginx安装部署

用户认证

Web虚拟主机

HTTPS加密网址

# 课堂笔记（命令）



# 课堂笔记（文本）

## 构建Nginx服务器

### 安装包及依赖

```
yum -y install gcc make            #安装编译工具
yum -y install pcre-devel            #正则表达式依赖包
yum -y install openssl-devel        #SSL加密依赖包
```

### 指定编译信息

Nginx是一个模块化软件

```
./configure --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_ssl_moudle
--prefix  指定安装路径
--user 指定用户
--group 指定组
--with-http_ssl_moudle 开启ssl加密功能
```

### 二进制及主程序安装

```
make && make install
```

### 目录文件信息

> conf	配置文件
>
> html	网站页面
>
> log	日志
>
> sbin	主程序

### 创建用户

```
# 由于指定了用户 组，
useradd nginx -s /sbin/nologin 
```

### 启动

```
cd /usr/local/nginx
/sbin/nginx
curl 192.168.99.5
```

### 查看端口

```
ss -antulp|grep nginx
```

### 停止

```
cd /usr/local/nginx
sbin/nginx -s top
```

### 查询配置信息

```
cd /usr/local/nginx
sbin/nginx -V
```

### 重新加载配置文件

```
# 修改配置文件后，需要加载配置文件
cd /usr/local/nginx
sbin/nginx -s reload  # 注意，服务开启时才能使用
```

### 编写测试页面

```
cd /usr/local/nginx
echo nginx-test～ > html/abc.html
curl 192.168.99.5/abc.html
```

## 修改配置文件

### 添加用户认证

**作用：**用户访问网页时，需输入指定用户名密码才能进行访问

```
vim /usr/local/nginx/conf/nginx.conf

server {
        listen       80;
        server_name  localhost;
        auth_basic "Input Password:";    #认证提示符信息,开启认证
        auth_basic_user_file  "/usr/local/nginx/pass";        #认证的密码文件
        location / {
            root   html;
            index  index.html index.htm;
        }
  }
  
# 重新加载文件
sbin/nginx -s reload
```

### 添加用户及密码

```
# 安装加密密码软件
yum -y install httpd-tools

cd /usr/local/nginx
htpasswd -c pass yyh # 新创建用户，随后输入密码
cat pass # 查看生成的用户及密码
yyh:$apr1$Jk2iQD/s$/fD1bltVmVRgcg.yHgBQX.
hpasswd pass tom # 追加一个用户
cat /etc/pass
yyh:$apr1$Jk2iQD/s$/fD1bltVmVRgcg.yHgBQX.
tom:$apr1$RvEMl0dZ$B2MGYQi0LBxoLh9kES0bW0
```

### 加载配置配置文件

```
sbin/nginx -s reload
```

### 访问测试

> 浏览器访问测试

### 恢复配置文件

```
cd /usr/local/nginx
cp conf/nginx.conf.default /conf/ngixn.conf
```

## 虚拟主机

### 参数详解

```
listen  监听端口
server_name 域名
root 根目录
index 默认页
```

### 配置虚拟主机

```
cd /usr/local/nginx/
vim conf/nginx.conf
    server{
        listen 80;
        server_name www.b.com;
        root html_b;
        index index.html;
   }
# 重新加载文件
sbin/nginx -s reload
# 创建文件夹，编写默认页
mkdir html_b
echo test_b > html_b/index.html
# 添加本机域名映射
echo 192.168.99.5 www.b.com >> /etc/hosts
# 访问
curl www.b.com
```

## 网站加密

### 加密分类

> 加密算法一般分为对称算法、非对称算法、信息摘要。
>
> 对称算法有：AES、DES，主要应用在单机数据加密。
>
> 非对称算法有：RSA、DSA，主要应用在网络数据加密。
>
> 信息摘要：MD5、sha256，主要应用在数据完整性校验。

### 网站配置https

```
vim /usr/local/nginx/conf/nginx.conf
    # HTTPS server
    #
    server {
    listen       443 ssl;
    server_name  localhost;

    ssl_certificate      cert.pem;
    ssl_certificate_key  cert.key;

    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;

    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers  on;

    location / {
    root   https;
    index  index.html index.htm;
    }
    }
```

### 生成公私钥

```
# 公钥
openssl genrsa > /usr/local/nginx/conf/cert.key  
# 私钥
openssl req -x509 -key /usr/local/nginx/conf/cert.key >usr/local/nginx/conf/cert.pem
Country Name (2 letter code) [XX]:dc    国家名
State or Province Name (full name) []:dc    省份
Locality Name (eg, city) [Default City]:dc     城市
Organization Name (eg, company) [Default Company Ltd]:dc    公司
Organizational Unit Name (eg, section) []:dc    部门
Common Name (eg, your name or your server's hostname) []:dc    服务器名称
Email Address []:dc@dc.com     电子邮件
```

### 创建文件夹及访问

```
mkdir /usr/local/nginx/https
echo nginx-https > /usr/local/nginx/https/index.html
curl -k https://192.168.99.5
```



# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习
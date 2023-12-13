- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [Tomcat](#tomcat)
    - [安装](#安装)
    - [目录文件介绍](#目录文件介绍)
    - [启动](#启动)
    - [杀死](#杀死)
    - [访问](#访问)
    - [关闭程序](#关闭程序)
    - [编写静态测试页面](#编写静态测试页面)
    - [编写动态测试时间](#编写动态测试时间)
  - [虚拟主机](#虚拟主机)
    - [编写配置文件](#编写配置文件)
    - [编写测试页面](#编写测试页面)
    - [虚拟主机额外参数](#虚拟主机额外参数)
  - [war包](#war包)
    - [安包](#安包)
    - [打包](#打包)
    - [查看是否自动解包](#查看是否自动解包)
  - [tomcat访问路径与页面位置](#tomcat访问路径与页面位置)
    - [默认页面位置](#默认页面位置)
    - [测试默认页](#测试默认页)
    - [测试当前文件夹页](#测试当前文件夹页)
    - [测试当根目录页](#测试当根目录页)
    - [测试添加访问路径](#测试添加访问路径)
    - [总结](#总结)
  - [配置加密网站](#配置加密网站)
    - [修改配置文件](#修改配置文件)
    - [生成密钥对](#生成密钥对)
    - [测试访问](#测试访问)
  - [开启日志](#开启日志)
    - [修改配置](#修改配置)
    - [测试](#测试)
  - [Maven](#maven)
    - [装包](#装包)
    - [修改配置](#修改配置-1)
    - [导入sql文件](#导入sql文件)
    - [构建war包](#构建war包)
    - [访问网站](#访问网站)
- [快捷键](#快捷键)
- [问题](#问题)
  - [那些参数影响了omcat部署网站时的路径](#那些参数影响了omcat部署网站时的路径)
  - [使用keytool生成密钥文件命令时](#使用keytool生成密钥文件命令时)
  - [Tomcat配置虚拟主机的关键词是什么](#tomcat配置虚拟主机的关键词是什么)
  - [Maven的功能是什么](#maven的功能是什么)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)

# 学习目标

tomcat服务器

tomcat应用案例

maven应用

# 课堂笔记（命令）



# 课堂笔记（文本）

## Tomcat

### 安装

```
yum -y install java-1.8.0-openjdk
tar -xf apache-tomcat-8.0.30.tar.gz 

# 将程序拷贝到/usr/local下取名tomcat
cp -r /apache-tomcat-8.0.30 /usr/local/tomcat  
```

### 目录文件介绍

> bin 主程序
>
> conf 配置文件
>
> logs 日志存放
>
> webapps 页面
>
> temp 临时存放
>
> lib 库文件目录
>
> work 编译文件目录

### 启动

```
cd /usr/local/tomcat
/bin/startup.sh
```

### 杀死

```
ss -ntulp|grep java  # 查询tomcat进程 | 8005 8009 8080
killall java # 杀死进程
```

### 访问

```
http://192.168.99.100:8080
```

### 关闭程序

```
# 使用杀死进程
killall java

# 使用程序关闭
bin/shutdown.sh
```

### 编写静态测试页面

```
echo tomcat-test > webapps/ROOT/test.01
bin/startup.sh

curl 192.168.99.100:8080/test.01
```

### 编写动态测试时间

tomcat 默认支持动静分离

```
vim webapps/ROOT/test02.jsp
<html>
<body>
<center>
Now time is <%=new java.util.Date()%>
</center>
</body>
</html>

curl 192.168.99.100/test02.jsp
```

## 虚拟主机

### 编写配置文件

```
<Host name="域名" appBase="站点目录"></Host>
```

```
vim conf/server.xml # 虚拟主机需要写在<Engine></Engine>里面
<Engine>
.....
<Host name="www.a.com" appBase="web_b">
</Host>
.....
</Engine>
```

### 编写测试页面

```
cd /usr/local/tomcat
mkdir -p web_b/ROOT  # 创建站点目录
echo tomcat-test-B > web_b/ROOT/index.html # 编写测试页面
bin/shutdown.sh
bin/startup.sh
echo 192.168.99.100 www.a.com >> /etc/hosts # 使本机能够识别域名
curl www.a.com:8080
tomcat-test-B
```

### 虚拟主机额外参数

```
<Host name="" appBase=""
unpackWARs="true" # 开启war包自动解压
autoDeploy="true" # 自动更新网站功能>
</Host>
```

## war包

### 安包

```
yum -y install java-1.8.0-openjdk-devel
```

### 打包

```
jar -cf a.tar /var/log
```

### 查看是否自动解包

```
cd /usr/local/tomcat
cp a.war webapp/

ls 
a a.war
```

## tomcat访问路径与页面位置

### 默认页面位置

```
vim conf/server.xml

<Host name="www.b.com" appBase="web_b">            
<Context  path=""  docBase=""/> # path:访问www.b.com下的路径，不写即代表任意
# docBace：访问页面在什么位置，不写即是web_b下的任意位置
</Host>
```

### 测试默认页

前提：配置了一个虚拟主机，且使用的是域名`www.b.com`

```
cd /usr/local/tomcat

echo "web_b/ROOT/index.html" > web_b/ROOT/index.html
echo "web_b/index.html" > web_b/index.html

curl www.b.com:8080  # 修改配置前
web_b/ROOT/index.html

bin/shutdown.sh # 重启服务
bin/startup.sh
curl www.b.com:8080 # 修改配置后
web_b/index.html
```

### 测试当前文件夹页

```
cd /usr/local/tomcat

vim conf/server.xml
<Host name="www.b.com" appBase="web_b">            
<Context  path=""  docBase="abc"/> # docBase"abc":在当前web_b/abc目录下去访问页面
</Host>

mkdir web_b/abc
echo "web_b/abc/index.html" > web_b/abc/index.html
cul www.b.com:8080 # 继上一个未修改配置前测试
web_b/index.html

bin/shutdown.sh # 重启服务
bin/startup.sh
curl www.b.com:8080 # 修改配置后
web_b/abc/index.html
```

### 测试当根目录页

```
cd /usr/local/tomcat

vim conf/server.xml
<Host name="www.b.com" appBase="web_b">            
<Context  path=""  docBase="/abc"/> # docBase"abc":在根下abc目录下去访问页面
</Host>

mkdir /abc
echo "/abc/index.html" > /abc/index.html
cul www.b.com:8080 # 继上一个未修改配置前测试
web_b/abc/index.html

bin/shutdown.sh # 重启服务
bin/startup.sh
curl www.b.com:8080 # 修改配置后
/abc/index.html
```

### 测试添加访问路径

```
cd /usr/local/tomcat

vim conf/server.xml
<Host name="www.b.com" appBase="web_b">            
<Context  path="/test"  docBase="abc"/> 
# path="/test"访问路径/test/就将会去web_b/abc目录寻找资源；区分docBase的相对或绝对路径；
# docBase="abc":在目录web_b/abc目录下去访问页面
</Host>

mkdir web_b/abc
echo "web_b/abc/index.html" > web_b/abc/index.html

bin/shutdown.sh # 重启服务
bin/startup.sh
curl www.b.com:8080 # 修改配置后
/web_b/ROOT/index.html
curl www.b.com:8080/test/  #注意访问路径资源斜线要写对
/web_b/abc/index.html
```

### 总结

```
<Host name="域名" appBase="站点根目录">
<Context path="访问路径" docBae="页面位置"/> #上下文：设置URL路径与页面位置
</Host>
```

> path 内容将决定URL/后的路径
>
> docBase 内容将决定访问路径后看到什么内容页面，为空：默认ROOT根目录下存储；相对路径则是参照虚拟主机的目录下；绝对路径：则只参考所写的绝对路径下；
>
> 不管相对或绝对都需要创建所写的目录
>
> -------------------
>
> 可以配置多个上下文(Context)

## 配置加密网站

### 修改配置文件

```
cd /usr/local/tomcat
vim conf/server.xml

# 84行
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
               maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
               clientAuth="false" sslProtocol="TLS" keystoreFile="/usr/local/tomcat/keystore" keystorePass="123456" />  
# keystorreFile 存放公钥私钥的文件
# keystorePass 访问公钥私钥文件的密码
```

###  生成密钥对

```
keytool -genkeypair -alias tomcat -keyalg RSA -keystore /usr/local/tomcat/keystore # 敲击回车输入相关信息，否的时候输入y敲击回车回车

#-genkeypair     生成密钥对
#-alias tomcat     密钥别名
#-keyalg RSA     定义密钥算法为RSA算法
#-keystore         定义密钥文件存储在:/usr/local/tomcat/keystore

ls
a.war  conf      lib      logs    RELEASE-NOTES  temp     web_b
bin    keystore  LICENSE  NOTICE  RUNNING.txt    webapps  work

```

### 测试访问

```
bin/shutdown.sh
bin/startup.sh

curl -k https://www.b.com:8443
```

## 开启日志

> 默认主机已经开启日志
>
> 新添加的虚拟机需要配置日志

### 修改配置

```
vim conf/server.xml
# 在新添加的虚拟主机里添加
<Host name="www.b.com" appBase="web_b">
 <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="web_b_access_log" suffix=".log" # prefix：日志名，不同主机日志不能同名；suffix：日志格式
               pattern="%h %l %u %t &quot;%r&quot; %s %b" /> # 日志内容格式
</Host>
```

### 测试

```
bin/shutdown.sh
bin/startup.sh

curl www.b.com:8080

cat logs/web_b_access_log.log
```

## Maven

**实验**

搭建一个maven项目

准备一个半成品web项目，需要构建maven：cms.tar.gz

已经下载的.m2 Maven依赖

### 装包

```
# 下载maven包
apache-maven-3.6.3-bin.tar.gz # 解压
```

### 修改配置

```
mv apache-maven-3.6.3 /usr/local/maven
vim /usr/local/maven/conf/settings.xml
 <mirrors>
 ............
    <mirror>  # 修改镜像地址
      <id>nexus-aliyun</id>
      <mirrorOf>*</mirrorOf>
      <name>Nexus aliyun</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
 </mirrors>
```

### 导入sql文件

```
yum -y install mariadb mariadb-devel mariadb-server
systemctl start mariadb 

# 导入数据库文件
mysql -uroot < sql/install.sql 
# 修改root密码
mysqladmin password # 输入两次密码即可
# 进入数据库
mysql -uroot -p12345678 # 设置数据库密码
show databases; # 查询是否导入
```

### 构建war包

```
cd CMS
/usr/local/maven/bin/maven clean package

ls target
antrun             maven-archiver    shishuocms-2.0.1.war
classes            maven-status
generated-sources  shishuocms-2.0.1
```

### 访问网站

```
\cp target/shishuocms-2.0.1.war /usr/local/tomcat/webapps/ROOT.war
rm -rf /usr/local/tomcat/webapps/ROOT/
killall java
/usr/local/tomcat/bin/startup.sh

http://192.168.99.100:8080
```



# 快捷键



# 问题

## 那些参数影响了omcat部署网站时的路径

> appBase、path、docBase

## 使用keytool生成密钥文件命令时

> keytool -genkeypair -alias tomcat -keyalg RSA -keystore /usr/local/tomcat/keystore

## Tomcat配置虚拟主机的关键词是什么

> <Host name="域名" appBase="根路径"></Host>

## Maven的功能是什么

>  软件项目管理工具

# 补充



# 今日总结



# 昨日复习

uri过长：client_header_buffer_size



IP_hash

seesion共享

cookie--sessionid

多台服务器时，解决重复登陆：session共享

redis：key-value：数据存储在内存

yum -y install redis

修改redis配置文件：

注释bind 127.0.0.1

更改保护模式：protected_mode=no

yum -y install phpredis

修改php-fpm配置文件使其外部能访问

解决session不再以文件形式存储，而使用redis传输

指定路径：tcp://192.168.99.5



跨域

同源策略：协议、端口、域名

nginx配置文件解决同源策略：

add_header  "Access-Control-Allow-Origin" "192.168.99.5";

只允许192.168.99.5服务器访问时添加解除同源策略。
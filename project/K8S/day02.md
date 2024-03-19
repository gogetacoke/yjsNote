- [学习目标](#学习目标)
- [课堂笔记（文本）](#课堂笔记文本)
  - [Harbor使用](#harbor使用)
    - [部署harbor](#部署harbor)
    - [制作镜像](#制作镜像)
  - [华为云数据库](#华为云数据库)
  - [nacos部署](#nacos部署)
    - [概述](#概述)
    - [部署nacos](#部署nacos)
    - [导入数据](#导入数据)
    - [启动](#启动)
    - [配置负载均衡](#配置负载均衡)
  - [MQ服务部署](#mq服务部署)
    - [概述](#概述-1)
    - [服务安装配置](#服务安装配置)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)


# 学习目标

Harbor使用

华为云购买数据库

nacos注册中心使用

RocketMQ服务搭建

# 课堂笔记（文本）

## Harbor使用

### 部署harbor

```shell
# 使用跳板机连接到控制端
[root@jumpserver ~]# ssh -p2222 operation@192.168.1.252
operation@192.168.1.252's password:  # Aa123456+
                运维人员,  JumpServer 开源堡垒机

        1) 输入 部分IP，主机名，备注 进行搜索登录(如果唯一).
        2) 输入 / + IP，主机名，备注 进行搜索，如：/192.168.
        3) 输入 p 进行显示您有权限的主机.
        4) 输入 g 进行显示您有权限的节点.
        5) 输入 d 进行显示您有权限的数据库.
        6) 输入 k 进行显示您有权限的Kubernetes.
        7) 输入 r 进行刷新最新的机器和节点信息.
        8) 输入 s 进行中英日语言切换.
        9) 输入 h 进行显示帮助.
        10) 输入 q 进行退出.
Opt> p
  ID | 主机名                                   | IP       | 平台     | 组织     | 备注    
-----+------------------------------------------+----------+----------+----------+---------
  1  | harbor                                   | 19...30  | Linux    | Default  |         
  2  | nacos                                    | 19...13  | Linux    | Default  |         
  3  | rocketmq                                 | 19...14  | Linux    | Default  |         
  4  | test                                     | 19...01  | Linux    | Default  |         
页码：1，每页行数：15，总页数：1，总数量：4
提示：输入资产ID直接登录，二级搜索使用 // + 字段，如：//192 上一页：b 下一页：n
搜索：
Opt> 1
开始连接到 普通用户op@192.168.1.30  0.1

        Welcome to Huawei Cloud Service

Last login: Tue Mar  5 09:54:42 2024 from 192.168.1.252
[op@harbor ~]$ sudo -s
# 通过跳板机连接到了harbor主机，安装harbor仓库即可(前面已经安装过，此处简单跳过)
[root@harbor op]# dnf -y install docker-ce docker-compose-plugin
[root@harbor op]# systemctl enable --now docker

# 拷贝harbor-v2.7.0.tgz 到 harbor 主机 
[root@jumpserver ~]# scp /root/s4/public/harbor-v2.7.0.tgz 192.168.1.30:/root/

#创建 https 证书，后期harbor使用https的方式访问，使用openssl工具生成一个RSA密钥对，放到/root/tls/，名称为cert.key，秘钥位数2048位；使用openssl工具生成一个有效期为10年的证书，私钥使用之前生成的/root/tls/cert.key文件，将生成的证书保存到/root/tls/cert.crt文件中，并设置证书的主题信息
[root@harbor op]# cd /root/
[root@harbor ~]# tar -xf harbor-v2.7.0.tgz -C /usr/local/
[root@harbor ~]# cd /usr/local/harbor
[root@harbor harbor]# mkdir /root/tls
[root@harbor harbor]# openssl genrsa -out /root/tls/cert.key 2048
[root@harbor harbor]# openssl req -new -x509 -days 3650 -key /root/tls/cert.key -out /root/tls/cert.crt -subj "/C=CN/ST=BJ/L=BJ/O=Tedu/OU=NSD/CN=harbor"

# 修改配置文件
[root@harbor harbor]# cp harbor.yml.tmpl harbor.yml
[root@harbor harbor]# vim harbor.yml
  5 hostname: harbor               #harbor地址
  8 #http:                         #加上注释
 10   #port: 80                    #加上注释
 17   certificate: /root/tls/cert.crt    #nginx的证书文件位置
 18   private_key: /root/tls/cert.key    #nginx的秘钥文件位置

# 安装harbor
[root@harbor harbor]# ./install.sh 
[root@harbor harbor]# docker compose ps     # 查看容器状态,全部是healthy 
# 使用负载均衡发布443端口
```

### 制作镜像

```shell
# 进入test主机进行镜像制作
[root@harbor op]# exit
exit
[op@harbor ~]$ exit
logout
[Host]> 4
开始连接到 普通用户op@192.168.1.101  0.1

        Welcome to Huawei Cloud Service

Last login: Tue Mar  5 10:48:05 2024 from 192.168.1.252
[op@test ~]$ sudo -s
[root@test op]# echo -e "192.168.1.30\tharbor" >> /etc/hosts
# 安装制作镜像的服务
[root@test op]# dnf -y install docker-ce
[root@test op]# mkdir -p /etc/docker
[root@test op]# vim /etc/docker/daemon.json
{
   "registry-mirrors": ["https://harbor:443"],
   "insecure-registries": ["harbor:443"]
}
[root@test op]# systemctl enable --now docke
# 登陆harbor仓库，使用管理登陆密码即可
[root@test op]# docker login harbor:443
Username: admin     #用户名admin
Password:           #密码(admin123)
# 制作启动jar的基础镜像;拷贝基础镜像
[root@test op]# scp 192.168.1.252:/root/s4/public/myos.tar.xz /root/
[root@test op]# docker load -i /root/myos.tar.xz 
[root@test op]# mkdir /root/jar-package
[root@test op]# cd /root/jar-package/
[root@test jar-package]# vim Dockerfile
FROM myos:8.5
RUN yum -y install java-1.8.0-openjdk java-1.8.0-openjdk-devel && yum clean all
CMD ["/bin/bash"]
[root@harbor jar-package]# docker build -t  jar:base .

# 打标签上传jar:base镜像
[root@test jar-package]# docker tag jar:base harbor:443/tea/jar:base
[root@test jar-package]# docker push harbor:443/tea/jar:base

# 打标签上传tea:nginx镜像
[root@test jar-package]# docker tag myos:nginx harbor:443/tea/tea:nginx
[root@test jar-package]# docker push harbor:443/tea/tea:nginx
```

## 华为云数据库

> 登陆华为云按需购买数据库即可
>
> 1. 购买数据库RDS，主备
>
> 2. Redis GeminiDB  主备
>
> 3. 导入数据库文件
>
>    [访问网址下载](https://gitee.com/cc-0001/tea)

## nacos部署

### 概述

> + nacos是一个用于动态服务发现、配置管理和服务治理的开源平台。提供了服务注册和发现、配置管理、DNS服务、动态DNS和服务治理等功能，用于帮助开发者构建和管理微服务和云原生应用 
> + 微服务架构中，服务的数量庞大且动态变化，每个服务都需要向注册中心注册自己的地址和端口信息，以便其他服务可以发现和调用它 
> + Nacos作为注册中心，负责接收服务的注册信息，并提供服务发现接口，使服务消费者能够动态地获取服务提供者的地址和端口

### 部署nacos

```shell
# 通过jumpserver连接nacos
[root@jumpserver ~]# ssh -p2222 operation@192.168.1.252
operation@192.168.1.252's password: 
Opt> p          #输入p
  ID  | 主机名                                   | IP            | 平台     | 组织     | 备注    
------+------------------------------------------+---------------+----------+----------+---------
  1   | harbor                                   | 192.168.1.30 | Linux    | Default  |         
  2   | nacos                                    | 192.168.1.13  | Linux    | Default  |         
  3   | rocketmq                                 | 192.168.1.14  | Linux    | Default  |         
  4   | test                                     | 192.168.1.101 | Linux    | Default  |         
页码：1，每页行数：21，总页数：1，总数量：4
提示：输入资产ID直接登录，二级搜索使用 // + 字段，如：//192 上一页：b 下一页：n
搜索：
[Host]> 2       #进入nacos主机
[op@nacos ~]$ sudo -s

#另开一个jumpserver终端，解压tar包
[root@jumpserver ~]# tar -xf package.tar.gz  -C /root

#安装nacos，上传nacos到jumpserver主机的root目录下，拷贝到nacoa主机
[root@nacos op]# rsync -av 192.168.1.252:/root/package/nacos-server-2.0.2.tar.gz /root/
[root@nacos op]# tar -xpf /root/nacos-server-2.0.2.tar.gz -C /usr/local/

#安装JDK8环境
[root@nacos op]# yum -y install java-1.8.0-openjdk-devel

#配置nacos
[root@nacos ~]# cd /usr/local/nacos/
[root@nacos nacos]# vim conf/application.properties
 33 spring.datasource.platform=mysql # 数据库
 36 db.num=1 # 配置的数据库数量
 39 db.url.0=jdbc:mysql://192.168.1.12:3306/nacos?characterEncoding=utf8&connectTim    eout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&se    rverTimezone=UTC # 详细的数据源信息
 40 db.user.0=nacos # 数据库连接的账号
 41 db.password.0=Taren123 # 数据库密码
```

### 导入数据

> 在华为云RDS上创建一个数据库：nacos，添加用户nacos，权限：全部，主机：%，
>
> 创建导入任务导入nacos-mysql的sql到nacos库中

### 启动

```shell
# 修改nacos启动脚本
[root@nacos nacos]#vim bin/startup.sh
55 export MODE="standalone" # 以单机模式进行运行
# 启动nacos服务
[root@nacos nacos]#./bin/startup.sh
....
nacos is starting，you can check the /usr/local/nacos/logs/start.out
# 检查是否启动成功
[root@nacos nacos]#tail -2 logs/start.out
2024-03-05 14:32:45,239 INFO Nacos started successfully in stand alone mode. use external storage
# 查询进程及端口信息
[root@nacos nacos]#jps
12484 nacos-server.jar
12826 Jps
[root@nacos nacos]#ss -lnt|grep 8848
LISTEN 0      100                *:8848            *:*    
# 设置开机自启动
[root@nacos nacos]#echo "/usr/local/nacos/bin/startup.sh" >> /etc/rc.d/rc.local
[root@nacos nacos]#chmod +x /etc/rc.d/rc.local
```

### 配置负载均衡

> 使用华为云ELB监听192.168.1.13机器的8848端口，将其开方出来
>
> 浏览器访问：http://ELBIP:8848/nacos
>
> 账密：nacos  nacos

## MQ服务部署

### 概述

> + RocketMQ是一个高性能的分布式消息队列系统，被广泛运用于异步通信、解耦和削峰填谷等
> + RocketMQ采用了发布－订阅模式，消息的发送者将消息发送到消息 broker，接收者则从 broker 订阅消息。这种模式使得发送者和接收者完全解耦，可以独立演化和扩展。
>   RocketMQ提供了可靠的消息传递、顺序消息处理、事务消息等特性，确保消息的可靠性和顺序性。
>
> MQ核心组件
>
> + Producer：生产者，负责发送消息到Broker
> + Consumer：消费者，从Broker获取信息并进行消费
> + Broker：接收、存储和分发消息的中心节点（消息队列服务器）
> + Nameserver：维护broker集群和topic信息的路由中心，支持Topic、Broker 的动态注册与发现

### 服务安装配置

```shell
# 进入rocketmq主机；拷贝安装包到该机器
[root@rocketmq op]# rsync -av 192.168.1.252:/root/package/rocketmq-all-4.9.4-bin-release.zip /root/
[root@rocketmq op]# unzip /root/rocketmq-all-4.9.4-bin-release.zip 
[root@rocketmq op]# mkdir /usr/local/rocketmq
[root@rocketmq op]# mv rocketmq-all-4.9.4-bin-release/ /usr/local/rocketmq
[root@rocketmq op]# ls /usr/local/rocketmq
benchmark  bin  conf  lib  LICENSE  NOTICE  README.md
# 安装java环境
[root@rocketmq op]#dnf -y install java-1.8.0-openjdk-devel
# 配置rocketMQ服务,配置文件末尾追加
[root@rocketmq op]#vim /usr/local/rocketmq/conf/broker.conf 
namesrvAddr=0.0.0.0:9876    #指定RocketMQ Broker要连接的NameServer的地址和端口，0.0.0.0表示监听所有网络接口
brokerIP1=192.168.1.14      #指定Broker所在主机的IP地址，写自己本机eth0的ip
brokerIP2=127.0.0.1         #写自己本机127.0.0.1

# 修改JVM虚拟机参数；修改对应Xms后面的数值；根据机器配置修改，以免mq启动不起来
[root@rocketmq rocketmq]#cd /usr/local/rocketmq
[root@rocketmq rocketmq]# vim +71 bin/runserver.sh
71       JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m -Xmn256m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"

[root@rocketmq rocketmq]# vim +85 bin/runbroker.sh 
 85 JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m"

# 启动MQ服务
[root@rocketmq rocketmq]# nohup /usr/local/rocketmq/bin/mqnamesrv -n 0.0.0.0:9876 &> /usr/local/rocketmq/bin/namesrv.log &
[root@rocketmq rocketmq]# nohup /usr/local/rocketmq/bin/mqbroker -n 192.168.1.14:9876 autoCreateTopicEnable=true -c /usr/local/rocketmq/conf/broker.conf &> /usr/local/rocketmq/bin/mqbroker.log 

# 查询启动结果
[root@roocketmq op]# jps
10550 BrokerStartup
10507 NamesrvStartup
10797 Jps
[root@roocketmq op]# ss -lnt|grep 9876
LISTEN 0      1024               *:9876             *:*

#设置RocketMQ服务开机自启动
[root@rocketmq rocketmq]# vim /etc/rc.d/rc.local    #文件末尾追加
nohup /usr/local/rocketmq/bin/mqnamesrv -n 0.0.0.0:9876 &> /usr/local/rocketmq/bin/namesrv.log &
nohup /usr/local/rocketmq/bin/mqbroker -n 192.168.1.14:9876 autoCreateTopicEnable=true -c /usr/local/rocketmq/conf/broker.conf &> /usr/local/rocketmq/bin/mqbroker.log &
[root@rocketmq rocketmq]# chmod +x /etc/rc.d/rc.local
```











# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习
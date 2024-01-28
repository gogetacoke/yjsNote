- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)

# 学习目标

代码自动发布

Elasticsearch服务

RabbitMQ服务

# 课堂笔记（命令）

# 课堂笔记（文本）

## Elasticsearch部署

### 环境配置

```perl
[root@Fontend ~]# yum -y install java-1.8.0-openjdk-devel.x86_64 
[root@Services ~]# ln -s /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.332.b09-1.el8_5.x86_64/ /usr/lib/jvm/jdk   #创建JDK软链接
[root@Services ~]# vim /etc/bashrc       #配置环境变量
[root@Services ~]# tail -3 /etc/bashrc 
export JAVA_HOME="/usr/lib/jvm/jdk/"        #声明JAVA_HOME变量
export CLASSPATH=.                          #声明类库加载目录
export PATH=${JAVA_HOME}/bin/:$PATH         #声明PATH变量
[root@Services ~]# source /etc/bashrc       #刷新bash环境
[root@Services ~]# echo ${JAVA_HOME}       #测试JAVA_HOME变量
/usr/lib/jvm/jdk/
[root@Services ~]# which java                           
/usr/lib/jvm/jdk/bin/jav
```

### ES介绍

#### ES是什么

> + Elasticsearch是一个基于Lucene的搜索服务器
> + 它提供一个分布式多用户能力的全文搜索引擎，基于RESTful WEB接口
> + 实时搜索，稳定，可靠快速

#### ES能做什么

> + 企业搜索：提升任何用例的搜索和发现体验
> + 日志检测：快照且可扩展的日志管理
> + 基础架构检测：对西质保进行检测和可视化
> + Maps：实时探索位置数据

#### ES特性

![](../../pic/pro/d3-1.png)

#### ES存储方式

> Elasticsearch的数据存储方式基于Apache Lucene库，它采用了倒排索引（Inverted Index）的数据结构来存储和管理文本数据。在倒排索引中，每个词项（Term）都维护着一个包含该词项的文档ID列表，以及该文档ID出现在哪些字段上的信息。
>
> 具体地说，每个索引由多个分片（Shard）组成，每个分片是一个独立的Lucene索引，包含了一部分原始文档的数据。当文档被索引时，它会被拆分成多个词项，并将这些词项与对应的文档ID、字段信息等元数据一起存储到倒排索引中。此外，Elasticsearch还提供了副本（Replica）机制，可以将索引的数据复制到其他节点上，以提高系统的可用性和容错性。

### ES安装

> 下载相关软件包

```perl
[root@Services ~]# yum -y localinstall ./elasticsearch-6.8.0.rpm 
#配置Elasticsearch服务
[root@Services ~]# vim /etc/elasticsearch/elasticsearch.yml 
23 node.name: Services                 #ES节点名称
33 path.data: /var/lib/elasticsearch   #ES数据存储路径
37 path.logs: /var/log/elasticsearch   #ES日志存储路径
55 network.host: 0.0.0.0               #监听地址
59 http.port: 9200                     #HTTP端口
[root@services ~]# systemctl enable elasticsearch.service --now
[root@services ~]# ss -nutlp | grep java
tcp   LISTEN 0      128                *:9200            *:*    users:(("java",pid=19409,fd=209))
tcp   LISTEN 0      128                *:9300            *:*    users:(("java",pid=19409,fd=196))
```

### 测试访问

```json
# 浏览器访问
http://192.168.88.50:9200
{
  "name" : "services",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "yh_F8DHoTY6mjesY-iAqBA",
  "version" : {
    "number" : "6.8.0",
    "build_flavor" : "default",
    "build_type" : "rpm",
    "build_hash" : "65b6179",
    "build_date" : "2019-05-15T20:06:13.172855Z",
    "build_snapshot" : false,
    "lucene_version" : "7.7.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

### IK分词器

> IK分词器是一个开源的中文分词工具，专门用于将中文文本切分成一个个单独的词汇。它基于字典和规则的匹配方式，能够较好地处理中文的复杂语义和歧义问题。

#### 安装

> + 本地安装：下载好ik压缩包
>
> + 网络安装
>
>   > elasticsearch-plugin install http://addresss/xx
>   > elasticsearch-plugin install ftp://address/xx

```perl
# 本地安装
[root@services ~]# /usr/share/elasticsearch/bin/elasticsearch-plugin install file:///root/elasticsearch-analysis-ik-6.8.0.zip   
# 检查是否安装
[root@services ~]# /usr/share/elasticsearch/bin/elasticsearch-plugin list
analysis-ik
```

#### 测试

```perl
[root@Services ~]# curl -H "Content-Type: application/json" -XPOST http://localhost:9200/_analyze?pretty -d '   
{
    "analyzer": "ik_smart",
    "text": "华为手机"
}'                                                                  #测试IK分词器
{
  "tokens" : [
    {
      "token" : "华为",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "手机",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 1
    }
  ]
}
```

### HEAD插件(容器部署)

> 将容器tar包下载到本地

#### 安装容器

```perl
[root@Services ~]# yum -y install podman                    #安装podman
Complete!
[root@Services ~]# podman --version                         #确认podman安装
podman version 4.0.2
```

#### 容器导入

```perl
[root@services ~]# podman load -i elasticsearch-head.tar 
[root@services ~]#podman images
REPOSITORY                    TAG         IMAGE ID      CREATED        SIZE
localhost/elasticsearch-head  latest      d008a8ccd029  12 months ago  862 MB

[root@services ~]# podman run -d --name es-head --hostname es-head -p 9100:9100 localhost/elasticsearch-head:latest 
```

#### 修改配置

```perl
#修改Elasticsearch配置，开启跨域访问
[root@Services ~]# vim /etc/elasticsearch/elasticsearch.yml 
[root@Services ~]# sed -rn '59,61p' /etc/elasticsearch/elasticsearch.yml 
http.port: 9200
http.cors.enabled: true         #开启HTTP跨域访问支持
http.cors.allow-origin: "*"     #允许跨域的访问范围
[root@Services ~]# systemctl restart elasticsearch.service  # 启动稍满
```

> 浏览器访问：http://192.168.88.50:9100
>
> 修改loclhost为本机IP，点击连接

#### 测试API

```perl
#调用API批量导入数据
[root@Services ~]# ls data.sh logs.jsonl accounts.json 
accounts.json  data.sh  logs.jsonl
[root@Services ~]# cat data.sh 
#!/bin/bash
curl -H "Content-Type: application/json" -XPUT http://localhost:9200/account/user/_bulk --data-binary @accounts.json
curl -H "Content-Type: application/json" -XPUT http://localhost:9200/_bulk --data-binary @logs.jsonl
[root@Services ~]# bash data.sh 
```

## RabbitMQ部署

```perl
#安装Erlang
[root@Services ~]# yum clean all; yum repolist -v
[root@Services ~]# ls erlang-25.2-1.el8.x86_64.rpm 
erlang-25.2-1.el8.x86_64.rpm
[root@Services ~]# yum -y install ./erlang-25.2-1.el8.x86_64.rpm 

#安装RabbitMQ
[root@Services ~]# ls rabbitmq-server-3.11.5-1.el8.noarch.rpm 
rabbitmq-server-3.11.5-1.el8.noarch.rpm
[root@Services ~]# yum -y localinstall ./rabbitmq-server-3.11.5-1.el8.noarch.rpm 

#启动RabbitMQ服务
[root@Services ~]# systemctl enable rabbitmq-server.service #设置RabbitMQ开机自启动    
[root@Services ~]# systemctl start rabbitmq-server.service  #启动RabbitMQ服务
[root@Services ~]# ss -antpul | grep :5672                  #确认5672端口监听
tcp   LISTEN 0      128    *:5672     *:*    users:(("beam.smp",pid=13298,fd=35))

# 启用RabbitMQ网页管理插件
[root@Services ~]# rabbitmq-plugins list                        #列出所有插件
[root@Services ~]# rabbitmq-plugins enable rabbitmq_management  #启动网页管理插件
# 列出插件列表
[root@Services ~]# rabbitmq-plugins list
# 端口查询
[root@Services ~]# ss -antpul | grep :15672
tcp LISTEN 0 128   0.0.0.0:15672  0.0.0.0:*    users:(("beam.smp",pid=13298,fd=37))

#访问RabbitMQ管理页面： http://192.168.88.50:15672/
```

#### 用户创建

```perl
#RabbitMQ创建用户
[root@Services ~]# rabbitmqctl list_users           #列出RabbitMQ已有用户
Listing users ...
user    tags
guest   [administrator]
[root@Services ~]# rabbitmqctl add_user admin  hisadmin     #添加admin用户及密码
[root@Services ~]# rabbitmqctl list_users           #列出RabbitMQ已有用户
Listing users ...
user    tags
admin   []
guest   [administrator]
```

#### 用户标签管理

```perl
#RabbitMQ用户标签解析 
    #超级管理员(administrator)
        #可登陆管理控制台，可查看所有的信息，并且可以对用户，策略(policy)进行操作。
    #监控者(monitoring)
        #可登陆管理控制台，同时可以查看rabbitmq节点的相关信息(进程数，内存使用情况，磁盘使用情况等)
    #策略制定者(policymaker)
        #可登陆管理控制台, 同时可以对policy进行管理。但无法查看节点的相关信息(上图红框标识的部分)。
    #普通管理者(management)
        #仅可登陆管理控制台，无法看到节点信息，也无法对策略进行管理。
    #其他(guest)
        #无法登陆管理控制台，通常就是普通的生产者和消费者
        
#给admin用户添加administrator标签
[root@Services ~]# rabbitmqctl set_user_tags admin administrator
Setting tags for user "admin" to [administrator] ...
[root@Services ~]# rabbitmqctl list_users
Listing users ...
user    tags
admin   [administrator]
guest   [administrator]
```

#### 虚拟主机管理

```perl
#创建/his虚拟主机
[root@Services ~]# rabbitmqctl list_vhosts          #列出已有虚拟主机
Listing vhosts ...  
name
/
[root@Services ~]# rabbitmqctl add_vhost /his       #创建/his虚拟主机，后续项目使用
Adding vhost "/his" ...
[root@Services ~]# rabbitmqctl list_vhosts          #列出已有虚拟主机
Listing vhosts ...
name
/his
/
```

#### 设置访问权限

```perl
#设置admin用户对/his虚拟主机有所有权限
[root@Services ~]# rabbitmqctl list_user_permissions admin      #查看admin用户权限
Listing permissions for user "admin" ...

[root@Services ~]# rabbitmqctl set_permissions -p /his admin ".*" ".*" ".*" #设置权限
Setting permissions for user "admin" in vhost "/his" ...
[root@Services ~]# rabbitmqctl list_user_permissions admin      #查看admin用户权限
Listing permissions for user "admin" ...
vhost   configure       write   read
/his    .*              .*      .*
```

#### 工作模式

![](../../pic/pro/d3-2.png)

#### 消息确认机制

![](../../pic/pro/d3-3.png)

# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习
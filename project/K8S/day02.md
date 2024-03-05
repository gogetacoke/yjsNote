- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
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



# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习
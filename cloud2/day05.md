- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)

# 学习目标

污点与容忍策略

Pod抢占与优先级

Pod安全性

# 课堂笔记（命令）

# 课堂笔记（文本）

## 污点策略

### 污点概述

> 什么是污点？
>
> + 污点是使节点与Pod产生排斥的一类规则
>
> 污点策略是如何实现？
>
> + 污点策略通过嵌合在键值对上的污点标签进行声明
>
> 污点标签
>
> + 尽量不调度：PreferNoSchedule （尽量不运行）
> + 不会被调度：NoSchedule（不会被运行）
> + 驱逐节点：NoExecute
>
> 管理污点标签
>
> ```shell
> # 污点标签必须绑定在键值对上，格式为：
> key=value:[污点标签]
> # 查看污点标签(Taints)
> kubectl describe nodes [节点名字] 
> kubectl describe nodes [节点名字] | grep Taints
> # 删除污点标签
> kubectl taint node [节点名字] key=value:污点标签-
> ```

### 管理污点标签

```shell
# 查看污点标签
[root@master ~]# kubectl describe nodes|grep Taints
Taints:             node-role.kubernetes.io/master:NoSchedule
Taints:             <none>
Taints:             <none>
Taints:             <none>
Taints:             <none>
Taints:             <none>

# node-0001设置污点策略 PreferNoSchedule
[root@master ~]# kubectl taint node node-0001 k=v1:PreferNoSchedule
node/node-0001 tainted
# node-0002设置污点策略 NoSchedule
[root@master ~]# kubectl taint node node-0002 k=v1:NoSchedule
node/node-0002 tainted

# 查询各个节点的污点策略
[root@master ~]# kubectl describe nodes | grep -i taints
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
Taints:             k=v1:PreferNoSchedule
Taints:             k=v1:NoSchedule
Taints:             <none>
Taints:             <none>
Taints:             <none>
```

### 验证污点策略

> 通过编写资源清单文件测试在节点上的污点策略

```shell
# 编写资源文件；为每个Pod分配了1500mCPU，真实节点的CPU为2000m，所以一个Pod就占用一个node
[root@master ~]# vim myphp.yaml
---
kind: Pod
apiVersion: v1
metadata:
  name: myphp
spec:
  containers:
  - name: php
    image: myos:php-fpm
    resources:
      requests:
        cpu: 1500m  
# 按照上面的资源文件，最多只能创建5个Pod，但由于node-0001与node-0002设置了污点策略，前三个Pod一定在node-0003-0005上，第四个在node-0001，第五个将会被阻塞
[root@master 2.27]# sed "s,myphp,php1," myphp.yaml | kubectl apply -f -
pod/php1 created
[root@master 2.27]# sed "s,myphp,php2," myphp.yaml | kubectl apply -f -
pod/php2 created
[root@master 2.27]# sed "s,myphp,php3," myphp.yaml | kubectl apply -f -
pod/php3 created
[root@master 2.27]# kubectl get pods -o wide
NAME   READY   STATUS    RESTARTS   AGE   IP               NODE        NOMINATED NODE   READINESS GATES
php1   1/1     Running   0          35s   10.244.243.210   node-0003   <none>           <none>
php2   1/1     Running   0          29s   10.244.153.145   node-0005   <none>           <none>
php3   1/1     Running   0          25s   10.244.240.139   node-0004   <none>           <none>
# 创建第四个Pod；最后使用PreferNoSchedule节点
[root@master 2.27]# sed "s,myphp,php4," myphp.yaml | kubectl apply -f -
pod/php3 created
[root@master 2.27]# kubectl get pods php4 -o wide
NAME   READY   STATUS    RESTARTS   AGE   IP              NODE        NOMINATED NODE   READINESS GATES
php4   1/1     Running   0          18s   10.244.21.154   node-0001   <none>           <none>
# 创建第五个Pod；发现已经是pending；不会使用NoSchedule 节点，即使创建失败也不会使用NoSchedule所在的节点
[root@master 2.27]# sed "s,myphp,php5," myphp.yaml | kubectl apply -f -
pod/php3 created
[root@master 2.27]# kubectl get pods -o wide
NAME   READY   STATUS    RESTARTS   AGE    IP               NODE        NOMINATED NODE   READINESS GATES
php1   1/1     Running   0          2m5s   10.244.243.210   node-0003   <none>           <none>
php2   1/1     Running   0          119s   10.244.153.145   node-0005   <none>           <none>
php3   1/1     Running   0          115s   10.244.240.139   node-0004   <none>           <none>
php4   1/1     Running   0          47s    10.244.21.154    node-0001   <none>           <none>
php5   0/1     Pending   0          7s     <none>           <none>      <none>           <none>
```

### 验证已创建的Pod

> 为已经创建的Pod配置污点策略进行验证

```shell
[root@master 2.27]# kubectl describe nodes | grep -i taint
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
Taints:             k=v1:PreferNoSchedule
Taints:             k=v1:NoSchedule
Taints:             <none>
Taints:             <none>
Taints:             <none>

# 配置NoSchedule策略不会影响已经创建的Pod；只对新建Pod有效，对于已经创建完成的Pod不产生影响
[root@master 2.27]# kubectl taint node node-0003 k=v3:NoSchedule
node/node-0003 tainted
[root@master 2.27]# kubectl describe nodes | grep -i taint
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
Taints:             k=v1:PreferNoSchedule
Taints:             k=v1:NoSchedule
Taints:             k=v3:NoSchedule
Taints:             <none>
Taints:             <none>
# node-0003的pod还是running
[root@master 2.27]# kubectl get pods -o wide
NAME   READY   STATUS    RESTARTS   AGE     IP               NODE        NOMINATED NODE   READINESS GATES
php1   1/1     Running   0          6m24s   10.244.243.210   node-0003   <none>           <none>
php2   1/1     Running   0          6m18s   10.244.153.145   node-0005   <none>           <none>
php3   1/1     Running   0          6m14s   10.244.240.139   node-0004   <none>           <none>
php4   1/1     Running   0          5m6s    10.244.21.154    node-0001   <none>           <none>
php5   0/1     Pending   0          4m26s   <none>           <none>      <none>           <none>

# 配置NoExecute会删除节点上的Pod
[root@master 2.27]# kubectl taint node node-0004 k=v4:NoExecute
node/node-0004 tainted
[root@master 2.27]# kubectl describe nodes | grep -i taint
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
Taints:             k=v1:PreferNoSchedule
Taints:             k=v1:NoSchedule
Taints:             k=v3:NoSchedule
Taints:             k=v4:NoExecute
Taints:             <none>
# node-0003已经被驱逐；会删除该节点上所有的Pod
[root@master 2.27]# kubectl get pods -o wide
NAME   READY   STATUS    RESTARTS   AGE     IP               NODE        NOMINATED NODE   READINESS GATES
php1   1/1     Running   0          8m8s    10.244.243.210   node-0003   <none>           <none>
php2   1/1     Running   0          8m2s    10.244.153.145   node-0005   <none>           <none>
php4   1/1     Running   0          6m50s   10.244.21.154    node-0001   <none>           <none>
php5   0/1     Pending   0          6m10s   <none>           <none>      <none>           <none>

```

### 清理环境配置

```shell
# 删除所有污点策略
[root@master 2.27]# kubectl delete pods --all
pod "php1" deleted
pod "php2" deleted
pod "php4" deleted
pod "php5" deleted
[root@master ~]# kubectl taint node node-000{1..4} k-
node/node-0001 untainted
node/node-0002 untainted
node/node-0003 untainted
node/node-0004 untainted
[root@master 2.27]# kubectl describe nodes | grep -i taint
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
Taints:             <none>
Taints:             <none>
Taints:             <none>
Taints:             <none>
Taints:             <none>
```

## 容忍策略

### 容忍概述

> 容忍策略是什么？
>
> + 容忍刚好与污点相反，某些时候我们需要在有污点的节点上运行Pod，这种无视污点标签的调度方式称为容忍
>
> 容忍策略：
>
> + Equal  精准匹配
> + Exists 部分匹配
>
> 格式
>
> ```shell
> spec:
>   tolerations:
>   - operator: 容忍策略
>     key: 键
>     value: 值
>     effect: 污点标签
> ```

### 设置污点标签

```shell
# 节点 node-0001,node-0002 设置污点标签 k=v1:NoSchedule
[root@master ~]# kubectl taint node node-000{1..2} k=v1:NoSchedule
node/node-0001 tainted
node/node-0002 tainted
# 节点 node-0003,node-0004 设置污点标签 k=v2:NoSchedule
[root@master ~]# kubectl taint node node-000{3..4} k=v2:NoSchedule
node/node-0003 tainted
node/node-0004 tainted
# 节点 node-0005 设置污点标签 k=v1:NoExecute
[root@master ~]# kubectl taint node node-0005 k=v1:NoExecute
node/node-0005 tainted

[root@master 2.27]# kubectl describe nodes | grep -i taints
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
Taints:             k=v1:NoSchedule
Taints:             k=v1:NoSchedule
Taints:             k=v2:NoSchedule
Taints:             k=v2:NoSchedule
Taints:             k=v1:NoExecute
```

### 精确匹配策略

> Equal

```shell
# 容忍k=v:NoSchedule污点
[root@master 2.27]# vim myphp.yaml 
---  
kind: Pod
apiVersion: v1
metadata:
  name: myphp
spec:
  tolerations:
  - operator: Equal # 完全匹配键值对
    key: k # 键
    value: v1 # 值
    effect: NoSchedule # 污点标签
  containers:
    - name: php
      image: myos:php-fpm
      resources:
        requests:
          cpu: 1500m
# 创建Pod进行测试
[root@master ~]# for i in php{1..3};do sed "s,myphp,${i}," myphp.yaml ;done|kubectl apply -f -
pod/php1 created
pod/php2 created
pod/php3 created
[root@master 2.27]# kubectl get pods -o wide
NAME   READY   STATUS    RESTARTS   AGE   IP   NODE  NOMINATED NODE  READINESS GATES
php1   1/1     Running   0          26s   10.244.147.18   node-0002   <none>           <none>
php2   1/1     Running   0          11s   10.244.21.156   node-0001   <none>           <none>
php3   0/1     Pending   0          1s    <none>          <none>      <none>           <none>

[root@master ~]# kubectl delete pod --all
pod "php1" deleted
pod "php2" deleted
pod "php3" deleted
```

### 模糊匹配策略

> Exists
>
> 适合于键值对不同，标签相同

```shell
# 修改资源对象文件；容忍k=*:NoSchedule污点
[root@master 2.27]# vim myphp.yaml 
---
kind: Pod
apiVersion: v1
metadata:
  name: myphp
spec:
  tolerations:
  - operator: Exists # 部分匹配
    key: k
    effect: NoSchedule
  containers:
    - name: php
      image: myos:php-fpm
      resources:
        requests:
          cpu: 1500m
          
[root@master 2.27]# for i in php{1..5};do sed "s,myphp,${i}," myphp.yaml ;done | kubectl apply -f -
pod/php1 created
pod/php2 created
pod/php3 created
pod/php4 created
pod/php5 created
# 只有node0001-0004是running
[root@master 2.27]# kubectl get pods -o wide
NAME   READY   STATUS    RESTARTS   AGE   IP               NODE        NOMINATED NODE   READINESS GATES
php1   1/1     Running   0          9s    10.244.21.157    node-0001   <none>           <none>
php2   1/1     Running   0          9s    10.244.243.212   node-0003   <none>           <none>
php3   1/1     Running   0          9s    10.244.147.19    node-0002   <none>           <none>
php4   1/1     Running   0          9s    10.244.240.140   node-0004   <none>           <none>
php5   0/1     Pending   0          9s    <none>           <none>      <none>           <none>   

[root@master 2.27]# kubectl delete pods --all
```

### 容忍所有污点

> k=\*:\*

```shell
# 修改资源对象文件
[root@master 2.27]# vim myphp.yaml 
---
kind: Pod
apiVersion: v1
metadata:
  name: myphp
spec:
  tolerations:
  - operator: Exists
    key: k
    effect: "" # 设置空或删除，代表所有污点标签
  containers:
    - name: php
      image: myos:php-fpm
      resources:
        requests:
          cpu: 1500m

[root@master 2.27]# for i in php{1..5};do sed "s,myphp,${i}," myphp.yaml ;done | kubectl apply -f -
pod/php1 created
pod/php2 created
pod/php3 created
pod/php4 created
pod/php5 created
[root@master 2.27]# kubectl get pods -o wide
NAME   READY   STATUS    RESTARTS   AGE   IP               NODE        NOMINATED NODE   READINESS GATES
php1   1/1     Running   0          5s    10.244.147.21    node-0002   <none>           <none>
php2   1/1     Running   0          5s    10.244.243.214   node-0003   <none>           <none>
php3   1/1     Running   0          5s    10.244.240.142   node-0004   <none>           <none>
php4   1/1     Running   0          5s    10.244.153.148   node-0005   <none>           <none>
php5   1/1     Running   0          5s    10.244.21.159    node-0001   <none>           <none>
[root@master 2.17]# kubectl delete pods --all

# 清理策略
[root@master 2.17]# kubectl taint node node-000{1..5} k-
node/node-0001 untainted
node/node-0002 untainted
node/node-0003 untainted
node/node-0004 untainted
node/node-0005 untainted
[root@master 2.17]# kubectl describe nodes | grep -i taints
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
Taints:             <none>
Taints:             <none>
Taints:             <none>
Taints:             <none>
Taints:             <none>
```

### 注意

> 在配置容忍策略时，至少写上一个key，不能只有个容忍策略key、value、标签什么都没有，切忌不行，这样将有可能在master上创建，会破坏master，导致无法控制整个pod集群

## 优先级与抢占

### 优先级概述

> 优先级是什么？
>
> + 优先级表示一个Pod相对于其他的Pod的重要性
>
> 优先级的作用
>
> + 优先级可以保证重要的Pod被调度运行
>
> 如何使用优先级和抢占
>
> + 配置优先级类PriorityClass
> + 创建Pod时为其设置对应的优先级
>
> PriorityClass
>
> + 是一个全局资源对象，它定义了从优先级类名称到优先级数值的映射。优先级在value字段中指定，可以设置小于10亿的整数值，值越大，优先级越高。
>
> PrioityClass还有两个可选字段
>
> + globalDefault用于设置默认优先级状态，如果没有任何优先级设置Pod的优先级为零
> + description用来配置描述性信息告诉用户优先级的用途
>
> 优先级策略
>
> + 非抢占优先：指的是在调度阶段优先进行调度匹配，一旦容器调度完成就不可以抢占，资源不足时，只能等待
> + 非抢占优先：强制调度一个Pod，如果资源不足无法被调度，调度程序会抢占（删除）较低优先级的Pod的资源，来保证高优先级Pod的运行

### 非抢占优先级配置

> 非抢占策略：Never

```shell
# 定义非抢占优先级(队列优先)
[root@master 2.17]# kubectl api-resources | grep -i priority
prioritylevelconfigurations                    flowcontrol.apiserver.k8s.io/v1beta3   false        PriorityLevelConfiguration
priorityclasses                   pc           scheduling.k8s.io/v1                   false        PriorityClass
[root@master 2.27~]# vim mypriority.yaml
---
kind: PriorityClass
apiVersion: scheduling.k8s.io/v1
metadata:
  name: high-non  # 优先级名称
preemptionPolicy: Never # 策略：非抢占
value: 1000 # 优先级值

---
kind: PriorityClass
apiVersion: scheduling.k8s.io/v1
metadata:
  name: low-non
preemptionPolicy: Never
value: 500
[root@master 2.27~]#  kubectl apply -f mypriority.yaml 
priorityclass.scheduling.k8s.io/high-non created
priorityclass.scheduling.k8s.io/low-non created
[root@master 2.27~]#  kubectl get priorityclasses.scheduling.k8s.io 
NAME                      VALUE        GLOBAL-DEFAULT   AGE
high-non                  1000         false            12s
low-non                   500          false            12s
system-cluster-critical   2000000000   false            45h
system-node-critical      2000001000   false            45h
```

### 创建测试资源文件

> 优先级属性：priorityClassName

```shell
# 无优先级的 Pod
[root@master 2.27~]#  vim php1.yaml 
---
kind: Pod
apiVersion: v1
metadata:
  name: php1
spec:
  nodeSelector:
    kubernetes.io/hostname: node-0004
  containers:
  - name: php
    image: myos:php-fpm
    resources:
      requests:
        cpu: "1500m"
# 低优先级 Pod
[root@master 2.27~]#  vim php2.yaml 
---
kind: Pod
apiVersion: v1
metadata:
  name: php2
spec:
  nodeSelector:
    kubernetes.io/hostname: node-0004
  priorityClassName: low-non      # 指定优先级名称：低优先级
  containers:
  - name: php
    image: myos:php-fpm
    resources:
      requests:
        cpu: "1500m"

# 高优先级 Pod
[root@master 2.27~]#  vim php3.yaml 
---
kind: Pod
apiVersion: v1
metadata:
  name: php3
spec:
  nodeSelector:
    kubernetes.io/hostname: node-0004
  priorityClassName: high-non     # 指定优先级名称：高优先级
  containers:
  - name: php
    image: myos:php-fpm
    resources:
      requests:
        cpu: "1500m"        
```

### 验证非抢占优先级

> 开启两个终端方便查询整个过程
>
> kubectl get pods -w 实时查看  

```shell
[root@master 2.27~]#  kubectl apply -f php1.yaml 
pod/php1 created
[root@master 2.27~]#  kubectl apply -f php2.yaml 
pod/php2 created
[root@master 2.27~]#  kubectl apply -f php3.yaml 
pod/php3 created
[root@master 2.27~]#  kubectl get pods
NAME   READY   STATUS    RESTARTS   AGE
php1   1/1     Running   0          9s
php2   0/1     Pending   0          6s
php3   0/1     Pending   0          4s
[root@master 2.27~]#  kubectl delete pod php1
pod "php1" deleted
[root@master 2.27~]#  kubectl get pods
NAME   READY   STATUS    RESTARTS   AGE
php2   0/1     Pending   0          20s
php3   1/1     Running   0          18s

# 清理实验 Pod
[root@master 2.27~]#  kubectl delete pod php2 php3
pod "php2" deleted
pod "php3" deleted
```

> 非抢占优先级是一种调度策略，它允许管理员设置特定Pod的优先级级别，使得较高优先级的Pod不会因为资源紧张而被较低优先级的Pod抢占，即使高优先级Pod需要更多资源也不会影响已运行的低优先级Pod。

### 抢占策略

> 抢占策略：PreemptLowerPriority 

```shell
# 在非抢占优先级配置文件添加两条
[root@master 2.27~]#  vim mypriority.yaml
.....
---
kind: PriorityClass
apiVersion: scheduling.k8s.io/v1
metadata:
  name: high
preemptionPolicy: PreemptLowerPriority
value: 1000

---
kind: PriorityClass
apiVersion: scheduling.k8s.io/v1
metadata:
  name: low
preemptionPolicy: PreemptLowerPriority
value: 500

# 刷新配置
[root@master 2.27~]#  kubectl apply -f mypriority.yaml 
priorityclass.scheduling.k8s.io/high created
priorityclass.scheduling.k8s.io/low created
[root@master 2.27~]#  kubectl get priorityclasses.scheduling.k8s.io 
NAME                      VALUE        GLOBAL-DEFAULT   AGE
high                      1000         false            25s
high-non                  1000         false            153m
low                       500          false            25s
low-non                   500          false            153m
system-cluster-critical   2000000000   false            6d
system-node-critical      2000001000   false            6d
```

### 验证抢占优先级

> 开启两个终端
>
> kubectl get pods -w

```shell
# 替换优先级策略
[root@master 2.27~]#  sed 's,-non,,' -i php?.yaml

# 默认优先级 Pod
[root@master 2.27~]#  kubectl apply -f php1.yaml 
pod/php1 created
[root@master 2.27~]#  kubectl get pods
NAME   READY   STATUS    RESTARTS   AGE
php1   1/1     Running   0          6s

# 高优先级 Pod
[root@master 2.27~]#  kubectl apply -f php3.yaml
pod/php3 created
[root@master 2.27~]#  kubectl get pods
NAME   READY   STATUS    RESTARTS   AGE
php3   1/1     Running   0          9s


# 低优先级 Pod
[root@master 2.27~]#  kubectl apply -f php2.yaml
pod/php2 created
[root@master 2.27~]#  kubectl get pods
NAME   READY   STATUS    RESTARTS   AGE
php2   0/1     Pending   0          3s
php3   1/1     Running   0          9s
```

> 抢占优先级是指一种调度策略，当集群资源紧张时，高优先级的Pod能够抢占（即强制终止）低优先级的Pod，以便为自己分配所需资源并得以运行。

## Pod安全

### 特权容器概述

> 什么是特权容器？
>
> + 容器是通过名称空间技术隔离的，有时候我们执行一些应用程序，需要使用或修改敏感的系统信息，这时容器需要突破隔离限制，获取更高的权限，这类容器统称特权容器。
>
> + 运行特权容器会有一些安全风险，这种模式下运行容器对宿主机拥有 root 访问权限，可以突破隔离直接控制宿主机的资源配置。

### 特权容器1

> 更改容器主机名 和 /etc/hosts 文件
>
> 前提：当创建的容器发生故障时，系统会重新构建，原先容器里面的数据将会被清除；需要特权容器来保证前提数据

```shell
[root@master 2.27~]#  vim root.yaml
---
kind: Pod
apiVersion: v1
metadata:
  name: root
spec:
  hostname: myhost         # 修改主机名
  hostAliases:             # 修改 /etc/hosts
  - ip: 192.168.1.30       # IP 地址
    hostnames:             # 名称键值对
    - harbor               # 主机名
  containers:
  - name: apache
    image: myos:httpd

[root@master 2.27~]# kubectl apply -f root.yaml 
pod/root created
[root@master 2.27~]#  kubectl exec -it root -- /bin/bash
[root@myhost html]# hostname
myhost
[root@myhost html]# cat /etc/hosts
... ...
# Entries added by HostAliases.
192.168.1.30    harbor

[root@master 2.27~]#  kubectl delete pod root 
pod "root" deleted
```

### 特权容器2

> 共享宿主机的进程信息
>
>  hostPID: true 
>
> 不推荐

```shell
[root@master 2.27~]#  vim root.yaml 
---
kind: Pod
apiVersion: v1
metadata:
  name: root
spec:
  hostPID：true  # 共享系统进程
  containers:
  - name: apache
    image: myos:httpd
[root@master 2.27~]#  kubectl apply -f root.yaml 
[root@master 2.27~]#  kubectl exec -it root -- /bin/bash
[root@root html]# ps -ef 
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 00:58 ?        00:00:08 /usr/lib/systemd/systemd -
root           2       0  0 00:58 ?        00:00:00 [kthreadd]
root           3       2  0 00:58 ?        00:00:00 [rcu_gp]
root           4       2  0 00:58 ?        00:00:00 [rcu_par_gp]
root           5       2  0 00:58 ?        00:00:00 [slub_flushwq]
root           7       2  0 00:58 ?        00:00:00 [kworker/0:0H-events_highp

[root@root html]# exit
exit
[root@master 2.27~]# kubectl delete pods --all
pod "root" deleted
```

> [注]：设置后干扰宿主机，不建议这样做，因为在容器内就可以kil宿主机的进程

### 特权容器3

> 共享主机网络
>
> hostNetwork: true 

```shell
[root@master 2.27~]# vim root.yaml 
---
kind: Pod
apiVersion: v1
metadata:
  name: root
spec:
  hostPID：true  # 共享系统进程
  hostNetwork: true # 共享网络
  containers:
  - name: apache
    image: myos:httpd
[root@master 2.27~]#  kubectl apply -f root.yaml
[root@master 2.27~]#  kubectl exce -it -- /bin/bash
[root@master 2.27~]#  kubectl exec -it root -- /bin/bash
# 进入容器直接变成宿主机了
[root@node-0004 html]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.54  netmask 255.255.255.0  broadcast 192.168.1.255
[root@node-0004 html]#exit 
[root@master 2.27~]#  curl 192.168.1.54
Welcome to The Apache.        
```

> 未形成网络隔离

### 特权容器4

> privileged: true  最高特权

```shell
[root@master 2.27~]# vim root.yaml
---
kind: Pod
apiVersion: v1
metadata:
  name: root
spec:
  hostPID: true            
  hostNetwork: true        
  containers:
  - name: apache
    image: myos:httpd
    securityContext:       # 安全上下文值
      privileged: true     # root特权容器

# root用户特权
[root@node-0001 /]# mkdir /sysroot
[root@node-0001 /]# mount /dev/vda1 /sysroot
[root@node-0001 /]# mount -t proc proc /sysroot/proc
[root@node-0001 /]# chroot /sysroot
sh-4.2# : 此处已经是 node 节点上的 root 用户了

# 删除特权容器
[root@master 2.27~]#  kubectl delete pod root 
pod "root" deleted
```

### Pod安全概述

> 什么是 Pod 安全策略？ 
>
> + Pod 安全策略是集群级别的资源，它能够控制 Pod 运行的行为，以及它具有访问什么的能力。 
>
> 如何使用 Pod 安全策略？
>
> + Kubernetes 服务器版本必须不低于版本 v1. 22 –确保 PodSecurity 特性门控被启用
>
> Pod 安全策略
>
> + PodSecurity 提供一种内置的 Pod 安全性准入控制器， 作为 PodSecurityPolicies 特性的后续演化版本。Pod 安全性限制是在 Pod 被创建时，在名字空间层面实施的。
> + Pod 安全性标准定义了三种不同的策略（Policy），以广泛覆盖安全应用场景。 这些策略是渐进式的，安全级别从高度宽松至高度受限。
>
> 
>
> Pod 安全策略（LEVEL）
>
> + \_privileged：不受限制的策略，提供最大可能范围的权限许可。此策略允许特权
> + \_baseline：弱限制性的策略，禁止已知的策略提升权限。允许使用默认的 Pod 配置。
> + \_restricted：非常严格的限制性策略，遵循当前的保护 Pod 的最佳实践。
>
> 
>
> Pod 准入控制标签（MODE）
>
> + Kubernetes 定义了一组标签，你可以设置这些标签来定义某个名字空间上 Pod 安全性标准级别。你所选择的标签定义了检测到潜在违规时，所要采取的动作。
> +  enforce：策略违规会导致 Pod 被拒绝 
> + audit：策略违规会触发审计日志，但是 Pod 仍可被接受 
> + warn：策略违规会触发用户可见的警告信息，但是 Pod 仍是被接受的 pod-security.kubernetes.io/:\<MODE\>:\<LEVEL\>

### Pod安全策略

```shell
# 限制创建特权容器

# 生产环境设置严格的准入控制
[root@master 2.27~]#  kubectl create namespace myprod
namespace/myprod created
[root@master 2.27~]#  kubectl label namespaces myprod pod-security.kubernetes.io/enforce=restricted
namespace/myprod labeled

# 测试环境测试警告提示
[root@master 2.27~]# kubectl create namespace mytest
namespace/mytest created
[root@master 2.27~]#  kubectl label namespaces mytest pod-security.kubernetes.io/warn=baseline
namespace/mytest labeled


# 创建特权容器
[root@master 2.27~]# kubectl -n myprod apply -f root.yaml 
Error from server (Failure): error when creating "root.yaml": host namespaces (hostNetwork=true, hostPID=true), privileged (container "linux" must not set securityContext.privileged=true), allowPrivilegeEscalation != false (container "linux" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "linux" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "linux" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "linux" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
[root@master 2.27~]# 
[root@master 2.27~]# kubectl -n myprod get pods
No resources found in myprod namespace.

[root@master ~]# kubectl -n mytest apply -f root.yaml                                    
Warning: would violate "latest" version of "baseline" PodSecurity profile: host namespaces (hostNetwork=true, hostPID=true), privileged (container "linux" must not set securityContext.privileged=true)
pod/root created
[root@master ~]# 
[root@master ~]# kubectl -n mytest get pods               
NAME   READY   STATUS    RESTARTS   AGE
root   1/1     Running   0          7s

```

### 安全的Pod

```shell
[root@master ~]# vim nonroot.yaml
---
kind: Pod
apiVersion: v1
metadata:
  name: nonroot
spec:
  restartPolicy: Always
  containers:
  - name: php
    image: myos:php-fpm
    securityContext:
      allowPrivilegeEscalation: false
      runAsNonRoot: true
      runAsUser: 65534
      seccompProfile:
        type: "RuntimeDefault"
      capabilities:
        drop: ["ALL"]

[root@master ~]# kubectl -n myprod apply -f nonroot.yaml 
pod/nonroot created
[root@master ~]# kubectl -n myprod get pods
NAME      READY   STATUS    RESTARTS   AGE
nonroot   1/1     Running   0          6s
[root@master ~]# kubectl -n myprod exec -it nonroot -- id
uid=65534(nobody) gid=65534(nobody) groups=65534(nobody)
```



# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习
- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)

# 学习目标

POD调度策略

Pod标签管理

Pod资源配额与监控

# 课堂笔记（命令）

# 课堂笔记（文本）

## Pod调度策略

### 概述

#### 什么是调度分配

> + 在 k8s 中，调度是将 Pod 分配到合适的计算节点上，然后对应节点上的 Kubelet 运行这些 pod – kube-scheduler 是默认调度器，是集群的核心组件 

#### 调度器是如何工作

> + 调度器通过 k8s 的监测（Watch）机制来发现集群中尚未被调度到节点上的 Pod。调度器依据调度原则将 Pod 分配到一个合适的节点上运行

####  调度流程

> 基本流程：过滤-打分

##### 第一步过滤

> + 首先要筛选出满足 Pod 所有的资源请求的节点，这里包含计算资源、内存、存储、网络、端口号等等，如果没有节点能满足 Pod 的需求，Pod 将一直停留在 Pending 状态，直到调度器能够找到合适的节点运行它

##### 第二步打分

> + 在打分阶段，调度器会根据打分规则，为每一个可调度节点进行打分。选出其中得分最高的节点来运行 Pod。如果存在多个得分最高的节点，调度器会从中随机选取一个。

##### 绑定

> + 在确定了某个节点运行 Pod 之后，调度器将这个调度决定通知给 kube-apiserver， 这个过程叫做绑定。

### 基于节点的调度

> 通过节点名称进行调度

```shell
[root@master ~]# vim myhttp.yaml
---
kind: Pod
apiVersion: v1
metadata:
  name: myhttpd
spec:
  nodeName: node-0001 # 基于节点名进行调度
  containers:
  - name: apache
    image: myos:httpd
[root@master ~]# kubectl apply -f myhttp.yaml 
[root@master ~]# kubectl get pods myhttpd -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP              NODE        NOMINATED NODE   READINESS GATES
myhttpd   1/1     Running   0          5s    10.244.21.141   node-0001   <none>           <none>
# 删除后重新创建
[root@master ~]# kubectl delete pods myhttpd
pod "myhttpd" deleted
[root@master ~]# kubectl apply -f myhttp.yaml 
[root@master ~]# kubectl get pods myhttpd -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP              NODE        NOMINATED NODE   READINESS GATES
myhttpd   1/1     Running   0          5s    10.244.21.141   node-0001   <none>           <none>
```

#### 定向调度弊端

> + 灵活性受限
>
>   如果Pod配置中指定了具体的nodeName，那么Pod将只能运行在该名称的节点上。当该节点不可用（如维护、故障）时，Pod将无法调度到其他可用节点，从而导致服务中断
>
> + 资源利用率不均
>
>   直接指定节点可能导致集群资源分配不均衡，尤其是在动态变化的环境中，可能造成部分节点负载过高，而其他节点空闲的情况。
>
> + 扩展性和容错性降低
>
>   针对性的调度策略使得在添加新节点或者节点出现故障时，系统无法自动调整Pod的位置以适应新的集群状态，降低了系统的自我恢复能力。

### 标签管理

#### 概述

标签是什么

> 标签（Labels）是附加到 Kubernetes 对象上的键值对

标签的用途

> - k8s 在创建、删除、修改资源对象的时候可以使用标签来确定要修改的资源对象。在 Pod 调度的任务中，使用标签可以更加灵活的完成调度任务。
> - 标签可以在创建时附加到对象，也可以在创建之后随时添加和修改。标签可以用于组织和选择对象的子集。

#### 语法格式

```shell
# 设置标签
kubectl label 资源类型 [资源名称] <key>=<value>
# 删除标签
kubectl label 资源类型 [资源名称] <key>-
# 查看标签
kubectl label 资源类型 [资源名称] --show-labels
# 标签过滤
kubectl label 资源类型 [资源名称] -l <key>=<value>
```

#### 查询标签

```shell
# 查看pod标签
[root@master ~]# kubectl get pod myhttpd --show-labels 
NAME      READY   STATUS    RESTARTS   AGE    LABELS
myhttpd   1/1     Running   0          5m5s   port=80
# 查看节点标签
[root@master ~]# kubectl get nodes --show-labels 
NAME        STATUS   ROLES           AGE     VERSION   LABELS
master      Ready    control-plane   4d19h   v1.26.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=master,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
node-0001   Ready    <none>          4d17h   v1.26.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node-0001,kubernetes.io/os=linux
node-0002   Ready    <none>          4d17h   v1.26.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node-0002,kubernetes.io/os=linux
node-0003   Ready    <none>          4d17h   v1.26.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node-0003,kubernetes.io/os=linux
node-0004   Ready    <none>          4d17h   v1.26.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node-0004,kubernetes.io/os=linux
node-0005   Ready    <none>          4d17h   v1.26.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node-0005,kubernetes.io/os=linux
# 查看名称空间标签
[root@master ~]# kubectl get namespace --show-labels 
NAME              STATUS   AGE     LABELS
default           Active   4d19h   kubernetes.io/metadata.name=default
kube-node-lease   Active   4d19h   kubernetes.io/metadata.name=kube-node-lease
kube-public       Active   4d19h   kubernetes.io/metadata.name=kube-public
kube-system       Active   4d19h   kubernetes.io/metadata.name=kube-system
```

#### 标签过滤

```shell
# 过滤查询标签web=http的pod
[root@master ~]# kubectl get pods -l web=http --show-labels
# 过滤查询标签web=http,port=80的pod
[root@master ~]# kubectl get pods -l web=http,port=80 --show-labels
```

#### 添加标签

```shell
# 添加单个标签
[root@master ~]# kubectl label pod myhttp app=apache
pod/myhttp labeled
[root@master ~]# kubectl get pods --show-labels 
NAME      READY   STATUS    RESTARTS   AGE   LABELS
myhttp    1/1     Running   0          14m   app=apache
# 追加或添加多个标签
[root@master ~]# kubectl label pod myhttp ports=80,name=test
pod/myhttp labeled
[root@master ~]# kubectl get pods --show-labels 
NAME      READY   STATUS    RESTARTS   AGE   LABELS
myhttp    1/1     Running   0          14m   app=apache,ports=80,name=test
```

#### 删除标签

```shell
[root@master ~]# kubectl label pod myhttp app-
pod/myhttp labeled
[root@master ~]# kubectl get pods --show-labels 
NAME      READY   STATUS    RESTARTS   AGE   LABELS
myhttp    1/1     Running   0          14m   ports=80,name=test
```

#### 资源文件标签

```shell
[root@master ~]# vim myhttp.yaml 
---
kind: Pod
apiVersion: v1
metadata:
  name: myhttp
  labels:               # 声明标签
    web: apache         # 标签键值对
spec:
  containers:
  - name: apache
    image: myos:httpd

[root@master ~]# kubectl delete pods myhttp
pod "myhttp" deleted
[root@master ~]# kubectl apply -f myhttp.yaml  
pod/myhttp created
[root@master ~]# kubectl get pods --show-labels 
NAME      READY   STATUS    RESTARTS   AGE   LABELS
myhttp    1/1     Running   0          14m   web=apache
```

### Pod标签调度

#### 概述

> 标签选择运算符
>
> + 与名称和 UID 不同，标签不支持唯一性。通常，我们希望许多对象携带相同的标签。
> + 通过标签选择算符，客户端/用户可以识别一组对象。
> + 标签选择算符可以由多个需求组成。在多个需求的情况下，必须满足所有要求，相当于逻辑与（&&）运算符。
>
> ```shell
> # 格式
> .....
> spec:
>   nodeSelector: # 在pod.spec中声明标签选择的运算符
>     key: value  # 选择的标签
> ```

#### 基于标签调度

```shell
# 查询节点标签
[root@master ~]# kubectl get nodes --show-labels 
NAME        STATUS   ROLES           AGE     VERSION   LABELS
master      Ready    control-plane   4d20h   v1.26.0   
.....
[root@master ~]# vim myhttp.yaml
---  
kind: Pod
apiVersion: v1
metadata:
  name: web1
  labels:
    web: httpd
    ports: "8080"
spec:
  nodeSelector:  # 基于节点标签进行调度
    kubernetes.io/hostname: node-0002 # 节点标签
  containers:
  - name: apache
    image: myos:httpd
~       
[root@master ~]# kubectl apply -f myhttp.yaml
[root@master ~]# kubectl get pods -o wide
NAME   READY   STATUS    RESTARTS   AGE     IP             NODE        NOMINATED NODE   READINESS GATES
web1   1/1     Running   0          2m35s   10.244.147.7   node-0002   <none>           <none>
```

#### 案例

> 已知节点node-0001和0002使用的是ssd硬盘，创建5个pod[pod1，pod2，pod3，pod4，pod5]这些pod必须运行在ssd硬盘的节点上，完成后删除配置

```shell
# 为节点打上标签
[root@master ~]#kubectl label nodes node-0001 node-0002 disktype=ssd
# 编写资源配置文件
[root@master ~]#vim myhttp.yaml
---
kind: Pod
apiVersion: v1
metadata:
  name: myhttp
  labels:
    web: apache
spec:
  nodeSelector:
    disktype: ssd
  containers:
  - name: apache
    image: myos:httpd
# 使用循环进行创建5个pod

```

## Pod资源配额

### 概述

> 为什么要资源配额     
>
> + 当多个应用共享固定节点数目的集群时，人们担心某些应用无法获得足够的资源，从而影响到其正常运行，我们需要设定一些规则，用来保证应用能获得其运行所需资源 
>
> CPU资源类型
>
> +  CPU  资源的约束和请求以毫核（m）为单位。在  k8s  中  1m  是最小的调度单元，CPU  的一个核心可以看作1000m     
> + 如果你有  2  颗  CPU，且每  CPU  为  4  核心，那么你的  CPU  资源总量就是  8000m

### 资源对象文件

> 测试：当个Pod将把资源沾满

```shell
# 编写测试对象文件
[root@master ~]# vim minpod.yaml 
---
kind: Pod
apiVersion: v1
metadata:
  name: minpod
spec:
  terminationGracePeriodSeconds: 0
  containers:
  - name: linux
    image: myos:8.5
    command: ["awk", "BEGIN{while(1){}}"]
# 运行测试
[root@master ~]# kubectl top pods
NAME     CPU(cores)   MEMORY(bytes)   
minpod   997m         0Mi 
```

### 内存资源配额

> 通过requests.memory：来表示容器至少需要保证由多少MiB的内存可用

```shell
[root@master ~]# vim minpod.yaml 
---
kind: Pod
apiVersion: v1
metadata:
  name: minpod
spec:
  terminationGracePeriodSeconds: 0
  nodeSelector:                        # 配置 Pod 调度节点
    kubernetes.io/hostname: node-0003  # 在 node-0003 节点创建
  containers:
  - name: linux
    image: myos:8.5
    command: ["awk", "BEGIN{while(1){}}"] # 死循环，耗占CPU
    resources:               # 资源策略
      requests:              # 配额策略
        memory: 1100Mi       # 内存配额

# 验证配额策略
[root@master ~]# for i in app{1..5};do sed "s,minpod,${i}," minpod.yaml;done |kubectl apply -f -
pod/app1 created
pod/app2 created
pod/app3 created
pod/app4 created
pod/app5 created

[root@master ~]# kubectl get pods
NAME   READY   STATUS    RESTARTS   AGE
app1   1/1     Running   0          4s
app2   1/1     Running   0          4s
app3   1/1     Running   0          4s
app4   0/1     Pending   0          4s
app5   0/1     Pending   0          4s

# 清理实验配置
[root@master ~]# kubectl delete pod --all
```

> 总结：为什么创建5个pod只有三个pod时running呢？
>
> 1. 由于minpod.yaml 中配置了调度节点为node-0003，那么5个pod都将调度到该节点上
> 2. node-0003的资源情况为：2cpu4G
> 3. 根据minpod.yaml中的内存配额为1100Mi，4Gi/1100Mi=3.64；所以只能将内存分配给三台主机
> 4. 根据内存配额加上机器总的资源得出结论，所以只能有三个pod是running的

### 计算资源配额

```shell
[root@master ~]# vim minpod.yaml 
---
kind: Pod
apiVersion: v1
metadata:
  name: minpod
spec:
  terminationGracePeriodSeconds: 0
  nodeSelector:
    kubernetes.io/hostname: node-0003
  containers:
  - name: linux
    image: myos:8.5
    command: ["awk", "BEGIN{while(1){}}"]
    resources:
      requests:
        cpu: 800m          # 计算资源配额

# 验证配额策略
[root@master ~]# for i in app{1..5};do sed "s,minpod,${i}," minpod.yaml;done |kubectl apply -f -
pod/app1 created
pod/app2 created
pod/app3 created
pod/app4 created
pod/app5 created
[root@master ~]# kubectl get pods
NAME   READY   STATUS    RESTARTS   AGE
app1   1/1     Running   0          8s
app2   1/1     Running   0          8s
app3   0/1     Pending   0          8s
app4   0/1     Pending   0          8s
app5   0/1     Pending   0          8s

# 清理实验配置
[root@master ~]# kubectl delete pod --all
```

> 同内存资源配额一样的原理，node-0003的机器信息为2cpu4G
>
> 2cpu=2000m
>
> 只能创建2个pod

### 综合资源配额

```shell
[root@master ~]# vim minpod.yaml 
---
kind: Pod
apiVersion: v1
metadata:
  name: minpod
spec:
  terminationGracePeriodSeconds: 0
  nodeSelector:
    kubernetes.io/hostname: node-0003
  containers:
  - name: linux
    image: myos:8.5
    command: ["awk", "BEGIN{while(1){}}"]
    resources:
      requests:
        cpu: 800m          # 计算资源配额
        memory: 1100Mi     # 内存资源配额       
```

## Pod资源限额

### 概述

> 为什么要使用资源限额？
>
> + 限额策略是为了防止某些应用对节点资源过度使用，而配置的限制性策略，限额与配额相反，它不检查节点资源的剩余情况，只限制应用对资源的最大使用量
> + 资源限额使用 limits  进行配置

### 限制内存CPU

```shell
# 创建限额资源对象文件
[root@master ~]# vim maxpod.yaml 
---
kind: Pod
apiVersion: v1
metadata:
  name: maxpod
spec:
  terminationGracePeriodSeconds: 0
  containers:
  - name: linux
    image: myos:8.5
    command: ["awk", "BEGIN{while(1){}}"]
    resources:
      limits:
        cpu: 800m
        memory: 2000Mi

[root@master ~]# kubectl apply -f maxpod.yaml 
pod/maxpod created
```

### 验证内存限额

> 通过脚本文件来验证

```shell
[root@master ~]# kubectl cp memtest.py maxpod:/usr/bin/
[root@master ~]# kubectl exec -it maxpod -- /bin/bash
[root@maxpod /]# memtest.py 2500
Killed
[root@maxpod /]# memtest.py 1950
use memory success
press any key to exit : 
[root@maxpod /]# memtest.py 2000
Killed
```

### 验证CPU限额

> 根据编写资源对象文件中的死循环命令来验证

```shell
[root@master ~]# kubectl top pods
NAME     CPU(cores)   MEMORY(bytes)   
maxpod   800m         0Mi             
[root@master ~]# kubectl exec -it maxpod -- ps -aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1 79.1  0.0  22472  1772 ?        Rs   06:18   3:48 awk BEGIN{while(1){}}
root          42  0.0  0.0  47600  3572 pts/0    Rs+  06:23   0:00 ps -aux

[root@master ~]#kubectl delete pods --all
```

## 全局资源管理

### 概述

> 全局资源配额 
>
> + 如果有大量的容器需要设置资源配额，为每个 Pod 设置资源配额策略不方便且不好管理。管理员可以以名称空间为单位（namespace），限制其资源的使用与创建。在该名称空间中创建的容器都会受到规则的限制。 
>
> k8s 支持的全局资源配额方式有： 
>
> + －对单个 Pod 内存、CPU 进行配额：LimitRange －对资源总量进行配额：ResourceQuota

### LimitRange

#### 默认配额策略

```shell
# 创建名称空间
[root@master ~]# kubectl create namespace work
namespace/work created
# 设置默认配额
[root@master ~]# vim limit.yaml
---
apiVersion: v1
kind: LimitRange
metadata:
  name: mylimit   # 策略名称
  namespace: work    # 规则生效的名称空间
spec: 
  limits: # 全局规则               
  - type: Container # 资源类型    
    default:             # 对没有限制策略的容器添加规则
      cpu: 300m  # 计算资源限额
      memory: 500Mi      # 计算内存资源限额
    defaultRequest:   # 设置资源请求值
      cpu: 8m  # 计算资源配额
      memory: 8Mi   # 内存资源配额

[root@master ~]# kubectl -n work apply -f limit.yaml
limitrange/mylimit created 
```

#### 验证策略

> 在默认名称空间和work名称空间分别创建一个Pod，该Pod内有个死循环耗占CPU，观察两个Pod的CPU情况
>
> [注]：work名称空间设置了默认配额

```shell
# 编写资源对象文件
[root@master ~]# vim maxpod.yaml
---
kind: Pod
apiVersion: v1
metadata:
  name: maxpod
spec:
  terminationGracePeriodSeconds: 0
  containers:
  - name: linux
    image: myos:8.5
    command: ["awk", "BEGIN{while(1){}}"]
    
[root@master ~]# kubectl apply -f maxpod.yaml -n default 
pod/maxpod created
[root@master ~]# kubectl apply -f maxpod.yaml -n work 
pod/maxpod created

[root@master ~]# kubectl top pods -n default
NAME     CPU(cores)   MEMORY(bytes)   
maxpod   996m         0Mi             
          
[root@master ~]# kubectl top pods -n work
NAME     CPU(cores)   MEMORY(bytes)   
maxpod   299m         0Mi  

[root@master ~]#kubectl delete pods --all -n work
[root@master ~]#kubectl delete pods --all -n default
```

#### 自定义资源

> 根据需求可在默认的资源分配中自定义资源分配
>
> 假如需要在work中创建一个pod，默认的资源分配不满主需求；次时可以根据自定义设置

```shell
# 修改资源对象文件
[root@master ~]# vim maxpod.yaml
---
kind: Pod
apiVersion: v1
metadata:
  name: maxpod
spec:
  terminationGracePeriodSeconds: 0
  containers:
  - name: linux
    image: myos:8.5
    command: ["awk", "BEGIN{while(1){}}"]
    resources:  # 自定义设置资源分配
      requests:
        cpu: 10m
        memory: 10Mi
      limits:
        cpu: 1100m
        memory: 2000Mi

# 验证测试
[root@master ~]# kubectl apply -f  maxpod.yaml -n work
pod/maxpod created
[root@master ~]# kubectl top pods -n work
NAME     CPU(cores)   MEMORY(bytes)   
maxpod   997m         0Mi             
```

> 存在弊端：造成资源浪费，被一个pod所耗尽

#### 资源配额范围

> 解决自定义资源策略中的弊端，设置一个范围防止在自定义时超出总的资源范围，有效防止资源被一个pod耗尽

```shell
# 修改work名称空间的资源策略
[root@master ~]# vim limit.yaml 
---
apiVersion: v1
kind: LimitRange
metadata:
  name: mylimit 
  namespace: work   
spec:
  limits:               
  - type: Container     
    default:            
      cpu: 300m 
      memory: 500Mi     
    defaultRequest:
      cpu: 8m  
      memory: 8Mi 
    max:  # 添加max与min，max：创建最大不能超过，min：创建最小不能低于
      cpu: 800m
      memory: 1000Mi
    min:
      cpu: 2m
      memory: 8Mi
# 生效配置文件
[root@master ~]# kubectl -n work apply -f limit.yaml 
limitrange/mylimit created
# 查看配置
[root@master ~]# kubectl describe namespace work
.....
Resource Limits
 Type       Resource  Min  Max     Default Request  Default Limit  Max Limit/Request Ratio
 ----       --------  ---  ---     ---------------  -------------  -----------------------
 Container  memory    8Mi  1000Mi  8Mi              500Mi          -
 Container  cpu       2m   800m    8m               300m          
```

#### 验证策略

```shell
# 以自定义资源中的资源对象文件创建测试
[root@master ~]# kubectl apply -f maxpod.yaml  -n work
Error from server (Forbidden): error when creating "maxpod.yaml": pods "maxpod" is forbidden: [maximum memory usage per Container is 1000Mi, but limit is 2000Mi, maximum cpu usage per Container is 800m, but limit is 1100m]
```

> 存在弊端：以上只是限制了一个pod中某一个容器的资源最大与最小值，当在一个pod中创建多个容器时还是会把资源耗尽

#### 多容器资源配额

> 一个Pod创建多个容器耗光资源

```shell
[root@master ~]# vim maxpod.yaml 
---
kind: Pod
apiVersion: v1
metadata:
  name: maxpod
spec:
  terminationGracePeriodSeconds: 0
  containers:
  - name: linux
    image: myos:8.5
    command: ["awk", "BEGIN{while(1){}}"]
    resources:
      requests:
        cpu: 10m
        memory: 10Mi
      limits:
        cpu: 800m
        memory: 1000Mi
  - name: linux1
    image: myos:8.5
    command: ["awk", "BEGIN{while(1){}}"]
    resources:
      requests:
        cpu: 10m
        memory: 10Mi
      limits:
        cpu: 800m
        memory: 1000Mi

[root@master ~]# kubectl -n work apply -f maxpod.yaml 
pod/maxpod created
[root@master ~]# kubectl -n work get pods
NAME     READY   STATUS    RESTARTS   AGE
maxpod   2/2     Running   0          50s
[root@master ~]# kubectl -n work top pods maxpod
NAME     CPU(cores)   MEMORY(bytes)   
maxpod   1610m        0Mi  
```

#### Pod资源配额

> 解决多容器资源配额，限制Pod的总共资源，不关心创建的容器个数，资源耗光将无法创建

```shell
# 修改work名称空间的资源策略
[root@master ~]# vim limit.yaml 
---
apiVersion: v1
kind: LimitRange
metadata:
  name: mylimit
  namespace: work
spec:
  limits:               
  - type: Container     
    default:            
      cpu: 300m 
      memory: 500Mi     
    defaultRequest:
      cpu: 8m  
      memory: 8Mi 
    max:
      cpu: 800m
      memory: 1000Mi
    min:
      cpu: 2m
      memory: 8Mi
  - type: Pod  # 添加一个Pod的最大与最小资源限制
    max:
      cpu: 1200m
      memory: 1200Mi
    min:
      cpu: 2m
      memory: 8Mi
# 生效配置
[root@master ~]# kubectl -n work apply -f limit.yaml
limitrange/mylimit configured
# 还是以多容器的资源对象文件为例
[root@master ~]# kubectl -n work apply -f maxpod.yaml 
Error from server (Forbidden): error when creating "maxpod.yaml": pods "maxpod" is forbidden: [maximum cpu usage per Pod is 1200m, but limit is 1600m, maximum memory usage per Pod is 1200Mi, but limit is 2097152k]



# 存在弊端；创建多个Pod，资源也将会耗尽
[root@master ~]# vim maxpod.yaml 
---
kind: Pod
apiVersion: v1
metadata:
  name: maxpod
spec:
  terminationGracePeriodSeconds: 0
  containers:
  - name: linux
    image: myos:8.5
    command: ["awk", "BEGIN{while(1){}}"]
    resources:
      requests:
        cpu: 10m
        memory: 10Mi
      limits:
        cpu: 800m
        memory: 1000Mi
[root@master ~]# for i in app{1..9};do sed "s,maxpod,${i}," maxpod.yaml ;done |kubectl -n work apply -f -
# Pod 创建成功后，查看节点资源使用情况
[root@master ~]# kubectl top nodes
NAME        CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
master      74m          3%     1409Mi          40%       
node-0001   1994m        99%    685Mi           19%       
node-0002   1993m        99%    686Mi           19%       
node-0003   1625m        81%    668Mi           19%       
node-0004   27m          1%     678Mi           19%       
node-0005   830m         41%    702Mi           20%


[root@master ~]# kubectl delete pods --all -n work
```

### ResourceQuota

> 最佳方法：可以限制Pod的个数以及总的内存与cpu的资源分配

#### 全局配额策略

```shell
[root@master ~]# vim quota.yaml
---
apiVersion: v1 
kind: ResourceQuota # 全局资源限额对象
metadata:
  name: myquota # 规则名称
  namespace: work  # 规则作用的名称空间
spec: # ResourceQuota.spec定义
  hard: # 创建强制规则
    requests.cpu: 1000m # 计算资源配额总数
    requests.memory: 2000Mi # 内存资源配额总数
    limits.cpu: 5000m # 计算资源限额
    limits.memory: 8Gi # 内存资源限额
    pods: 3
    
# 修改work名称空间的资源策略
[root@master ~]# kubectl -n work apply -f quota.yaml 
resourcequota/myquota created
```

#### 验证quota配额

```shell
[root@master ~]# vim maxpod.yaml 
---
kind: Pod
apiVersion: v1
metadata:
  name: maxpod
spec:
  terminationGracePeriodSeconds: 0
  containers:
  - name: linux
    image: myos:8.5
    command: ["awk", "BEGIN{while(1){}}"]
    resources:
      requests:
        cpu: 10m
        memory: 10Mi
      limits:
        cpu: 800m
        memory: 1000Mi

# 根据全局限额，最终只能创建三个Pod
[root@master ~]# for i in app{1..5};do sed "s,maxpod,${i}," maxpod.yaml ;done |kubectl -n work apply -f -
pod/app1 created
pod/app2 created
pod/app3 created
Error from server (Forbidden): error when creating "STDIN": pods "app4" is forbidden: exceeded quota: myquota, requested: pods=1, used: pods=3, limited: pods=3
Error from server (Forbidden): error when creating "STDIN": pods "app5" is forbidden: exceeded quota: myquota, requested: pods=1, used: pods=3, limited: pods=3



[root@master ~]# kubectl -n work delete pods --all
pod "app1" deleted
pod "app2" deleted
pod "app3" deleted
[root@master ~]# kubectl -n work delete -f limit.yaml -f quota.yaml
limitrange "mylimit" deleted
resourcequota "myquota" deleted

[root@master ~]# kubectl delete namespace work
namespace "work" deleted
```

# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习
- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [模块](#模块)
    - [模块多参数写法](#模块多参数写法)
    - [软件包组安装](#软件包组安装)
    - [更新所有系统包](#更新所有系统包)
  - [预定义变量](#预定义变量)
    - [facts变量-setup模块](#facts变量-setup模块)
    - [setup快速查找内容](#setup快速查找内容)
  - [debug模块](#debug模块)
  - [自定义变量](#自定义变量)
    - [定义方式](#定义方式)
    - [案例1](#案例1)
    - [inventory变量](#inventory变量)
    - [使用inventory变量](#使用inventory变量)
    - [playbook变量](#playbook变量)
    - [变量定义在文件](#变量定义在文件)
  - [firewalld模块](#firewalld模块)
    - [安装测试服务](#安装测试服务)
    - [安装firewalld](#安装firewalld)
    - [配置防火墙规则](#配置防火墙规则)
  - [template模块](#template模块)
  - [playbook错误处理](#playbook错误处理)
    - [错误案例](#错误案例)
    - [忽略错误-单个](#忽略错误-单个)
    - [忽略错误-全局](#忽略错误-全局)
  - [触发执行任务](#触发执行任务)
    - [案例1-未调整](#案例1-未调整)
    - [弊端](#弊端)
    - [案例1-调整](#案例1-调整)
  - [when条件](#when条件)
  - [motd文件](#motd文件)
  - [regitster注册变量](#regitster注册变量)
    - [提取内容](#提取内容)
- [快捷键](#快捷键)
  - [文件中删除单个词](#文件中删除单个词)
- [问题](#问题)
  - [请简述facts变量的作用与用法](#请简述facts变量的作用与用法)
  - [ansible的自定义变量有什么作用](#ansible的自定义变量有什么作用)
  - [ansible的自定义变量都可以在哪些地方定义](#ansible的自定义变量都可以在哪些地方定义)
  - [ansible的ignore\_errors作用是什么？](#ansible的ignore_errors作用是什么)
  - [ansible的handlers作用是什么？](#ansible的handlers作用是什么)
  - [ansible的when作用是什么？](#ansible的when作用是什么)
- [补充](#补充)
  - [查看端口对应服务](#查看端口对应服务)
  - [vim打开指定行](#vim打开指定行)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)


# 学习目标

Ansible模块进阶及变量

Ansible进阶

# 课堂笔记（命令）



# 课堂笔记（文本）

## 模块

### 模块多参数写法

```shell
[root@pubserver ansible]# vim pkg.yml
---      
- name: install package
  hosts: webservers
  tasks: 
    - name: install web package
      yum:
        name: httpd,php,php-mysqlnd,php-fpm            # 默认书写模式
        state: present         
```

```shell
[root@pubserver ansible]# vim pkg.yml
---      
- name: install package
  hosts: webservers
  tasks: 
    - name: install web package
      yum:
        name: [httpd,php,php-mysqlnd,php-fpm]      # 使用python列表模式书写参数
        state: present 
```

```shel
[root@pubserver ansible]# vim pkg.yml
---        
- name: install package
  hosts: webservers
  tasks:   
    - name: install web package
      yum: 
        name:                          # 常用多参数书写格式
          - httpd
          - php
          - php-mysqlnd
          - php-fpm
        state: present 
```

### 软件包组安装

```shell
# 根据功能等，可以将一系列软件放到一个组中，安装软件包组，将会把很多软件一起安装上。比如gcc、java等都是开发工具，安装开发工具包组，将会把它们一起安装。
[root@node1 ~]# yum grouplist   # 列出所有的软件包组
[root@node1 ~]# yum groupinstall "Development Tools"
# 如果列出的组名为中文，可以这样进行：
[root@node1 ~]# LANG=C yum grouplist

# 继续编辑pkg.yml，在webservers组中的主机上安装Development tools组
[root@pubserver ansible]# vim pkg.yml
---
- name: install pkckage
  hosts: webservers
  tasks:
    - name: install web pkgs    # 此任务通过yum安装三个包
      yum:
        name:
          - httpd
          - php
          - php-mysqlnd
          - php-fpm
        state: present

    - name: install dev group    # 此任务通过yum安装一组包
      yum:
        name: "@Development Tools"   # @表示后面的名字是组名
        state: present

[root@pubserver ansible]# ansible-playbook pkg.yml
```

### 更新所有系统包

```shell
[root@pubserver ansible]# vim pkg.yml
---     
- name: install package
  hosts: webservers
  tasks:
    - name: update system
      yum:
        name: "*"    # 表示系统已经安装的所有包    
        state: latest
```

## 预定义变量

### facts变量-setup模块

```shell
# 查看系统所有fscts变量:预定义变量
[root@pubserver ansible]# ansible webservers -m setup
```

```shell
#facts变量是一个大的由{}构成的键值对字典。在{}中，有很多层级的嵌套。可以通过参数过滤出第一个层级的内容。
# 查看所有的IPV4地址，filter是过滤的意思
[root@pubserver ansible]# ansible webservers -m setup -a "filter=ansible_all_ipv4_addresses"

# 查看可用内存
[root@pubserver ansible]# ansible webservers -m setup -a "filter=ansible_memfree_mb"
```

### setup快速查找内容

```shell
[root@pubserver ansible]# ansbile webservers -m setup | less # 使用/输入需要查找的内容，内容按照层级查找，每一层级加上一个点，最高级为ansible_

# 例：查找vda1磁盘大小
[root@pubserver ansible]# ansbile webservers -m setup | less
        "ansible_devices": {
            "vda": {
                "holders": [],
                "host": "",
                "links": {
                    "ids": [],
                    "labels": [],
                    "masters": [],
                    "uuids": []
                },
                "model": null,
                "partitions": {
                    "vda1": {
                        "holders": [],
                        "links": {
                            "ids": [],
                            "labels": [],
                            "masters": [],
                            "uuids": [
                                "4590ca1e-1149-487f-ba53-c994940fede0"
                            ]
                        },
                        "sectors": "41940959",
                        "sectorsize": 512,
                        "size": "20.00 GB",
                        "start": "2048",
                        "uuid": "4590ca1e-1149-487f-ba53-c994940fede0"
                    }
                }
# 通过ansbile webservers -m setup | less命令输入/输入关键词sda1，按n进行翻页，以上为关键内容，其层级关系为：ansible_devices.vda.partitions.vda1.size
```

## debug模块

```shell
# 使用debug模块配合facts与定义变量输出内容
[root@pubserver ansible]# vim debug.yml
---
- name: display host info
  hosts: webservers
  tasks:
    - name: display hostname and memory
      debug:
        msg: "hostname:{{ansible_hostname}}, mem:{{ansible_memtotal_mb}}MB，eth0 ip:{{ansible_eth0.ipv4.address}},vda1 size: {{ansible_devices.vda.partitions.vda1.size}}"	
```

## 自定义变量

> 引入变量，可以方便Playbook重用。比如装包的playbook，包名使用变量。多次执行playbook，只要改变变量名即可，不用编写新的playbook。

### 定义方式

> - ansible支持10种以上的变量定义方式。常用的变量来源如下：
>   - inventory变量。变量来自于主机清单文件
>   - facts变量。
>   - playbook变量。变量在playbook中定义。
>   - 变量文件。专门创建用于保存变量的文件。推荐变量写入单独的文件。

### 案例1

> webservers创建用户zhaoliu
>
> db1创建用户wangwu

**多个play写法**

```shell
[root@pubserver ansible]# vim var1.yml
---
- name: webservers create user
  hosts: webservers
  tasks:
    - name: create user
      user:
        name: zhaoliu
        state: present
        
- name: create user in dbs
  hosts: dbs
  tasks:
    - name: create some users
      user:
        name: wangwu
        state: present
```

### inventory变量

```shell
[root@pubserver ansible]# vim inventory
[webservers]
web[1:2]
[webservers:vars]  # 定义组变量的方法，:vars是固定格式；组里面每个用户都作用于
username=zhaoliu

[dbs]
db1 username=wangwu  # 定义主机变量的方法，只是针对单个主机

[cluster:children]
webservers
dbs

# 两种方法都适用
```

### 使用inventory变量

> 简化案例1

```shell
[root@pubserver ansible]# vim var1.yml
---
- name: webservers create user
  hosts: webservers,dbs
  tasks:
    - name: create user
      user:
        name: "{{username}}"
        state: present

# 通过调用清单中设置的变量
```

### playbook变量

> vars 用于声明变量

```shell
[root@pubserver ansible]# vim var1.yml
---
- name: create user
  hosts: webservers
  vars:    # 固定格式，用于声明变量
    username: "jack"    # 此处引号可有可无
    mima: '123456'      # 此处引号是需要的，表示数字字符，若含有字母可不加引号
  tasks:
    - name: create some users
      user:
        name: "{{username}}"   # {}出现在开头，必须有引号
        state: present
        password: "{{mima|password_hash('sha512')}}"
```

 password: "{{mima|password_hash('sha512')}}"

### 变量定义在文件

> vars_file 固定格式，声明变量文件

```shell
[root@pubserver ansible]# vim vars.yml   # 文件名自定义
---
yonghu: rose
mima: abcd
[root@pubserver ansible]# vim var1.yml 
---
- name: create user
  hosts: webservers
  vars_files: vars.yml   # vars_files用于声明变量文件
  tasks:
    - name: create some users
      user:
        name: "{{yonghu}}"   # 这里的变量来自于vars.yml
        state: present
        password: "{{mima|password_hash('sha512')}}"
```

password: "{{mima|password_hash('sha512')}}"

## firewalld模块

> - 用于配置防火墙的模块
> - 常用选项：
>   - port：声明端口
>   - permanent：永久生效，但不会立即生效
>   - immediate：立即生效，临时生效
>   - state：enabled，放行；disabled拒绝
> - 防火墙一般默认拒绝，明确写入允许的服务。
> - 有一些服务有名字，有些服务没有名字。但是最终都是基于TCP或UDP的某些端口。比如http服务基于TCP80端口。服务名和端口号对应关系的说明文件是：`/etc/services`
> - 配置服务器的防火墙，一般来说只要配置开放哪些服务或端口即可。没有明确开放的，都默认拒绝

### 安装测试服务

```shell
# 安装nginx服务
[root@pubserver ansible]# vim firewall.yml
---
- name: configure webservers
  hosts: webservers
  tasks:
    - name: install nginx pkg   # 这里通过yum模块装nginx
      yum:
        name: nginx
        state: present

    - name: start nginx service   # 这里通过service模块启nginx服务
      service:
        name: nginx
        state: started
        enabled: yes
[root@pubserver ansible]# ansible-playbook firewall.yml
[root@pubserver ansible]# curl http://192.168.88.11/  # 可访问
```

### 安装firewalld

```shell
# 安装firewalld启动firewalld
[root@pubserver ansible]# vim firewall.yml
---
- name: configure webservers
  hosts: webservers
  tasks:
    - name: install nginx pkg   # 这里通过yum模块装nginx
      yum:
        name: nginx,firewalld
        state: present

    - name: start nginx service   # 这里通过service模块启nginx服务
      service:
        name: nginx
        state: started
        enabled: yes # 开机自启动
        
    - name: start firewalld service   # 这里通过service模块启firewalld服务
      service:
        name: firewalld
        state: started
        enabled: yes   
[root@pubserver ansible]# ansible-playbook firewall.yml
[root@pubserver ansible]# curl http://192.168.88.11/  # 被拒绝       
```

### 配置防火墙规则

```shell
# 配置防火墙规则，放行http协议
[root@pubserver ansible]# vim firewall.yml
---
- name: configure webservers
  hosts: webservers
  tasks:
    - name: install nginx pkg   # 这里通过yum模块装nginx
      yum:
        name: nginx,firewalld
        state: present

    - name: start nginx service   # 这里通过service模块启nginx服务
      service:
        name: nginx
        state: started
        enabled: yes

    - name: start firewalld service   # 这里通过service模块启service服务
      service:
        name: firewalld
        state: started
        enabled: yes
  
    - name: set firewalld rules   # 通过firewalld模块开放80端口
      firewalld:
        port: 80/tcp
        permanent: yes   # 永久生效，单独配置需要重启才生效
        immediate: yes   # 立即生效
        state: enabled  # 放行

[root@pubserver ansible]# ansible-playbook firewall.yml 
[root@pubserver ansible]# curl http://192.168.88.11/  # 可访问
```

## template模块 

> - copy模块可以上传文件，但是文件内容固定
> - template模块可以上传具有特定格式的文件（如文件中包含变量）
> - 当远程主机接收到文件之后，文件中的变量将会变成具体的值
> - template模块上传的文件，使用的语法叫Jinja2。
> - 常用选项：
>   - src：要上传的文件
>   - dest：目标文件路径

```shell
[root@pubserver ansible]# vim index.html
Welcome to {{ansible_hostname}} on {{ansible_eth0.ipv4.address}}

[root@pubserver ansible]# vim templ.yml
---
- name: upload index
  hosts: webservers
  tasks:
    - name: create web index
      template:
        src: index.html
        dest: /usr/share/nginx/html/index.html

[root@pubserver ansible]# ansible-playbook templ.yml
[root@pubserver ansible]# curl http://192.168.88.11/
Welcome to web1 on 192.168.88.11
[root@pubserver ansible]# curl http://192.168.88.12
Welcome to web2 on 192.168.88.12
```

## playbook错误处理

> 当Playbook中包含多个任务时，当某个任务遇到错误，它将崩溃，终止所有执行

### 错误案例

```shell
[root@pubserver ansible]# vim merr.yml   # 前提web机器中未安装mysql
---
- name: install package
  hosts: webservers
  tasks: 
    - name: start mysqld
      service: 
        name: mysqld
        state: started
        enabled: yes
    - name: create file
      file: 
        path: /tmp/service
        state: touch
        
---------报错信息------------       
 TASK [start mysqld] ************************************************************
fatal: [web2]: FAILED! => {"changed": false, "msg": "Could not find the requested service mysqld: host"}
fatal: [web1]: FAILED! => {"changed": false, "msg": "Could not find the requested service mysqld: host"}       
```

### 忽略错误-单个

```shell
[root@pubserver ansible]# vim merr.yml 
---    
- name: install package
  hosts: webservers
  tasks:
    - name: start mysqld
      ignore_errors: yes # 即使这个任务失败了，也要继续执行下去，只是针对单个任务
      service:
        name: mysqld
        state: started
        enabled: yes
    - name: create file
      file:
        path: /tmp/service
        state: touch
        
--------------错误信息--------------------   
TASK [start mysqld] ************************************************************
fatal: [web2]: FAILED! => {"changed": false, "msg": "Could not find the requested service mysqld: host"}
...ignoring
fatal: [web1]: FAILED! => {"changed": false, "msg": "Could not find the requested service mysqld: host"}
...ignoring

TASK [create file] *************************************************************
changed: [web2]
changed: [web1]   
```

### 忽略错误-全局

```
[root@pubserver ansible]# vim merr.yml 
---    
- name: install package
  ignore_errors: yes # 作用全局，一个play中的多个任务都享用忽略错误功能
  hosts: webservers
  tasks:
    - name: start mysqld
      service:
        name: mysqld
        state: started
        enabled: yes
    - name: create file
      file:
        path: /tmp/service
        state: touch
```

## 触发执行任务

> - 通过handlers定义触发执行的任务
> - handlers中定义的任务，不是一定会执行的
> - 在tasks中定义的任务，通过notify关键通知handlers中的哪个任务要执行
> - 只有tasks中的任务状态是changed才会进行通知。

### 案例1-未调整

```shell
# 下载web1上的/etc/nginx/nginx.conf
[root@pubserver ansible]# vim get_conf.yml
---
- name: download nginx.conf
  hosts: web1
  tasks:
    - name: get nginx.conf
      fetch:
        src: /etc/nginx/nginx.conf
        dest: ./
        flat: yes    # 直接下载文件，不要目录
[root@pubserver ansible]# ansible-playbook get_conf.yml

# 修改nginx.conf的端口为变量
[root@pubserver ansible]# vim +39 nginx.conf # 光标移动到39行
... ...
    server {
        listen       {{http_port}} default_server;
        listen       [::]:{{http_port}} default_server;
        server_name  _;
... ...

# 修改nginx服务的端口为8000，重启nginx
[root@pubserver ansible]# vim trigger.yml
---
- name: configure nginx
  hosts: webservers
  vars:
    http_port: "8000"   # 定义nginx.conf中的变量和值
  tasks:
    - name: upload nginx.conf   # 上传nginx.conf
      template:
        src: ./nginx.conf
        dest: /etc/nginx/nginx.conf

    - name: restart nginx    # 重启服务
      service:
        name: nginx
        state: restarted
# 第一次执行trigger.yml，上传文件和重启服务两个任务的状态都是黄色changed
[root@pubserver ansible]# ansible-playbook trigger.yml
# 第二次执行trigger.yml，上传文件的任务状态是绿色的ok，重启服务任务的状态是黄色changed
[root@pubserver ansible]# ansible-playbook trigger.yml
```

### 弊端

> 既然配置文件没有改变，那么服务就不应该重启
> 修改Playbook，只有配置文件变化了，才重启服务

### 案例1-调整

>  handler
>
>  **Handlers：**处理器，当changed状态条件满足时，（notify）触发执行的操作

```shell
[root@pubserver ansible]# vim trigger.yml
---                      
- name: configure nginx  
  hosts: webservers      
  vars:                  
    http_port: "800"     
  tasks:                 
    - name: upload nginx file
      template:          
        src: nginx.conf  
        dest: /etc/nginx/nginx.conf
      notify: restart nginx   #如以上操作后为changed的状态时，会通过notify指定的名称触发对应名称的handlers操作
  handlers: #handlers中定义的就是任务，此处handlers中的任务使用的是service模           
    - name: restart nginx  #notify和handlers中任务的名称必须一致
      service:           
        name: nginx      
        state: restarted
        
----------解释-------------------  
# 任务1改变后才通知任务2；tasks下面有两个name即为两个任务，当任务1发生改变后才执行通知notify中的对象；在playbook中黄色代表改变，绿色代表执行成功
```

## when条件

> - 只有满足某一条件时，才执行任务
> - 常用的操作符：
>   - ==：相等
>   - !=：不等
>   - `>`：大于
>   - `<`：小于
>   - `<=`：小于等于
>   - `>=`：大于等于
> - 多个条件或以使用and或or进行连接类似shell的逻辑与逻辑或
> - when表达式中的变量，可以不使用`{{}}`

```shell
# 当dbs组中的主机内存大于2G的时候，才安装mysql-server
[root@pubserver ansible]# vim when1.yml
---
- name: install mysql-server
  hosts: dbs
  tasks:
    - name: install mysql-server pkg
      yum:
        name: mysql-server
        state: present
      when: ansible_memtotal_mb>2048

# 如果目标主机没有2GB内存，则不会安装mysqld-server
[root@pubserver ansible]# ansible-playbook when1.yml
```

## motd文件

> 作用：用于向用户提供欢迎信息、系统更新信息、安全警告、系统状态等动态信
>
> /etc/motd

## regitster注册变量

> 作用：用于保存一个task任务的执行结果，以便于debug时使用。或者将此次task任务的结果作为条件，去判断是否去执行其他task任务

```shell
[root@pubserver ansible]# vim reg.yml
---      
- name: create
  hosts: web1
  tasks: 
    - name: create file
      file:
        path: /tmp/2.txt
        state: touch
      register: result
    - name: display content
      debug:
        msg: "{{result}}"
        
--------------输出result信息-------------------       
TASK [display content] *********************************************************
ok: [web1] => {
    "msg": {
        "changed": true,
        "dest": "/tmp/2.txt",
        "diff": {
            "after": {
                "atime": 1703065841.5716975,
                "mtime": 1703065841.5716975,
                "path": "/tmp/2.txt",
                "state": "touch"
            },
            "before": {
                "atime": 1703065841.570927,
                "mtime": 1703065841.570927,
                "path": "/tmp/2.txt",
                "state": "absent"
            }
        },
        "failed": false,
        "gid": 0,
        "group": "root",
        "mode": "0644",
        "owner": "root",
        "size": 0,
        "state": "file",
        "uid": 0
    }
}
```

### 提取内容

> 类似python中的字典取值

```shell
[root@pubserver ansible]# vim reg.yml
---      
- name: create
  hosts: web1
  tasks: 
    - name: create file
      file:
        path: /tmp/2.txt
        state: touch
      register: result
    - name: display content
      debug:
        msg: "{{result['diff']['after']}}"

-------------------输出信息---------------------------
TASK [display content] *********************************************************
ok: [web1] => {
    "msg": {
        "atime": 1703065891.463906,
        "mtime": 1703065891.463906,
        "path": "/tmp/2.txt",
        "state": "touch"
    }
}
```





# 快捷键

## 文件中删除单个词

> cw  ： 删除单个词语并进入编辑模式
>
> dw：删除单个单词

# 问题

## 请简述facts变量的作用与用法

> Ansible在执行任务时自动收集的主机信息，包括主机名、IP地址、操作系统、内存、CPU、磁盘等硬件信息，以及用户、进程、服务等软件信息。Facts可以通过setup模块来收集。收集到的Facts信息可以用于任务的条件判断、变量赋值、模板渲染等操作
>
> 在Ansible中，Facts是以字典的形式存储的，可以通过变量{{ ansible_facts }}来引用。例如，可以使用{{ ansible_facts['hostname'] }}来获取主机名，使用{{ ansible_facts['os_family'] }}来获取操作系统家族。除了ansible_facts变量外，还有一些特殊的Facts变量，例如ansible_architecture、ansible_distribution、ansible_memtotal_mb等，可以直接引用。

## ansible的自定义变量有什么作用

> Ansible的自定义变量可以用于存储和传递Playbook中的数据，例如主机列表、软件包名称、配置文件路径等。自定义变量可以在Playbook中定义和使用，也可以在命令行中传递和覆盖。使用自定义变量可以实现Playbook的可重用性和可配置性，使得同一个Playbook可以适用于不同的环境和需求

## ansible的自定义变量都可以在哪些地方定义

> Playbook中的vars关键字：可以在Playbook的顶层定义变量，这些变量对所有任务都可见。
>
> Playbook中的vars_files关键字：可以从外部文件中加载变量，这些变量对所有任务都可见。
>
> Playbook中的vars_prompt模块：可以在运行Playbook时提示用户输入变量的值，这些变量对所有任务都可见。
>
> Playbook中的set_fact模块：可以在任务中动态创建变量，这些变量只对当前任务可见。
>
> Inventory中的组变量：可以在Inventory文件中定义组变量，这些变量对该组内的所有主机都可见。
>
> Inventory中的主机变量：可以在Inventory文件中定义主机变量，这些变量只对该主机可见。
>
> 命令行中的-e选项：可以在运行ansible-playbook命令时传递变量，这些变量会覆盖Playbook中定义的同名变量。
>
> Ansible配置文件中的变量：可以在Ansible配置文件中定义变量，这些变量对所有Playbook都可见。
>
> 在实际使用中，应根据需要选择合适的方式创建变量，以实现最佳的可读性、可维护性和可重用性。

## ansible的ignore_errors作用是什么？

> ignore_errors是Ansible Playbook中的一个关键字参数，用于指定在执行任务时忽略错误。当设置ignore_errors为true时，即使任务执行失败，Ansible也不会停止执行Playbook，而是继续执行下一个任务。这个参数通常用于处理一些非关键性的错误，例如文件不存在、命令执行失败等情况。使用ignore_errors可以提高Playbook的健壮性和容错性，避免因为一些小错误导致整个Playbook执行失败。

```shell
- name: Execute command and ignore errors  # 执行命令并忽略错误
 command: /path/to/command
 ignore_errors: tru
```

## ansible的handlers作用是什么？

> Handlers是Ansible中的一种特殊任务，它们通常用于在Playbook中定义一些需要在任务执行后立即执行的操作，例如重启服务、重新加载配置文件等。Handlers只有在被通知时才会执行，通知可以由任何任务发出，只要它们使用了notify关键字并指定了要通知的Handler名称即可。

```shell
- name: Install and start nginx on hosts in slave group  # 在web组中的主机上安装并启动nginx服务
 hosts: slave
 become: true
 tasks:
   - name: Install nginx  # 安装nginx
    yum:
      name: nginx
      state: present
    notify: Restart nginx  # 安装完成后通知Restart nginx Handler
 handlers:
   - name: Restart nginx  # 重启nginx服务
    service:
      name: nginx
      state: restarted
```

## ansible的when作用是什么？

> when关键字用于在任务执行前判断一个条件是否成立，如果条件成立则执行任务，否则跳过任务。when关键字可以判断变量是否相等、是否包含某个字符串、是否大于或小于某个值等。when关键字可以在任务级别使用，也可以在块级别使用。

```shell
- name: Check memory size on slave01  # 检查slave01的内存大小
 hosts: slave01
 tasks:
   - name: Check memory size  # 检查内存大小
    debug:
      msg: "Memory OK"
    when: ansible_memtotal_mb > 4096  # 当内存大于4GB时输出
```

# 补充

## 查看端口对应服务

> cat /etc/services

## vim打开指定行

```shell
vim +10 /etc/passwd  # 打开passwd文件将光标移动到第10行
```



# 今日总结



# 昨日复习

ping、command、shell、script、file、copy、fetch、lineinfile、replace、user、group、yum_repository、yum、service、lvg、lvol、filesystem、mount、parted
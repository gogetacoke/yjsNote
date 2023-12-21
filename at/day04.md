- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [读文前提](#读文前提)
  - [when-条件判断](#when-条件判断)
    - [案例1-文件创建](#案例1-文件创建)
  - [block-任务块](#block-任务块)
    - [demo](#demo)
    - [rescue和always](#rescue和always)
  - [loop循环](#loop循环)
    - [列表](#列表)
    - [字典](#字典)
  - [role角色](#role角色)
    - [配置清单](#配置清单)
    - [参数解释](#参数解释)
    - [案例测试1](#案例测试1)
    - [案例测试2](#案例测试2)
    - [案例测试3](#案例测试3)
  - [角色与剧本](#角色与剧本)
  - [ansible加接密](#ansible加接密)
    - [加密](#加密)
    - [解密-永久解开](#解密-永久解开)
    - [解密-临时解开](#解密-临时解开)
    - [修改密码](#修改密码)
    - [高级使用](#高级使用)
    - [playbook加密](#playbook加密)
  - [特殊主机清单变量](#特殊主机清单变量)
    - [准备前提](#准备前提)
    - [配置特殊变量](#配置特殊变量)
- [快捷键](#快捷键)
  - [多行同时缩进](#多行同时缩进)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)


# 学习目标

任务块及循环

角色

普通用户使用Ansible

特殊主机清单变量

# 课堂笔记（命令）



# 课堂笔记（文本）

## 读文前提

> 文中有\{\{\}\}的都加上了斜线进行转移，为了在github上显示，实际代码操作不用添加

## when-条件判断

### 案例1-文件创建

```shell
[root@pubserver ansible]# vim when.yml
---      
- name: block
  hosts: webservers
  ignore_errors: yes
  tasks: 
    - name: create file
      file:
        path: /tmp/abc  # 文件夹存在
        state: touch
      register: rs
    - name: debug failed
      debug:
        msg: "content error"
      when: rs.failed == True  # 当failed为true时才执行
      
------------输出--------------------- 
TASK [create file] *************************************************************
changed: [web1]
changed: [web2]

TASK [debug failed] ************************************************************
skipping: [web1]
skipping: [web2]
---------------------------------------

[root@pubserver ansible]# vim when.yml
---      
- name: block
  hosts: webservers
  ignore_errors: yes
  tasks: 
    - name: create file
      file:
        path: /tmp/demos/abc  # 文件夹不存在
        state: touch
      register: rs
    - name: debug failed
      debug:
        msg: "content error"
      when: rs.failed == True  # 当failed为true时才执行
   
-----------------输出----------------------
TASK [create file] *************************************************************
fatal: [web1]: FAILED! => {"changed": false, "msg": "Error, could not touch target: [Errno 2] No such file or directory: b'/tmp/demo/abc'", "path": "/tmp/demo/abc"}
...ignoring
fatal: [web2]: FAILED! => {"changed": false, "msg": "Error, could not touch target: [Errno 2] No such file or directory: b'/tmp/demo/abc'", "path": "/tmp/demo/abc"}
...ignoring

TASK [debug failed] ************************************************************
ok: [web1] => {
    "msg": "content error"
}
ok: [web2] => {
    "msg": "content error"
}
```

**代码解释**

> 通过register模块，获取任务执行后的详细信息中有个failed值，当任务执行成功时值为false，执行失败为true，以上通过when判断该值，来执行debug任务，用于任务1失败时给出提示

## block-任务块

> 可以通过block关键字，将多个任务组合到一起
> 可以将整个block任务组，一起控制是否要执行

### demo

```shell
# 如果webservers组中的主机系统发行版是Rocky，则安装并启动nginx
[root@pubserver ansible]# vim block1.yml
---
- name: block tasks
  hosts: webservers
  tasks:
    - name: define a group of tasks
      block:
        - name: install nginx   # 通过yum安装nginx
          yum:
            name: nginx
            state: present

        - name: start nginx     # 通过service启动nginx服务
          service:
            name: nginx
            state: started
            enabled: yes

      when: ansible_distribution=="Rocky"   # 条件为真才会执行上面的任务

[root@pubserver ansible]# ansible-playbook block1.yml
```

### rescue和always

> - block和rescue、always联合使用：
>   - block中的任务都成功，rescue中的任务不执行
>   - block中的任务出现失败（failed），rescue中的任务执行
>   - block中的任务不管怎么样，always中的任务总是执行

```shell
[root@pubserver ansible]# vim block2.yml
---
- name: block test
  hosts: webservers
  tasks:
    - name: block / rescue / always test1
      block:
        - name: touch a file
          file:
            path: /tmp/test1.txt
            state: touch
      rescue:
        - name: touch file test2.txt
          file:
            path: /tmp/test2.txt
            state: touch
      always:
        - name: touch file test3.txt
          file:
            path: /tmp/test3.txt
            state: touch

# 执行playbook web1上将会出现/tmp/test1.txt和/tmp/test3.txt
[root@pubserver ansible]# ansible-playbook block2.yml
[root@web1 ~]# ls /tmp/test*.txt
/tmp/test1.txt  /tmp/test3.txt
```

```shell
# 修改上面的playbook，使block中的任务出错
[root@web1 ~]# rm -f /tmp/test*.txt
[root@pubserver ansible]# vim block2.yml
---
- name: block test
  hosts: webservers
  tasks:
    - name: block / rescue / always test1
      block:
        - name: touch a file
          file:
            path: /tmp/abcd/test11.txt
            state: touch
      rescue:
        - name: touch file test22.txt
          file:
            path: /tmp/test22.txt
            state: touch
      always:
        - name: touch file test33.txt
          file:
            path: /tmp/test33.txt
            state: touch
# 因为web1上没有/tmp/abcd目录，所以block中的任务失败。但是playbook不再崩溃，而是执行rescue中的任务。always中的任务总是执行
[root@pubserver ansible]# ansible-playbook block2.yml
[root@web1 ~]# ls /tmp/test*.txt
/tmp/test22.txt  /tmp/test33.txt
```

## loop循环

> - 相当于shell中for循环
> - ansible中循环用到的变量名是固定的，叫item

### 列表

```shell
# 在test组中的主机上创建5个目录/tmp/{aaa,bbb,ccc,ddd,eee}
[root@pubserver ansible]# vim loop1.yml
---
- name: use loop
  hosts: webservers
  tasks:
    - name: create directory
      file:
        path: /tmp/{{item}}
        state: directory
      loop: [aaa,bbb,ccc,ddd,eee]

# 上面写法，也可以改为：
---
- name: use loop
  hosts: webservers
  tasks:
    - name: create directory
      file:
        path: /tmp/{{item}}
        state: directory
      loop: 
        - aaa
        - bbb
        - ccc
        - ddd
        - eee

[root@pubserver ansible]# ansible-playbook loop1.yml
[root@web1 ~]# ls /tmp/
aaa  bbb  ccc  ddd  eee 
```

### 字典

> 使用复杂变量。创建zhangsan用户，密码是123；创建lisi用户，密码是456
> item是固定的，用于表示循环中的变量
> 循环时，loop中每个-后面的内容作为一个整体赋值给item。
> loop中{}中的内容是自己定义的，写法为key:val
> 取值时使用句点表示。如下例中取出用户名就是{{item.uname}}

```shell
[root@pubserver ansible]# vim loop2.yml
---               
- name: dict value
  hosts: webservers
  tasks:          
    - name: create user set passwd
      user:       
        name: "{{item.username}}"
        password: "{{item.passwd|password_hash('sha512')}}"
      loop:       
        - {'username':'yyh','passwd':'123456'}
        - {'username':'yh','passwd':'123456'}
        - {'username':'hy','passwd':'123456'}
```

## role角色

> - 为了实现playbook重用，可以使用role角色
> - 角色role相当于把任务打散，放到不同的目录中
> - 再把一些固定的值，如用户名、软件包、服务等，用变量来表示
> - role角色定义好之后，可以在其他playbook中直接调用

### 配置清单

```shell
# 创建角色
# 1. 声明角色存放的位置
[root@pubserver ansible]# vim ansible.cfg 
[defaults]
inventory = hosts
roles_path = roles    # 定义角色存在当前目录的roles子目录中

# 2. 创建角色目录
[root@pubserver ansible]# mkdir roles

# 3. 创建名为motd的角色
[root@pubserver ansible]# ansible-galaxy init roles/motd
[root@pubserver ansible]# ls roles/
motd     # 生成了motd角色目录

[root@pubserver ansible]# yum install -y tree
[root@pubserver ansible]# tree roles/motd/
roles/motd/
├── defaults         # 定义变量的目录，优先级最低
│    └── main.yml
├── files            # 保存上传的文件（如copy模块用到的文件）
├── handlers         # handlers任务写到这个目录的main.yml中
│    └── main.yml
├── meta             # 保存说明数据，如角色作者、版本等
│    └── main.yml
├── README.md        # 保存角色如何使用之类的说明
├── tasks            # 保存任务
│    └── main.yml
├── templates        # 保存template模块上传的模板文件
├── tests            # 保存测试用的playbook。可选
│    ├── inventory
│    └── test.yml
└── vars             # 定义变量的位置，推荐使用的位置
     └── main.yml
```

### 参数解释

> `vars` 目录通常用于存储 role 中的私有变量，也就是只对该 role 有效的变量。这些变量通常用于定义该 role 中的任务，或者用于设置 role 的行为和选项。因此，`vars` 目录中的变量具有较高的优先级，并且可以覆盖其他层级中定义的同名变量。
>
> `defaults` 目录用于存储 role 的默认变量。这些变量通常用于设置 role 的默认行为和选项。如果没有在其他层级中定义同名变量，则这些变量将应用于 role 中的所有任务。在变量冲突的情况下，`defaults` 目录中的变量具有较低的优先级。

### 案例测试1

```shell
# 将不同的内容分别写到对应目录的main.yml中
# 创建motd模板文件
[root@pubserver ansible]# vim roles/motd/templates/motd
Hostname: {{ansible_hostname}}
Date: {{ansible_date_time.date}}
Contact to: {{admin}}

# 创建变量
[root@pubserver ansible]# vim roles/motd/vars/main.yml  # 追加一行
admin: yyh@163.c

# 创建任务
[root@pubserver ansible]# vim roles/motd/tasks/main.yml  # 追加
- name: modify motd
  template:
    src: motd      # 这里的文件，自动到templates目录下查找
    dest: /etc/motd
    
# 创建playbook，调用motd角色
[root@pubserver ansible]# vim role_motd.yml
---
- name: modify motd with role
  hosts: webservers
  roles:
    - motd 
    
# 执行playbook
[root@pubserver ansible]# ansible-playbook role_motd.yml   

---------------------执行结果-------------------
TASK [motd : create] ***********************************************************
changed: [web2]
changed: [web1]
```

### 案例测试2

```shell
# 可通过手动创建目录；编写装包角色pkgs
[root@pubserver ansible]# mkdir -p roles/pkgs/{tasks,defaults}
[root@pubserver ansible]# vim roles/pkgs/tasks/main.yml  # 配置角色任务
---  
- name: yum install
  yum:
    name: "{{pkg}}"
    state: present
[root@pubserver ansible]# vim roles/pkgs/defaults/main.yml # 配置角色参数
---
pkg: nginx

[root@pubserver ansible]# vim inst_pkg.yml # 编写剧本进行调用角色
---  
- name: yum install nginx
  hosts: webservers
  roles:
    - pkgs
     
- name: yum install mysql-server
  hosts: dbs
  vars:
    pkg: mysql-server
  roles:
    - pkgs
```

### 案例测试3

```shell
# 调用多个角色；编写服务启动角色start_svc
[root@pubserver ansible]# mkdir -p roles/start_svc/{tasks,defaults}
[root@pubserver ansible]# vim roles/start_svc/tasks/main.yml  # 配置角色任务
---  
- name: start task
  service:
    name: "{{svc}}"
    state: started
    enabled: yes
[root@pubserver ansible]# vim roles/start_svc/defaults/main.yml # 配置角色参数
---
pkg: nginx 
[root@pubserver ansible]# vim inst_pkg.yml # 编写剧本进行调用角色
---  
- name: yum install nginx
  hosts: webservers
  roles:
    - pkgs
    - start_svc
     
- name: yum install mysql-server
  hosts: dbs
  vars:
    pkg: mysql-server
    svc: mysqld
  roles:
    - pkgs
    - start_svc
```

## 角色与剧本

> 角色与剧本可以同存，但在剧本的执行顺序中，角色优先级最高

```shell
[root@pubserver ansible]# vim tasks1.yml
---  
- name: role test
  hosts: webservers
  vars:
    pkg: wget
  roles:
    - pkgs
  tasks:
    - name: say hello
      debug:
        msg: "Hello World"
```

## ansible加接密

### 加密

```shell
# 创建测试文件
[root@pubserver ansible]# echo -e "ni hao china,\ni love you" > hello.txt
[root@pubserver ansible]#ansible-vault encrypt hello.txt
New Vault password: 
Confirm New Vault password: 
Encryption successful
```

### 解密-永久解开

```shell
[root@pubserver ansible]#ansible-vault decrypt hello.txt
Vault password: 
Decryption successful
```

### 解密-临时解开

```shell
[root@pubserver ansible]#ansible-vault view hello.txt
Vault password: 
ni hao china,
i love you
```

### 修改密码

```shell
[root@pubserver ansible]#ansible-vault rekey hello.txt
Vault password: 123456    # 旧密码
New Vault password: abcd  # 新密码
Confirm New Vault password: abcd
Rekey successful
```

### 高级使用

```shell
# 使用密码文件进行加解密
# 1. 将密码写入文件
[root@pubserver ansible]# echo 'yyh.cn' > pass.txt
# 2. 创建明文文件
[root@pubserver ansible]# echo 'hello world' > data.txt
# 3. 使用pass.txt中的内容作为密码加密文件
[root@pubserver ansible]# ansible-vault encrypt --vault-id=pass.txt data.txt
Encryption successful
[root@pubserver ansible]# cat data.txt    # 文件已加密
# 4. 使用pass.txt中的内容作为密码解密文件
[root@pubserver ansible]# ansible-vault decrypt --vault-id=pass.txt data.txt
Decryption successful
[root@pubserver ansible]# cat data.txt 
hello world
```

### playbook加密

```shell
# 针对剧本加密，未解密将不能执行
[root@pubserver ansible]#ansible-vault encrypt user_hang.yml # 加密剧本
New Vault password: 
Confirm New Vault password: 
Encryption successful

[root@pubserver ansible]# ansible-playbook user_hang.yml # 普通执行-报错
ERROR! Attempting to decrypt but no vault secrets found

# 加上一个参数 --ask-vault-password即可输入密码
[root@pubserver ansible]# ansible-playbook user_hang.yml --ask-vault-password
Vault password: 
```

## 特殊主机清单变量

### 准备前提

```shell
[root@pubserver ansible]# ansible all -m path -a "path=/root/.ssh/authorized_keys state=absent" # 删除所有主机的免密码

[root@pubserver ansible]#ssh web1
root@web1's password: 

[root@web1 ansible]#echo 'Port 220' >> /etc/ssh/sshd_config # 更改web1 ssh端口
[root@web1 ansible]#systemctl restart sshd
[root@pubserver ansible]#ssh web1
ssh: Could not resolve hostname web1:220: Name or service not known

[root@pubserver ansible]#ssh web1 -p220
root@web1's password: 
```

### 配置特殊变量

```shell
[root@pubserver ~]#mkdir myansible
[root@pubserver ~]#cd myansible
[root@pubserver myansible]#cp ~/ansible/ansible.cfg .
[root@pubserver myansible]#vim inventory # 编写新的配置清单和配置特殊变量
[group]
web1 ansible_ssh_user=root ansible_ssh_pass=a ansible_ssh_port=220
web2 ansible_ssh_user=root ansible_ssh_pass=a
db1  ansible_ssh_user=root ansible_ssh_pass=a

[root@pubserver myansible]#ansible all -m ping # 测试
```

# 快捷键

## 多行同时缩进

> 光标放到需要缩进的第一行
>
> crel + v
>
> 按住向下箭头到需要缩进的最后一行
>
> 大I
>
> 4个空格
>
> ESC

> 使用：第几行,$s/^/    /

# 问题



# 补充



# 今日总结



# 昨日复习
- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [语法格式](#语法格式)
  - [条件测试](#条件测试)
    - [测试变量](#测试变量)
    - [字符串测试](#字符串测试)
      - [== 等等于](#-等等于)
      - [!= 不等于](#-不等于)
      - [-z 为空](#-z-为空)
      - [! -z/-n 非空](#--z-n-非空)
    - [逻辑组合](#逻辑组合)
      - [\&\& 逻辑与](#-逻辑与)
      - [|| 逻辑或](#-逻辑或)
      - [\&\& ||](#-)
    - [数值测试](#数值测试)
      - [-eq 等于](#-eq-等于)
      - [-ne 不等于](#-ne-不等于)
      - [-gt 大于](#-gt-大于)
      - [-ge 大于等于](#-ge-大于等于)
      - [-le 小于](#-le-小于)
      - [-lt 小于等于](#-lt-小于等于)
    - [文件测试](#文件测试)
      - [-e 是否存在](#-e-是否存在)
      - [-f 是否存在](#-f-是否存在)
      - [-d 目录](#-d-目录)
      - [-r、-w、-x权限判断](#-r-w-x权限判断)
  - [IF分支](#if分支)
    - [单分支](#单分支)
    - [双分支](#双分支)
    - [多分支](#多分支)
  - [for循环](#for循环)
- [快捷键](#快捷键)
- [问题](#问题)
  - [列出常见的数值比较操作，并说明各自作用](#列出常见的数值比较操作并说明各自作用)
  - [运用条件测试操作，检查当前的用户是否为root](#运用条件测试操作检查当前的用户是否为root)
  - [绘图描述if双分支结构的执行流程](#绘图描述if双分支结构的执行流程)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)



# 学习目标

条件测试

If选择结构

循环结构

# 课堂笔记（命令）



# 课堂笔记（文本）

## 语法格式

> 1. [ 表达式 ]  
> 2. test 表达式
>
> 使用括号时，需要注意在条件两边加上空格

```
]# test abc == abc
]# echo $?
0
]# test abc == bca
]# echo $?
1

]# [ 123 == 12 ]
]# echo $?
1
]# [ 321 == 32 ]
]# echo $?
1
```

## 条件测试

### 测试变量

```
]#a=123
]#b=456
```

### 字符串测试

#### == 等等于

两个字符串是否相等

```
]# [ $a == $a ]
]# echo $?
0

]# [ $a == $b ]
]# echo $?
1

# 使$a与空($c)进行对比
]# [ $a == $c ] 
-bash: [: 123: 需要一元表达式
]# [ $a == "$c" ] 
]# echo $?
1
```

[注]：变量不存在时进行比较，需要加上引号

#### != 不等于

两个字符串不相等

```
]# [ $a != $a ]
]# echo $?
1

]# [ $a != $b ]
]# echo $?
0
```

#### -z 为空

是否为空

```
]# [ -z $a ]
]# echo $?
1

]# [ -z $c ]
]# echo $?
0
```

#### ! -z/-n 非空

是否非空

```
]# [ ! -z $a ]
]# echo $?
0

]# [ ! -z $c ]
]# echo $?
1

]# [ -n $a ]
]# echo $?
0
```

### 逻辑组合

#### && 逻辑与

A && B 仅当A命令执行成功，才执行B命令

```
]# [ a != b ] && echo ok
ok
]# [ a == b ] && echo ok
]# 
```

#### `||` 逻辑或

A && B 仅当A命令执行失败，才执行B命令

```
]# [ a != b ] || echo ok

]# [ a == b ] || echo ok
ok
```

#### && `||`

```
]#touch a b c
]#ls a  &&  ls b  &&  ls c  
a
b
c
]#ls a  ||  ls b  ||  ls c 
a
]#ls a  &&  ls b  ||  ls c     
a
b
]#ls a  ||  ls b  &&  ls c    
a
c
```

### 数值测试

#### -eq 等于

```
]# [ $a -eq $b ]
]# echo $?
1
```

#### -ne 不等于

```
]# [ $a -ne $b ]
]# echo $?
0
```

#### -gt 大于

```
]# [ $a -gt $b ]
[root@pc2 opt]# echo $?
1
```

#### -ge 大于等于

```
[root@pc2 opt]# [ $a -ge $b ]
[root@pc2 opt]# echo $?
1
```

#### -le 小于

```
]# [ $a -le $b ]
]# echo $?
0
```

#### -lt 小于等于

```
]# [ $a -lt $b ]
]# echo $?
0
```

### 文件测试

```
]#touch a
]#mkdir abc
```

#### -e 是否存在

判断文件是否存在，不关心文件类型

```
]# [ -e a ] 
]# echo $?
0

]# [ -e abc ] 
]# echo $?
0
```

#### -f 是否存在

判断文件是否存在，需关心是否是普通文件

```
]#[ -f a  ] 
]# echo $?
0

]# [ -f abc  ] 
]# echo $?
1
```

#### -d 目录

是否是目录

```
]# [ -d abc  ] 
]# echo $?
0

]# [ -d a  ] 
]# echo $?
1
```

#### -r、-w、-x权限判断

判断当前用户是否拥有此权限，对root用户无效

```
]# [ -r a  ] 
]# echo $?
0
```

## IF分支

不仅仅放入条件测试，还可以放命令，来实现判断

### 单分支

```
if 条件测试;then
命令1
...
fi
```

```
]#vim test1.sh
#!/bin/bash
if [ $USER -eq root ];then
ehco 您是管理员啊！
fi
]#bash test1.sh
您是管理员啊！
```

### 双分支

```
if 条件测试;then
命令1
....
else
命令n
fi
```

```
]#vim test2.sh
#!/bin/bash
if [ $USER -eq root ];then
ehco 您是管理员啊！
else
echo 您不是管理员
fi
]#bash test1.sh
您是管理员啊！
]#su -yyh
]$bash test2.sh
您不是管理员
```

### 多分支

```
if 条件测试;then
    命令1
    ....
elif 条件测试2 ;then
	命令2
	......
elif 条件测试3;then
	命令3
	......
else 
	.....
fi
```

```
#!/bin/bash
read -p "this month exams source? " x
if [ $x -ge 90 ] && [ $x -le 100 ];then
    echo A
elif [ $x -ge 80 ] && [ $x -le 89 ];then
    echo B
elif [ $x -ge 0 ] && [ $x -le 79 ];then
    echo C
else
    echo "enter error"
fi
```

## for循环

```
for 变量名 in 值1 值2 值3 # 值的数量决定循环的次数
do
	命令序列
done
```

```
#!/bin/bash  
for i in 1 2 3
do
	echo $i 
done
执行结果：
1
2
3
```

> {1..100} # 生成1-100，在循环中不支持变量;用于已经知道循环的次数
>
> seq 10 # 生成1-10，支持变量;未知循环的次数

```
#!/bin/bash
a=10
for i in $(seq $a)
do
        echo $i
done


#!/bin/bash
for i in {1..10}
do
        echo $i
done

# C语言格式for循环
#!/bin/bash
for((i=1;i<=5;i++))
do
	echo abc
done
```



**案例**

1. 批量创建用户

   ```
   vim name.txt
   dcc
   xtt
   yyh
   haha
   vim test.sh
   #!/bin/bash
   for i in `cat name.txt`
   do
   	useradd $i
   done
   ```

2. 批量ping IP

   ```
   #!/bin/bash
   for i in {1..10}
   do
       ping -c 3 -i 0.2 -W 1 192.168.88.$i &> /dev/null
       if [ $? -eq 0 ];then
           echo 88.$i successful
       else
           echo 88.$i failed
       fi
   done    
   ```

   

# 快捷键



# 问题

## 列出常见的数值比较操作，并说明各自作用

> -eq 相等
>
> -ne 不等
>
> -gt 大于
>
> -ge 大等
>
> -lt 小于
>
> -le 小等

## 运用条件测试操作，检查当前的用户是否为root

> #!/bin/bash
>
> if [ $USER -eq root ];then
>
> ​	echo “hello”$USER
>
> else
>
> ​	echo "bye bye"$USER
>
> fi

# 补充

# 今日总结

# 昨日复习

uaddfor.sh:行9: 请选择你要操作的项目：: 未找到命令
uaddfor.sh:行26: 未预期的符号 `elif' 附近有语法错误
uaddfor.sh:行26: `elif [ $option -eq 3 ];then'


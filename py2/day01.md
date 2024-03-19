- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [shutil模块](#shutil模块)
    - [基本概念](#基本概念)
    - [主要方法](#主要方法)
      - [复制和移动](#复制和移动)
      - [目录操作](#目录操作)
      - [权限管理](#权限管理)
  - [OS模块](#os模块)
    - [基本概述](#基本概述)
    - [主要方法](#主要方法-1)
  - [subprocess模块](#subprocess模块)
    - [基本概念](#基本概念-1)
    - [主要方法](#主要方法-2)
      - [run方法](#run方法)
      - [run返回值](#run返回值)
    - [练习1：ping函数](#练习1ping函数)
  - [异常](#异常)
    - [基本概念](#基本概念-2)
    - [常见异常](#常见异常)
    - [错误类型捕获](#错误类型捕获)
      - [try-except](#try-except)
      - [多个try-except](#多个try-except)
    - [捕获未知错误](#捕获未知错误)
    - [异常参数e](#异常参数e)
      - [查看异常信息](#查看异常信息)
      - [捕获多个异常](#捕获多个异常)
    - [else子句](#else子句)
    - [finally子句](#finally子句)
  - [自定义异常](#自定义异常)
    - [基本概述](#基本概述-1)
    - [基本用法](#基本用法)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)

# 学习目标

系统模块管理

异常处理

# 课堂笔记（命令）

# 课堂笔记（文本）

## shutil模块

### 基本概念

> - `shutil` 可以简单地理解为 `sh + util`，**shell 工具** 的意思
> - `shutil` 模块是对 `os` 模块的补充，主要针对文件的 **拷贝、删除、移动、压缩和解压** 操作

### 主要方法

#### 复制和移动

```python
"""
shutil.copyfileobj(源文件对象,目标文件对象)
文件对象：使用open将文件打开后，此是称为文件对象
文件准备：在当前工程下创建一个a和b目录，将hosts文件拷贝到a目录
[注]：只能复制文件内容，不能复制权限
"""
import shutil

fs = open("./a/hosts", "r")
fd = open("./b/myhosts", "w")
shutil.copyfileobj(fs, fd)
fs.close()
fd.close()

"""
shutil.copyfile(源文件路径，目标文件路径)
只需要提供文件所在的路径即可完成复制
[注]：只能复制文件内容，不能复制权限
"""
shutil.copyfile("/usr/bin/ls", "./b/myls")



"""
shutil.copy(源文件路径,目标文件路径)
只需要提供文件所在的路径即可完成复制
[注]：既能复制文件的内容，也能复制权限
"""
shutil.copy("/usr/bin/ls", "./a/myls")


"""
shutil.move(源文件或目录,源文件或目录)
递归地将文件或目录(src)，剪切到另一个位置(dst),并返回目标
"""
dst = shutil.move("./b", "./a")
print(dst) # ./a/b
dst1 = shutil.move("./a/b", "./")
print(dst1) # ./b
```

#### 目录操作

```python
import shutil
"""
shutil.copytree(源目录，目标目录)
递归地复制以源目录为根的整个目录树，返回目标目录。
目标目录不能已经存在
"""
dst2 = shutil.copytree("./b", "./c") # 拷贝b目录到当前目录下取名为c
print(dst2) #  ./c
dst3 = shutil.copytree("./b", "./a") # 拷贝b目录到a目录下将报错
print(dst3) # FileExistsError: [Errno 17] File exists: './a'

"""
shutil.rmtree(路径)
删除整个目录树； 路径 必须指向目录
空目录或者非空目录都可使用
"""
shutil.rmtree("./c")
```

#### 权限管理

```python
"""
shutil.copymode(源文件路径，源文件路径)
将权限位从 源文件路径 复制到 源文件路径
文件内容，所有者和组不受影响
src 和 dst 是以字符串形式给出的路径名称
"""
shutil.copymode("/usr/bin/ls", "./b/myls")

-rw-rw-r-- 1 student student  776  3月 15 09:46 myhosts
-rw-rw-r-- 1 student student 147K  3月 15 09:51 myls # 改变前
[student@server1 day06]$ ls -lh b/
总用量 152K
-rw-rw-r-- 1 student student  776  3月 15 09:46 myhosts
-rwxr-xr-x 1 student student 147K  3月 15 09:51 myls # 改变后
    
    
"""
shutil.chown(文件路径,所有者，所属组)
更改给定 路径 的所有者 用户 和 组
"""
shutil.chown("./b/myls", user="nobody", group="nobody")    
    
# 改变后    
-rw-rw-r-- 1 student student  776  3月 15 09:46 myhosts
-rwxr-xr-x 1 nobody  nobody  147K  3月 15 09:51 myls
```

## OS模块

### 基本概述

> - 对文件系统的访问大多通过 python 的 os 模块实现
> - 该模块是 python 访问操作系统功能的主要接口
> - 有些方法，如：**copy** 等，并没有提供，可以使用 **shutil** 模块作为补充

### 主要方法

```python
import os

def use_os():
    print(os.getcwd()) # 返回当前所在的文件路径,类似于: pwd
    print(os.listdir()) # 查看当前目录下的所有文件(包括隐藏文件)，类似于：ls -a
    print(os.listdir("/opt")) # 查看/opt目录下的内容，类似于：ls /opt
    os.mkdir("c") # 当前路径下创建目录，类似于：mkdir ./c
    os.mkdir("/tmp/demo/") # 指定路径下创建目录，只能创建单级目录
    os.makedirs("/tmp/a/b/c") # 创建目录时，父目录不存在，会自动创建，类似于: mkdir -p

    os.chdir("/tmp/demo") # 切换当前所处的文件位置，类似于：cd /tmp/demo
    print(os.getcwd()) # 获取切换后的当前路径
    print(os.listdir()) # 查看当前目录下的文件
    os.makedirs("/tmp/demo/a/b") # 创建目录用于测试
    os.remove("c") # 只能删除单个文件，不能删除目录
    os.rmdir("a") # 只能删除空目录；不能删除目录树,要删除非空目录要使用shutil.rmtree()


    print(os.path.isabs("demo/a")) # 判断是否为绝对路径；不管文件是否存在,是True，不是False
    print(os.path.isdir("/tmp/demo")) # 判断是否是目录（文件必须存在，且为目录），是True，不是False
    print(os.path.isfile("/tmp/demo/cb"))# 判断是否为文件（文件必须存在，且为文件），是True，不是False
    print(os.path.islink("/usr/bin/dnf")) # 判断是否为链接文件（文件必须存在，且为连接文件）
    print(os.path.exists("/etc/hostname")) # 判断文件或目录是否存在


    print(os.path.basename("/etc/nginx/nginx.conf")) # 获取最右边'/'，右边的数据‘nginx.conf’
    print(os.path.dirname("/etc/nginx/nginx.conf")) # 获取最右边'/'，左边的数据'/etc/nginx'
    print(os.path.split("/tmp/demo/a")) # 路径切割，从最右边'/'开始，进行切割 ('/tmp/demo', 'a')
    print(os.path.join("/tmp/demo","a/b/c")) # 路径的拼接,将右边路径拼接起来   /tmp/demo/a/b/c

    

if __name__ == '__main__':
    use_os()
```

## subprocess模块

### 基本概念

> - subprocess 模块主要用于执行 **系统命令**
> - subprocess 模块允许你产生新的进程，并获得它们的返回状态
> - 通俗地说就是通过这个模块，你可以在 Python 的代码里执行操作系统级别的命令，比如 **"ifconfig" 、"du -sh"** 等等

### 主要方法

#### run方法

```python
import subprocess

"""
subprocess.run(args)
执行 args 参数所表示的命令，等待命令结束，并返回一个 CompletedProcess 类型对象
"""
# 第一种方法
subprocess.run(['ls', '/home']) # 将命令和操作的目录写道列表中，执行Linux命令
subprocess.run(["echo", "$HOME"]) #  $HOME 针对特殊的命令将实现不了
# 第二种方法
subprocess.run('ls /home', shell=True) # 以字符串形式执行，指定shell解释器
subprocess.run('echo $HOME', shell=True) # /home/student
```

#### run返回值

```python
"""
run返回值
run执行命令后的返回值是：CompletedProcess(args='执行的命令', returncode=0或其他)
os.listdir('./') 执行后返回是一个列表数据
"""
subprocess.run('ls ..', shell=True) # 返回上级目录中所有内容
result = subprocess.run('ls .. &> /dev/null', shell=True) # 将内容存储在变量中
print(result) # CompletedProcess(args='ls ..', returncode=0)
print(result.args) # 获取result中args列表内容
print(result.returncode) # returncode为返回码，0是成功其余是失败
print(f"你运行的命令是：{result.args}\n返回码是：{result.returncode}")
```

> - subprocess 模块虽然可以支持所有的 linux 命令，但不可乱用，放弃其他的模块
> - subprocess 模块编写的代码，**不具有跨平台性，不能在 windows，mac 等系统使用**

### 练习1：ping函数

```python
"""
需求
调用 ping 命令
编写 ping 函数
用于测试远程主机的联通性
ping 通显示：x.x.x.x:up
ping 不通显示：x.x.x.x:down
"""

def ping(host):
    # -c 3发送3个包   -i 0.2 每隔多少秒发一个包 -W 1 超过1s将发下一个包
    result = subprocess.run(f'ping -c 3 -i 0.2 -W 1 {host} &> /dev/null', shell=True) # &> 输出到黑洞，不将输出信息先是到屏幕
    if result.returncode == 0:  # 通过判断上面命令执行是否成功来输出
        print(f"{host}  is up")
    else:
        print(f"{host}  is down")
ping("www.baidu.com")
ping("www.google.com")
```

## 异常

### 基本概念

> 程序在运行时，如果 `Python 解释器` **遇到** 到一个错误，**会停止程序的执行，并且提示一些错误信息**，这就是 **异常**
>
> 异常是因为程序出现了错误，而在正常控制流以外采取的行为
>
> - 这个行为又分为两个阶段：
>   - 首先是引起异常发生的错误
>   - 然后是检测（和采取可能的措施）阶段

### 常见异常

```python
"""
常见异常
类型错误：TypeError:
索引错误：IndexError:
语法错误：SyntaxError:
强制终止程序：KeyboardInterrupt
值错误：ValueError
ctrl+d:EOFError
等等
"""
# c = 'a' + 5 # TypeError: can only concatenate str (not "int") to str

a = [1, 2, 1]
# print(a[4]) # IndexError: list index out of range

# if a.pop() = 10: # SyntaxError:
```

### 错误类型捕获

> - 在程序执行时，可能会遇到 **不同类型的异常**，并且需要 **针对不同类型的异常，做出不同的响应**，这个时候，就需要捕获错误类型了

```python
"""
语法格式：
"""
try:
    # 可能会出错的代码
    pass
except 错误类型1:
    # 针对错误类型1，对应的代码处理
    pass
except (错误类型2, 错误类型3):
    # 针对错误类型2 和 3，对应的代码处理
    pass
except Exception as result:
    print("未知错误 %s" % result)
```

#### try-except

```python
"""
try-except语句
"""
try:
    lists = [1, 2, 3, 4]
    print(lists[5])
except IndexError:
    print("索引超出列表长度，请检查后再操作") #  索引超出列表长度，请检查后再操作

# 不适用异常捕获
#lists = [1, 2, 3, 4]
#print(lists[5]) # IndexError: list index out of range
```

#### 多个try-except

```python
"""
多个try-except使用
"""
try:
    num = int(input("Please enter Num："))
    print(f"what you entered is {num}")
except ValueError:
    print("input error only enter int type!")
except KeyboardInterrupt:
    print("\nyou kill program")
except EOFError:
    print("not input")    
```

### 捕获未知错误

> - 在开发时，**要预判到所有可能出现的错误**，还是有一定难度的
> - 如果希望程序 **无论出现任何错误**，都不会因为 `Python` 解释器 **抛出异常而被终止**，可以再增加一个 `except`

```python
"""
使用基本语法
"""
except Exception as result:
    print("未知错误 %s" % result)
```

```python
"""
捕获未知异常错误
"""
try:
    a = {'name': 'coke', 'age': 12}
    strs = input("input query key：")
    print(a[strs]) 
except ValueError:
    print("input value error")
except KeyboardInterrupt:
    print("\nyou kill program")
except EOFError:
    print("not input")
except Exception:
    print("unknown error")

# 测试输入字典a不存在的键（abc），会报找不到键的错误，但我们不知道这个错误，就可以使用Exception这个错误的父类来接收，接收所有未提及的错误  
```

### 异常参数e

> - 异常也可以有参数，异常引发后它会被传递给异常处理器
> - 当异常被引发后参数是作为附加帮助信息传递给异常处理器的
> - 需要查看异常信息，但不能是系统报错

#### 查看异常信息

```python
"""
使用异常参数e
打印异常信息
"""
try:
    num = int(input("Please enter Num："))
    print(f"what you entered is {num}")
except ValueError as e:
    print("input error only enter int type!")
    print(f"abnormal info：{e}") # 输入avc
    # abnormal info：invalid literal for int() with base 10: 'sad'
except KeyboardInterrupt:
    print("\nyou kill program")
except EOFError:
    print("not input")
```

#### 捕获多个异常

```python
"""
将多个异常放在一起进行相同的处理
"""
try:
    num = int(input("Please enter a number to calculate："))
    result = 5 / num
    print(f"result：{result}")
except ValueError as e:
    print("Enter error")
    print(f"abnormal info：{e}")
except ZeroDivisionError as e:
    print("The divisor cannot be 0")
    print(f"abnormal info：{e}")
except (EOFError, KeyboardInterrupt):
    print("reenter")

# 除数为0测试
# 除数为abc测试
# 终止程序测试、ctrl+d测试
```

### else子句

> - 在 try 范围中没有异常被检测到时，执行 else 子句
> - 在else 范围中的任何代码运行前，try 中的所有代码必须完全成功

```python
"""
else子句
当try-except中没有检测到异常时执行else子句
"""
try:
    num = int(input("Please enter a number to calculate："))
    result = 5 / num
    print(f"result：{result}")
except ValueError as e:
    print("Enter error")
    print(f"abnormal info：{e}")
except ZeroDivisionError as e:
    print("The divisor cannot be 0")
    print(f"abnormal info：{e}")
except (EOFError, KeyboardInterrupt):
    print("reenter")
else: # 当上方没有出现异常执行的代码
    print('-'*20)
    print(f"Enter the number as {num}")
    print("You are greate！")

# 执行代码后输入 2  
Please enter a number to calculate：2
result：2.5
--------------------
Enter the number as 2
You are greate！
```

### finally子句

> - finally 子句是 **无论异常是否发生，是否捕捉都会执行的一段代码**
> - 如果打开文件后，因为发生异常导致文件没有关闭，可能会发生数据损坏，**使用finally 可以保证文件总是能正常的关闭**

```python
"""
finally子句
无论是否异常是否发生，都会执行finally子句
"""
try:
    num = int(input("Please enter a number to calculate："))
    result = 5 / num
    print(f"result：{result}")
except ValueError as e:
    print("Enter error")
    print(f"abnormal info：{e}")
except ZeroDivisionError as e:
    print("The divisor cannot be 0")
    print(f"abnormal info：{e}")
except (EOFError, KeyboardInterrupt):
    print("reenter")
else:
    print('-'*20)
    print(f"Enter the number as {num}")
    print("You are greate！")
finally: # 无论是否发生异常都执行的代码
    print("-"*20)
    print("anyway run")

# 第一次测试输入1 
Please enter a number to calculate：1
result：5.0
--------------------
Enter the number as 1
You are greate！
--------------------
anyway run

# 第二次测试输入abc
Please enter a number to calculate：abc
Enter error
abnormal info：invalid literal for int() with base 10: 'abc'
--------------------
anyway run
```

## 自定义异常

### 基本概述

> - 在开发中，除了 **代码执行出错** `Python` 解释器会 **抛出** 异常之外
> - 还可以根据 **应用程序** **特有的业务需求** **主动抛出异常**

### 基本用法

> 1. **创建** 一个 `Exception` 的 **对象**
> 2. 使用 `raise` **关键字** 抛出 **异常对象**

```python
"""
自定义异常
"""
def input_pwd():
    pwds = input("please passwd：")
    if len(pwds) >= 8:
        return pwds
    raise Exception("密码长度不够") # Exception: 密码长度不够
print(input_pwd())
```



# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习
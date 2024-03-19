- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [创建函数](#创建函数)
    - [def语句](#def语句)
    - [前向引用](#前向引用)
    - [函数调用](#函数调用)
    - [关键字参数](#关键字参数)
    - [练习1：随机加减法](#练习1随机加减法)
  - [匿名函数](#匿名函数)
    - [概述](#概述)
    - [方法使用](#方法使用)
  - [变量作用域](#变量作用域)
    - [全局变量](#全局变量)
    - [局部变量](#局部变量)
    - [global语句](#global语句)
  - [生成器](#生成器)
    - [概述](#概述-1)
    - [生成器函数](#生成器函数)
    - [生成器表达式](#生成器表达式)
    - [练习2：文件生成器](#练习2文件生成器)
  - [模块详解](#模块详解)
    - [概述](#概述-2)
    - [作用](#作用)
    - [模块文件](#模块文件)
    - [搜索路径](#搜索路径)
    - [导入模块](#导入模块)
    - [hashlib模块](#hashlib模块)
    - [练习3：计算文件md5值](#练习3计算文件md5值)
    - [tarfile模块](#tarfile模块)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)


# 学习目标

函数进阶

函数的高级应用

模块

# 课堂笔记（命令）

# 课堂笔记（文本）

## 创建函数

### def语句

> - 标题行由 def 关键字、函数的名字，以及参数的集合（如果有的话）组成
> - def 子句的剩余部分包括了一个虽然可选但是强烈推荐的文档字串，和必需的函数体

### 前向引用

> 函数不允许在函数未声明之前，对其引用或在调用

```python
"""
前向引用
"""
def func():
    print("12")

# func1() # NameError: name 'func1' is not defined. Did you mean: 'func'?
def func1():
    print("321")
func()
```

> [注]：定义函数时，函数的先后顺序不重要，函数在什么位置被调用才重要

### 函数调用

> - 使用一对圆括号() 调用函数，如果没有圆括号，只是对函数的引用
> - 任何输入的参数都是必须放置在括号中

```python
def func1():
    print("321")
print(func1) # <function func1 at 0x7f952244ab00>  位置对象
func1() #321                调用函数时，函数名必须有小括号，否则返回一个位置对象
```

### 关键字参数

> - 关键字参数的概念仅仅针对函数的调用
> - 这种理念是 **让调用者通过函数调用中的参数名字来区分参数**
> - 这种规范允许参数不按顺序
> - **位置参数应写在关键字参数前面**

```python
"""
关键字参数的传递
"""
def stu_info(name, age=20):
    print(f"我的姓名是：{name}  年龄是：{age}")
# 按照位置进行传递参数
stu_info("可乐", 20) # 我的姓名是：可乐  年龄是：20
stu_info(20, "宝器") # 我的姓名是：20  年龄是：宝器
# 按照关键字传参
stu_info(name="张三", age=18) # 我的姓名是：张三  年龄是：18
stu_info(age=18, name="李四") # 我的姓名是：李四  年龄是：18
# 注：关键字参数后不能跟有位置参数
stu_info(name="校长", 18) # SyntaxError: positional argument follows keyword argument
```

### 练习1：随机加减法

```python
"""
练习1：
需求
随机生成两个100以内的数字
随机选择加法或是减法
总是使用大的数字减去小的数字
"""
import random

def cale():
    operator_ = ['+', '-']
    random_num = [random.randint(1, 101) for i in range(2)]
    random_num.sort(reverse=True)
    if random.choice(operator_) == '+':
        values = random_num[0] + random_num[1]
    elif random.choice(operator_) == '-':
        values = random_num[0] - random_num[1]
    answer = int(input("%d %s %d = ?? 请作答"%(random_num[0],random.choice(operator_),random_num[1])))
    if answer == values:
        print("恭喜你答对了")
    else:
        print("猜错了")
```

## 匿名函数

### 概述

> - python 允许用 lambda 关键字创造匿名函数
> - 匿名是因为不需要以标准的 def 方式来声明
> - 一个完整的 lambda "语句"代表了一个表达式，这个表达式定义体必须和声明放在同一行
> - 匿名函数体中只有一行代码，是return形式，同时省略return关键字

### 方法使用

```python
"""
匿名函数
"""
# 编写一个函数，要求用户输入两个数返回和的结果
def add(n, m):
    return n + m
print(add(10, 20))
# 使用匿名函数进行封装
adds = lambda n,m:n+m # 显示调用不推荐
print(adds(50, 60))

# 判断该数是偶数还是奇数;lambad 型参:True if 表达式 else False
event_odd = lambda num:print("是偶数") if num % 2 == 0 else print("不是偶数")
event_odd(38)


# 使用匿名函数，传递任意三个数字，返回平均值
avg = lambda a,b,c:(a+b+c)//3
print(avg(80, 20, 24)) # 41
# 或者
print((lambda a,b,c:(a+b+c)//3)(3, 15, 9)) # 9
```

## 变量作用域

### 全局变量

> - 标识符的作用域是定义为其声明在程序里的可应用范围，也就是变量的可见性
> - 在一个模块中最高级别的变量有全局作用域
> - 全局变量的一个特征是除非被删除掉，否则他们会存活到脚本运行结束，且对于所有的函数，他们的值都是可以被访问的

```python
num = 10
def a():
    print(num)

def b():
    print(num)
print(num) # 10
a() # 10
b() # 10
```

### 局部变量

> - 局部变量只是暂时的存在，仅仅只依赖于定义他们的函数现阶段是否处于活动
> - 当一个函数调用出现时，其局部变量就进入声明它们的作用域。在那一刻，一个新的局部变量名为那个对象创建了
> - 一旦函数完成，框架被释放，变量将会离开作用域

```python
num = 10
def a():
    nums = 20 # 局部变量，只作用于当前a函数体中有效
    print(nums, num)

def b():
    print(nums, num)
print(num) # 10
print(nums) # name 'nums' is not defined. Did you mean: 'num'?
a() # 10
b() # 10
```

```python
# 当前全局变量与局部变量名称相同时，局部变量的优先级高于全局变量

num = 10
def a():
    num = 20
    print(num)

def b():
    print(num)
a() # 20 # 打印局部变量num的值
b() # 10 # 打印全局变量num的值
print(num) # 10
```

### global语句

> - 因为全局变量的名字能被局部变量给遮盖掉
> - 为了明确地引用一个已命名的全局变量，必须使用 global 语句

```python
num = 10 # 全局变量
def a(): 
    global num # 声明num为全局变量
    num = 20  # 修改全局变量num的值
    print(num)  # 输出20
a() # 20
print(num) # 20
```

> [注]：在函数内部使用`global`语句声明的变量在全局作用域中不存在，会引发`NameError`异常。另外，`global`语句只能在函数内部使用，不能在全局作用域中使用。

## 生成器

### 概述

> Python 使用生成器对延迟操作提供了支持。所谓延迟操作，是指在需要的时候才产生结果，而不是立即产生结果。这也是生成器的主要好处。

### 生成器函数

> - 常规函数定义，但是，**使用 yield 语句而不是 return 语句返回结果**
> - yield 语句一次返回一个结果，在每个结果中间，挂起函数的状态，以便下次重它离开的地方继续执行
> - 当调用生成器函数时，它会返回一个迭代器（iterator），我们可以使用迭代器来逐个获取生成器函数中 `yield` 语句产生的值。每次调用迭代器的 `next()` 方法或者使用 `for` 循环时，生成器函数会从上次暂停的地方继续执行，直到遇到下一个 `yield` 语句或函数结束

```python
def fun1():
    yield 1
    yield 4
    yield [1, 3]
    yield (5, 6)
gen1 = fun1()
print(gen1.__next__()) # 1
print(gen1.__next__()) # 4
print(gen1.__next__()) # [1, 3]
for i in gen1:
    print(i) # 根据上方输出得知，生成器函数的值停留在[1,3]，当再次调用时将接着停留值后继续输出
print(gen1.__next__()) # 报错 StopIteration；因为生成器中已经没有结果了
```

> 生成器函数在 Python 中非常有用，特别是处理大量数据时，可以节省内存空间并提高效率。通过 `yield`，我们可以实现惰性计算，只有在需要时才计算和生成值。

### 生成器表达式

> - 类似于列表推导，但是，生成器返回按需产生结果的一个对象，而不是一次构建一个结果列表

```python
# 常用的列表推导式
gen2 = [random.randrange(1, 20) for i in range(2)]
print(gen2) # [7, 6] 随机生成1,20之间的两个数，并返回到列表中
# 生成器推导式
gen3 = (random.randrange(1, 20) for i in range(2))
print(gen3) # <generator object <genexpr> at 0x7f9a20db2ce0> 返回一个可迭代对象
print(gen3.__next__()) # 12
print(gen3.__next__()) # 1
print(gen3.__next__()) # 报错 StopIteration；因为生成器结果只有两个值,上方已经将值取完了
```

### 练习2：文件生成器

```python
# 生成数据
with open("num.txt", "a") as f:
    for i in range(51):
        date = time.strftime('%H:%M:%S')
        txt = f"{date} 第{i}个数已经写入"
        f.write(txt + '\n')
        time.sleep(0.1)
```

```python
"""
文件生成器练习
使用函数实现生成器   yield
函数接受一个文件路径作为参数(读文件)
生成器函数每次返回文件的 10 行数据
"""
def file_yield(path):
    data = [] # 存储行数据
    with open(path, "r") as f: 
        for item in f.readlines(): # 循环遍历打印文件中所有行
            data.append(item.strip()) # 添加到data列表中
            if len(data) == 10: # 当长度等于10时
                yield data # 使用生成器进行返回
                data.clear() # 清空列表
    if len(data) != 0: # 当列表长度小于10时的处理
        yield data # 将剩余列表返回     
if __name__ == '__main__':
    gen4 = file_yield('./num.txt')
    for i in gen4: # 通过循环遍历取获取数据
        print(i)
        
        
# 使用while循环
def file_yield(path):
    with open(path, "r") as f:
        data = []
        line = f.readline()
        while line:
            data.append(line.strip())
            if len(data) == 10:
                yield data
                data.clear()
            line = f.readline()
        if data:
            yield data
if __name__ == '__main__':
    gen4 = file_yield('./num.txt')
    for i in gen4: # 通过循环遍历取获取数据
        print(i)            
```

## 模块详解

### 概述

> - 模块支持从逻辑上组织 python 代码
> - 当代码量变得非常大的时候，最好把代码分成一些有组织的代码段
> - 代码片段相互间有一定的联系，可能是一个包含数据成员和方法的类，也可能是一组相关但彼此独立的操作函数
> - 这些代码段是共享的，所有 python 允许 “调入” 一个模块，允许使用其他模块的属性来利用之前的工作成果，实现代码重用

### 作用

> - 模块可以实现代码的重用，导入模块，就可以使用模块中已经定义好的类，函数和变量，**减少代码的冗余性**

### 模块文件

> - 模块是从逻辑来组织 python 代码的方法，文件是物理层上组织模块的方法
> - 一个文件被看作是一个独立模块，一个模块也可以被看作是一个文件
> - 模块的文件名就是模块的名字加上扩展名 **.py**

### 搜索路径

> - 模块的导入需要一个叫做 “路径搜索” 的过程
> - python 在文件系统 “预定义区域” 中查找要调用的模块
> - 搜索路径在 sys.path 中定义
> - 也可以通过 PYTHONPATH 环境变量引入自定义目录

### 导入模块

```python
import sys
print(sys.path) # 模块在导入时，会一层一层搜索路径的目录中进行查询，匹配即停止

['/home/student/PycharmProjects/pythonProject1/day07', '/home/student/PycharmProjects/pythonProject1', '/usr/lib64/python310.zip', '/usr/lib64/python3.10', '/usr/lib64/python3.10/lib-dynload', '/home/student/.local/lib/python3.10/site-packages', '/usr/local/lib64/python3.10/site-packages', '/usr/local/lib/python3.10/site-packages', '/usr/lib64/python3.10/site-packages', '/usr/lib/python3.10/site-packages']


# 设置搜索路径的方法
sys.path.apend(路径地址)
```

### hashlib模块

> - hashlib 用来替换 MD5 和 sha 模块，并使他们的API一致，专门提供hash算法
> - 包括md5、sha1、sha224、sha256、sha384、sha512，使用非常简单、方便

```python
import hashlib
m = hashlib.md5(b'123456') # 一次性读取所有数据，适合于小文件数据
print(m.hexdigest()) # e10adc3949ba59abbe56e057f20f883e

# 分段加入
n = hashlib.md5() # 分段读取，适合大文件的数据读取
n.update(b'12')
n.update(b'34')
n.update(b'56')
print(n.hexdigest()) #e10adc3949ba59abbe56e057f20f883e
```

### 练习3：计算文件md5值

```python
"""
练习
编写用于计算文件 md5 值的脚本
文件名通过位置参数获得
打印出文件 md5 值
"""
import hashlib
def check_md5(path):
    with open(path, "rb")as f:
        md5s = hashlib.md5()
        n = 0
        while True:
            data = f.readline(4048)
            md5s.update(data)
            if len(data) == 0:
                break
            n += 1
            print(f"第{n}次的md5值为：{md5s.hexdigest()}")
check_md5("num.txt") # f52d88a6f5042204d5ce1ec2d693e9e4 

# 使用linux命令md5sum计算一下
]#md5sum num.txt
f52d88a6f5042204d5ce1ec2d693e9e4  num.txt        
```

### tarfile模块

> - tarfile模块允许创建、访问 tar 文件
> - 同时支持 gzip、bzip2 格式

```python
# 创建tar包
import tarfile
tar = tarfile.open("test.tar,gz","w:gz") # 在当前目录下再创建test.tar.gz文件不存在则自动创建
tar.add("/etc/hosts") # 向压缩包中添加文件
tar.add("/etc/passwd")
tar.close() # 关闭文件   test.tar.gz

# 解压tar包
tars = tarfile.open("test.tar,gz") # 打开文件
tars.extractall(path="./") # 指定解压路径，不指定默认在当前目录下解压
tars.close() # 关闭文件
```



# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习
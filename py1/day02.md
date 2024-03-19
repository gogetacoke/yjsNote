- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [判断语句](#判断语句)
    - [if语句语法结构](#if语句语法结构)
    - [练习1：账密判断](#练习1账密判断)
    - [练习2：成绩判定](#练习2成绩判定)
    - [练习3：石头剪刀布](#练习3石头剪刀布)
  - [while循环](#while循环)
    - [while循环语法结构](#while循环语法结构)
    - [练习1：打印5编hello world](#练习1打印5编hello-world)
    - [练习2：完善石头剪刀布游戏](#练习2完善石头剪刀布游戏)
    - [break、continue、else](#breakcontinueelse)
      - [break](#break)
      - [continue](#continue)
      - [else](#else)
    - [练习3：猜数字游戏](#练习3猜数字游戏)
  - [for循环详解](#for循环详解)
    - [for循环语法结构](#for循环语法结构)
      - [练习1：循环打印内容](#练习1循环打印内容)
      - [练习2：计算1+100](#练习2计算1100)
    - [遍历及迭代器](#遍历及迭代器)
    - [range函数](#range函数)
      - [语法结构](#语法结构)
      - [基本使用](#基本使用)
    - [列表推导式](#列表推导式)
      - [练习3：生成1-10列表](#练习3生成1-10列表)
    - [斐波那契数列](#斐波那契数列)
      - [练习：完成1](#练习完成1)
      - [练习：完成2](#练习完成2)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)


# 学习目标

判断语句

while循环

for循环

# 课堂笔记（命令）

# 课堂笔记（文本）

## 判断语句

> - 如果 **条件满足**，才能做某件事情，
> - 如果 **条件不满足**，就做另外一件事情，或者什么也不做
> - **判断语句** 又被称为 “分支语句”，正是因为有了判断，才让程序有了很多的分支

### if语句语法结构

> - 标准if条件语句的语法    缩进：python代码的层级关系，一般加4个空格
>
>   ```python
>   # 单分支
>   if 表达式:
>       if_suite
>   # 双分支
>   if 表达式:
>       if_suite
>   else:
>       else_suite
>   # 多分支    
>   if 表达式1:
>       if_suite
>   elif 表达式2：
>   	if_suite
>   else:
>       elss_suite
>   ```
>
> - 如果表达式的值 **非0** 或者为布尔值 **True**，则代码组 **if_suite** 被执行；否则就去执行 **else_suite**
>
> - 代码组是一个 **python术语**，它由一条或多条语句组成，表示一个子代码块
>
> 只要表达式数字为 **非零值** 即为 **True**
>
> **空字符串、空列表**的值均为 **False**

```python
# 单分支：根据表达式是否是true还是false
if 3 > 0:
    print('yes')
print('OK')  # 输出：yes，no；3>0条件成立，输出yes，程序执行是从上到下逐行执行的，判断执行完后再执行基础输出

# 双分支；为0的数字都是False	
if -0.0:
    print("Fasle")
else：
	print("值为0")  # 输出：值为0
# 空格则为空字符串，为True
if ' ':
    print("空格也是True")
# 空列表也是True;[False]为列表，列表中元素是False，因为列表存在，为真会打印；
if [False]: 
    print("非空列表，为True")
# None 为假，取反为真
if not None:
    print("None为假，取反为真")
```

### 练习1：账密判断

```python
"""
练习1：判断用户输入
提示用户输入用户名和密码
获得到相关信息后，将其保存在变量中
如果用户输的用户名为 bob，密码为 123456，则输出 Login successful，否则输出 Login incorrect
"""
# 提示用户输入用户名和密码
username = input("请输入用户名：")
password = input("请输入密码：")

# 判断用户输入的用户名和密码是否正确
if username == "bob" and password == "123456":
    print("Login successful")
else:
    print("Login incorrect")
```

### 练习2：成绩判定

```python
"""
练习2：判断成绩
如果成绩大于60分，输出“及格”
如果成绩大于70分，输出“良”
如果成绩大于80分，输出“好”
如果成绩大于90分，输出“优秀”
否则输出“你要努力了”
"""
score_input = float(input("请输入成绩："))
if score_input > 90:
    print("优秀")
elif 80 < score_input < 90:
    print("好")
elif 70 < score_input < 80:
    print("良")
elif 60 < score_input < 70:
    print("及格")
else:
    print("你要努力了！！")
```

### 练习3：石头剪刀布

```python
"""
练习3：猜拳游戏
计算机随机出拳
玩家自己决定如何出拳
代码尽量简化
"""
import random
# 定义列表存储出拳的每个选择
all_chioce = ['石头', '剪刀', '布']
# 使用random模块中的choice从列表中随机取出一个元素
computer_choice = random.choice(all_chioce)
# 获取用户输入
user_input = input("请出拳（石头、剪刀、布）：")
# 判断用户输入的值是否合法
if  user_input not in all_chioce:
    print("输入错误，请重新输入")
    # 真正处理电脑与用户输入值判断逻辑
        else:
            # 输出用户和电脑输出的选择
            print("电脑出拳为：%s" % computer_choice)
            print("玩家出拳为：%s" % user_input)
            # 双方输入都相同则表示平局
            if user_input == computer_choice:
                print("平局")
                # 下面是玩家能赢的所有条件
                elif (user_input == '石头' and computer_choice == '剪刀') or \
                (user_input == '剪刀' and computer_choice == '布') or \
                (user_input == '布' and computer_choice == '石头'):
                    print("玩家赢了")
                    # 以上条件都不满足则表示电脑赢了
                    else:
                        print("电脑赢了")
```

## while循环

> 1. 一组被重复执行的语句称之为 **循环体**，能否继续重复，**决定循环的终止条件**
> 2. Python 中的循环中 **while** 循环和 **for** 循环
> 3. 循环 **次数未知** 的情况下，建议采用 while 循环
> 4. 循环 **次数可以预知** 的情况下，建议采用 **for** 循环

### while循环语法结构

> - 循环的作用就是让 **指定的代码** 重复的执行
> - `while` 循环最常用的应用场景就是 **让执行的代码** 按照 **指定的次数** **重复** 执行

```python
while 表达式：
    while_suite
```

### 练习1：打印5编hello world

```python
# 定义重复次数的计算器
count = 0
# 使用while判断条件
while count < 5:
    # 需要重复的代码
    print("hello world!")
    # 处理计数器 count
    count += 1
```

### 练习2：完善石头剪刀布游戏

```python
import random
all_chioce = ['石头', '剪刀', '布']
computer_choice = random.choice(all_chioce)
games_count = 0 # 用于记录游戏的次数
user_count = 0 # 记录玩家猜对的次数
computer_count = 0 # 记录电脑猜对的次数
avg_count = 0 # 记录平局的次数
while True:
    user_input = input("请出拳（石头、剪刀、布）按q退出：")
    # 使用一个break语句做一个玩家退出
    if user_input == 'q':
        print("-" * 10)
        print("总共玩了%d局" % games_count)
        print("电脑赢了%d局，玩家赢了%d局，平局%d局" % (computer_count, user_count, avg_count))
        print("-" * 10)
        break # 结束整个循环
    elif user_input not in all_chioce:
        print("输入错误，请重新输入")
        continue # 结束本次循环，直到用户输入对为止
    else:
        print("电脑出拳为：%s" % computer_choice)
        print("玩家出拳为：%s" % user_input)

        if user_input == computer_choice:
            print("平局")
            avg_count += 1
        elif (user_input == '石头' and computer_choice == '剪刀') or \
                (user_input == '剪刀' and computer_choice == '布') or \
                (user_input == '布' and computer_choice == '石头'):
            print("玩家赢了")
            user_count += 1
        else:
            print("电脑赢了")
            computer_count += 1
        games_count += 1
```

### break、continue、else

> `break` 和 `continue` 是专门在循环中使用的关键字
>
> - `break` **某一条件满足时**，退出循环，不再执行后续重复的代码
> - `continue` **某一条件满足时，跳过当前循环**，不执行 continue 后续重复的代码
>
> `break` 和 `continue` 只针对 **当前所在循环** 有效

#### break

> - **在循环过程中**，如果 **某一个条件满足后**，**不** 再希望 **循环继续执行**，可以使用 `break` 退出循环

```python
# 计算1加到100，当和值大于2000时结束循环
count = 0
cale_num = 1
while count < 100:
    cale_num += count
    count += 1
    if cale_num > 2000:
        break
print(count)
```

#### continue

> - 当遇到 **continue** 语句时，程序会 **跳过当前循环**，并忽略剩余的语句，然后 **回到循环的顶端**
> - 如果仍然满足循环条件，循环体内语句继续执行，否则退出循环

```python
# 打印1到100之间不能被3整除的数
num = 0
while num < 100:
    num += 1
    if num % 3 == 0:
        continue
    print(num)
```

#### else

> - python 中的 while 语句也支持 else 子句
> - **else 子句只在循环完成后执行**
> - **break 语句也会跳过 else 块**

### 练习3：猜数字游戏

> 通过一个复合语句来编写本练习
>
> 复合语句：使用多种判断语句

```python
"""
系统随机生成 100 以内的整数
要求用户猜生成的数字是多少
最多猜 5 次，猜对结束程序
如果5次全部猜错，则输出正确结果
"""
import random
# 使用random模块生成随机1-100的整数
coputet_num = random.randint(1, 100)
# 记录猜测的次数
guess_count = 0
# 使用while循环
while guess_count < 5:
    # 获取用户猜测的数字
    guess_num = int(input("请输入你猜的数字："))
    with open("answer.txt", "a", encoding="utf-8") as f:
        # 开外挂；在当前目录下生成一个answer文档，里面存放本次系统给出的数字
        f.write("本论电脑给出的数为：" + str(guess_num))
        # 刷新直接写入到系统磁盘
        f.flush()
    # 猜测次数+1
    guess_count += 1
    # 猜测数与电脑生成数相同则结束循环
    if guess_num == coputet_num:
        print("恭喜你，猜对了！")
        break
    elif guess_num < coputet_num:
        print("你猜的数字太小了！")
        print("你还有", 5 - guess_count, "次机会")
        continue
    else:
        print("你猜的数字太大了！")
        print("你还有", 5 - guess_count, "次机会")
        continue
# 循环结束后，输出正确答案
else:
    print("很遗憾，你没有猜对，正确答案是：", coputet_num)
```

## for循环详解

### for循环语法结构

> 可迭代对象：字符串、列表、元组、字典等等

```python
for 变量 in 可迭代对象:
    for循环逻辑
```

#### 练习1：循环打印内容

```python
# for循环遍历字符串;
for strs in "ABCDEF":
    print(strs)
# 使用for循环遍历列表；
lanage = ["pyhon", "C++", "Java", "JS"]
for item in lanage:
    print(item)

```

> for一般用于指定次数的循环逻辑中，一般情况下，循环次数未知采用while循环，循环次数已知，采用for循环。在for关键字后面跟的这个item变量，变量名是可以根据变量命名规则更改的

#### 练习2：计算1+100

```python
"""
计算1+100的总和
"""
num = 0
for i in range(1, 101):
    num += i
else:
    print("总和为：%d" % num)
```

### 遍历及迭代器

> + 遍历
>
>   由练习1这样将字符串或列表中的值一个一个的取出然后再进行操作，这就是遍历
>
>   遍历不仅限于列表，还适用于元组，字典，和字符串类型
>
> + 可迭代对象
>
>   可迭代对象有字符串、列表、元组、集和、字典、range()，并且能被for循环的都是可迭代对象

### range函数

> range 函数是一个内建函数，它的返回值是一个半闭半开范围内的整数。for 循环常与range函数一起使用，range 函数为循环提供条件。

#### 语法结构

```python
range(start, end, step=1)
```

#### 基本使用

```python
"""
range函数的基本使用
"""
print(list(range(5))) # [0, 1, 2, 3, 4]

for i in range(5):
    print(i)
    
print(list(range(0, 5, 2)))     # [0, 2, 4]
```

### 列表推导式

> 它是一个非常有用、简单、灵活的工具，可以用来动态地创建列表，注意：它只是将for循环简写了，只是针对简单的for循环，复杂的循环不建议使用

```python
# 语法结构
[表达式 for 变量 in 可迭代对象]
```

#### 练习3：生成1-10列表

```python
# 使用普通for循环
nums = []
for i in range(1, 11):
    nums.append(i)
print(nums)

# 使用列表推导式
lists = [i for i in range(1, 11)]
print(lists)
```

### 斐波那契数列

> 斐波那契数列就是某一个数，总是前两个数之和，比如 0，1，1，2，3，5，8

```python
"""
1.使用for循环和range函数编写一个程序，计算有10个数字的斐波那契数列
2.改进程序，要求用户输入一个数字，可以生成用户需要长度的斐波那契数列
"""
```

#### 练习：完成1

```python
# 1.使用for循环和range函数编写一个程序，计算有10个数字的斐波那契数列
start_list = [0, 1] # 指定初始的斐波那契数列的初始两个值
for i in range(8): # for循环执行8次，列表fib中的元素个数变为10【初始2个 + 新增的8个】
    start_list.append(start_list[-1] + start_list[-2]) # 列表追加，每次都是最后一个元素和倒数第二个元素相加，产生新的元素
print(start_list)
```

#### 练习：完成2

```python
# 2.改进程序，要求用户输入一个数字，可以生成用户需要长度的斐波那契数列
start_list = [0, 1]
input_num = int(input("请输入需要生成指定长度的斐波那契数列："))
for i in range(2, input_num):
    start_list.append(start_list[-1] + start_list[-2])
print(start_list)
```



# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习
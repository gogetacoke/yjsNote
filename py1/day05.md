- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [列表](#列表)
    - [概述](#概述)
    - [列表定义](#列表定义)
    - [常用方法](#常用方法)
      - [insert(索引, 数据)](#insert索引-数据)
      - [append(数据)](#append数据)
      - [extend(列表2)](#extend列表2)
      - [修改](#修改)
      - [遍历循环](#遍历循环)
      - [del  列表\[索引\]](#del--列表索引)
      - [remove(数据)](#remove数据)
      - [列表.pop](#列表pop)
      - [列表.pop(索引)](#列表pop索引)
      - [列表.clear](#列表clear)
      - [len(列表)](#len列表)
      - [列表.count(数据)](#列表count数据)
      - [列表.sort()](#列表sort)
      - [列表.reverse()](#列表reverse)
      - [列表.index()](#列表index)
    - [列表综合练习](#列表综合练习)
  - [元组](#元组)
    - [概述](#概述-1)
    - [元组定义](#元组定义)
    - [常用方法](#常用方法-1)
      - [获取值](#获取值)
      - [count()](#count)
      - [index()](#index)
      - [len()](#len)
      - [元组遍历](#元组遍历)
  - [字典](#字典)
    - [概述](#概述-2)
    - [字典定义](#字典定义)
    - [常用方法](#常用方法-2)
      - [访问字典](#访问字典)
      - [更新键值](#更新键值)
      - [删除操作](#删除操作)
      - [补充](#补充)
    - [字典综合练习](#字典综合练习)
  - [集和](#集和)
    - [概述](#概述-3)
    - [集和定义](#集和定义)
    - [常用方法](#常用方法-3)
      - [集和操作符](#集和操作符)
      - [添加元素](#添加元素)
      - [删除元素](#删除元素)
      - [交集](#交集)
      - [并集](#并集)
      - [差集](#差集)
      - [集和转换](#集和转换)
    - [练习](#练习)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充-1)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)


# 学习目标

列表

元组

字典

集和

# 课堂笔记（命令）

# 课堂笔记（文本）

## 列表

### 概述

> - 描述一组数据
> - 列表是 **有序、可变** 的数据类型
> - 列表中可以包含 **不同类型** 的对象
> - 列表可以由 **[]** 创建
> - 支持 **下标** 及 **切片** 操作

### 列表定义

```python
"""
符号：[]
关键字：list()
"""

list1 = ['1', 2, '3']
list2 = list("abc")
print(list1) # ['1', 2, '3']
print(list2) # ['a', 'b', 'c']
```

### 常用方法

| 序号 | 分类 | 关键字 / 函数 / 方法    | 说明                     |
| ---- | ---- | ----------------------- | ------------------------ |
| 1    | 增加 | 列表.insert(索引, 数据) | 在指定位置插入数据       |
|      |      | 列表.append(数据)       | 在末尾追加数据           |
|      |      | 列表.extend(列表2)      | 将列表2 的数据追加到列表 |
| 2    | 修改 | 列表[索引] = 数据       | 修改指定索引的数据       |
| 3    | 删除 | del 列表[索引]          | 删除指定索引的数据       |
|      |      | 列表.remove(数据)       | 删除第一个出现的指定数据 |
|      |      | 列表.pop                | 删除末尾数据             |
|      |      | 列表.pop(索引)          | 删除指定索引数据         |
|      |      | 列表.clear              | 清空列表                 |
| 4    | 统计 | len(列表)               | 列表长度                 |
|      |      | 列表.count(数据)        | 数据在列表中出现的次数   |
| 5    | 排序 | 列表.sort()             | 升序排序                 |
|      |      | 列表.sort(reverse=True) | 降序排序                 |
|      |      | 列表.reverse()          | 逆序、反转               |

#### insert(索引, 数据)

```python
"""
insert(索引,数据)
根据索引在列表指定位置插入数据，且在指定位置前进行数据插入
"""
list3 = [1, 2, 3, 5]

list3.insert(3, "1")
print(list3)  # [1, 2, 3, '1', 5]

list3.insert(4, ["xixi", "哈哈"])
print(list3) # [1, 2, 3, '1', ['xixi', '哈哈'], 5]

# 插入索引超出列表范围，将追加到列表后，但索引已经被插入值
# 当再次插入数据的索引比它小时，将按照索引顺序进行添加
list3.insert(10, "a")
print(list3) # [1, 2, 3, '1', ['xixi', '哈哈'], 5, 'a']

list3.insert(6, "b")
print(list3) # [1, 2, 3, '1', ['xixi', '哈哈'], 5, 'b', 'a']

list3.insert(7, "c")
print(list3) # [1, 2, 3, '1', ['xixi', '哈哈'], 5, 'b', 'c', 'a']
```

#### append(数据)

```python
"""
append(数据)
在列表末尾追加数据
"""
list4 = [1, 2, 3, 4]
list4.append("哈哈")
print(list4) # [1, 2, 3, 4, '哈哈']

list4.append(["细细", "嘟嘟"])
print(list4) # [1, 2, 3, 4, '哈哈', ['细细', '嘟嘟']]
```

#### extend(列表2)

```python
"""
extend(列表)
同append一样，将数据追加到列表末尾，但extend会将数据进行拆分，将拆分后的数据加入到末尾
"""
list5 = [1, 2, 3, 4]
list5.extend("567")
print(list5) # [1, 2, 3, 4, '5', '6', '7']

list5.extend(['a', 'b', 'c'])
print(list5) # [1, 2, 3, 4, '5', '6', '7', 'a', 'b', 'c']
```

#### 修改

```python
"""
列表[索引] = 数据
修改列表中的值，通过索引位置进行修改，只能修改列表中存在的值,索引位置不能超出列表长度
"""
list6 = ["Python", "Java", "Linux", "Go"]
list6[3] = "C++"
print(list6) # ['Python', 'Java', 'Linux', 'C++']
```

#### 遍历循环

```python
"""
查询列表的途径：通过索引进行查询、通过切片进行查询
遍历循环：
遍历就是从头到尾依次从列表 获取数据
在 循环体内部 针对每一个元素，执行相同的操作
在 Python 中为了提高列表的遍历效率，专门提供的迭代 teration 遍历
使用 for 就能够实现迭代遍历
"""          
list12 = ["南京", "北京", "上海", "广州"]    
for item in list
    print(item) # 循环答打印列表中的元素
```

#### del  列表[索引]

```python
"""
del  列表[索引]
del将一个变量从内存中删除
如果使用了del进行删除，后续再调用该变量将不能再使用了
不推荐使用
"""
list7 = ["中国", "成都", "四川", "南充"]
print(list7[3]) # 南充
del list7[3]
print(list7[3])  # list index out of range
```

#### remove(数据)

```python
"""
remove(数据)
根据值进行删除，从左向右进行删除，如果列表中存在多个相同值，则匹配就停止，匹配到第一个就删除
注意：是根据数据值删除，不是索引
没有返回值
"""
list8 = [1, 2, 3, 4, 4, 7, 5, 6]
list8.remove(4)
print(list8) # [1, 2, 3, 4, 7, 5, 6]
```

#### 列表.pop

```python
"""                                    
列表.pop()                               
根据索引进行弹出，默认弹出尾部元素                      
有返回值且为弹出的元素                            
"""                                    
list9 = ["lol", "DNF", "cf", "volt"]   
pop_value1 = list9.pop()               
print(pop_value1)   # volt             
print(list9) # ['lol', 'DNF', 'cf']                                           
```

#### 列表.pop(索引)

```python
"""
根据索引值进行删除，且索引不能超出列表长度
"""
list10 = ["南京", "北京", "上海", "广州"]      
pop_value2 = list10.pop(2)             
print(pop_value2)   #  上海              
print(list10)  # ['南京', '北京', '广州']  
```

#### 列表.clear

```python
"""                      
列表.clear                 
清空列表所有元素                 
"""                      
list11 = ["南京", "北京", "上海
list11.clear()           
print(list11)   # []     
list11.append("中国")      
print(list11)  #  ['中国'] 
```

#### len(列表)

```python
"""                     
len(列表)                 
获取列表有几个元素（获取列表的长度）                
"""                     
list13 = ["南京", "北京", "上
print(len(list13))    # 
```

#### 列表.count(数据)

```python
"""                           
列表.count(数据)                  
统计列表中某个元素出现的次数                
不存在则返回0                       
"""                           
list14 = ["南京", "北京", "上海", "广
print(list14.count("北京"))  # 2
print(list14.count("上海"))  # 1
print(list14.count("西安"))  # 0
```

#### 列表.sort()

```python
"""                                             
列表.sort()                                       
给列表中的元素进行排序，默认从小到大的升序排
设置reverse为True则是降序排
"""                                             
list15 = ['a', 'b', 'd', 'g', 'e', 'f']         
list16 = [1, 3, 2, 5, 4]                        
list15.sort()                                   
list16.sort()                                   
print(list15) # ['a', 'b', 'd', 'e', 'f', 'g']  
print(list16)  # [1, 2, 3, 4, 5] 

list15.sort(reverse=True)                       
print(list15) # ['g', 'f', 'e', 'd', 'b', 'a']  
```

#### 列表.reverse()

```python
"""                                                  
列表.reverse()                                         
列表反转，将列表中的数据反转过来  
可以以根据切片进行反转
[::-1]
"""                                                  
list17 = ["南京", "北京", "上海", "广州", "北京"]              
print(list17[::-1])  # ['北京', '广州', '上海', '北京', '南京']

list17.reverse()                                     
print(list17)    # ['北京', '广州', '上海', '北京', '南京']    
```

#### 列表.index()

```python
"""
列表.index(str)
返回str在列表中的索引位置
存在相同的元素时，则返回匹配到第一个出现的位置索引
"""
list18 = ["南京", "北京", "上海", "广州", "北京"]
print(list18.index("上海")) # 2
print(list18.index("北京")) # 1
```

### 列表综合练习

```python
"""
计算列表的 长度 并输出
请通过步长获取索引为 偶数 的所有值，并打印出获取后的列表
列表中追加元素 mike，并输出添加后的列表
请在列表的第 1个位置插入元素 Tony ，并输出添加后的列表
请修改列表第 2 个位置的元素为 Kelly，并输出修改后的列表
请将列表 l2 = [1,”a”,3,4,”heart”] 的每一个元素追加到列表 li 中，并输出添加后的列表
请删除列表中的元素 ”barry”，并输出删除后的列表
请删除列表中的第 2 个元素，并 输出 删除元素后的列表
"""
li = ["alex", "jerry", "tom", "barry", "wall"]
# 计算列表的 长度 并输出
print(len(li))

# 请通过步长获取索引为 偶数 的所有值，并打印出获取后的列表
print(li[::2])

# 列表中追加元素 mike，并输出添加后的列表
li.append('mike')
print(li)

# 请在列表的第 1个位置插入元素 Tony ，并输出添加后的列表
li.insert(0, "Tony")
print(li)

# 请修改列表第 2 个位置的元素为 Kelly，并输出修改后的列表
li[1]="Kelly"
print(li)

# 请将列表 l2 = [1,”a”,3,4,”heart”] 的每一个元素追加到列表 li 中，并输出添加后的列表
l2 = [1, "a", 3, 4, "heart"]
li.extend(l2)
print(li)

# 请删除列表中的元素 ”barry”，并输出删除后的列表
li.remove("barry")
print(li)

# 请删除列表中的第 2 个元素，并 输出 删除元素后的列表
del_value = li.pop(2)
print(del_value)
print(li)
```

## 元组

### 概述

> - 通过 `()`  创建元组
> - 元组是**有序的、不可变类型**
> - 与列表类似，作用于列表的操作，绝大数也可以作用于元组
> - 一般用于存储一些在程序中不应该被修改的一系列值

### 元组定义

```python
"""
符号：()
关键字：tuple
"""
str1 = "1china中国"
t1 = tuple(str1)
print(t1) # ('1', 'c', 'h', 'i', 'n', 'a', '中', '国')

# 如果一个元组中只有一个元素，那么创建该元组时，需要加上一个 逗号，否则创建失败
a = ("a")
b = ("a", )
print(type(a)) # <class 'str'>
print(type(b)) # <class 'tuple'>
```

### 常用方法

#### 获取值

```python
"""
通过索引或切片获取元组中的值
"""
tuple1 = (1, 2, 3, 4, 3, 3)
print(tuple1[0]) # 1
print(tuple1[:2]) # (1, 2)
```

#### count()

```python
"""
count(str)
统计str在元组中出现的次数
"""
tuple2 = (1, 2, 3, 4, 3, 3)
print(tuple2.count(3)) # 3
```

#### index()

```python
"""
index(str)
返回str在元组中的索引位置，如果存在多个相同的值，则返回第一个值的索引
"""
tuple3 = (1, 2, 3, 4, 3, 3)
print(tuple3.index(2)) # 1
print(tuple3.index(3)) # 2
```

#### len()

```python
"""
len()
获取元组的长度,返回元组元素的个数
"""
tuple4 = (1, 2, 3, 4, 3, 3)
print(len(tuple4)) # 6
```

#### 元组遍历

```python
"""
元组遍历
循环打印元组的元素
"""
tuple5 = (1, 2, 3, 4, 3, 3)
for item in tuple5:
    print(item)
```

## 字典

### 概述

> - `dictionary`（字典） 是 **除列表以外** `Python` 之中 **最灵活** 的数据类型
> - 字典同样可以用来 **存储多个数据**
>   - 通常用于存储 **描述一个 `物体` 的相关信息** 
> - 和列表的区别
>   - **列表** 是 **有序** 的对象集合
>   - **字典** 是 **无序** 的对象集合
> - 字典用 `{}` 定义
> - 字典使用 **键值对** 存储数据，键值对之间使用 `,` 分隔
>   - **键** `key` 是索引
>   - **值** `value` 是数据
>   - **键** 和 **值** 之间使用 `:` 分隔
>   - **键必须是唯一的**
>   - **值** 可以取任何数据类型，但 **键** 只能使用 **字符串**、**数字**或 **元组**
>   - 可变对象不能作为字典的key，例如：列表和字典；
>   - 不可变对象可以作为字典的键，例如：数值，字符串，和元组；

### 字典定义

```python
"""
符号：{}
关键字：dict
"""
dict1 = {"name": "可乐", "age": 18}
print(type(dict1))
print(dict1)
```

### 常用方法

#### 访问字典

> 字典是映射类型，意味着它没有下标，访问字典中的值需要使用相应的键，通过键 `(key)`，访问字典

```python
"""
字典的访问方法
通过键获取对应的值
"""

dict2 = {"name":"小航", "age":18, "hobby":"唱跳RAP"}
# 直接获取
print(dict2["age"]) # 18
# print(dict2["2"]) # KeyError: '2'

# 使用get方法进行获取；获取键的值存在就返回，不存在就返回None；
print(dict2.get("hobby")) # 唱跳RAP
print(dict2.get("sex")) # None
# 也可以对不存在的键设置默认值
print(dict2.get("sex", "库里南")) # 库里南

# 获取所有键和值
print(dict2.keys()) # dict_keys(['name', 'age', 'hobby'])
print(dict2.values()) # dict_values(['小航', 18, '唱跳RAP'])

# 通过遍历打印键所对应的值
for i in dict2.keys():
    # 打印所有键
    print(i)
    # 打印所有键所对应的值
    print(dict2[i])

# 直接循环遍历字典输出的是字典的键
for key in dict2:
    print(key)
    # 也可以根据键获取对应的值
    print(key, dict2[key])    
    
# 获取所有的键值对;返回的是一个列表，列表中每个键值对用元组进行包裹
print(dict2.items()) #dict_items([('name', '小航'), ('age', 18), ('hobby', '唱跳RAP')])
# 循环遍历键值对
for i in dict2.items():
    print(i) # ('name', '小航')  
    print(i[0], i[1]) # 通过索引在元组中取值
# 通过两个变量遍历获取键值对
for k,v in dict2.items():
    print(k,v)
    
# 通过 in 或 not in 判断键是否存在字典中
print("name" in dict2) # True
print("color" in dict2) # False    
```

#### 更新键值

```python
"""
更新键值对
1.dict.update(k:v)
    字典中不存在的键值，则追加到后面，存在就西修改
2.dict[k]=v
    存在就更新，不存在则添加
"""
dict3 = {"name": "iphone 14", "price": 41999, "colr": "white"}
# 通过update方法进行修改
dict3.update({"name": "iphone 18", "mode": "512GB"})
print(dict3) # {'name': 'iphone 18', 'price': 41999, 'colr': 'white', 'mode': '512GB'}

# 通过字典对象的方式进行更新
dict3["price"] = 5399 # 更新存在的键值
dict3["title"] = "买不起" # 更新不存在的键值
print(dict3) # {'name': 'iphone 18', 'price': 5399, 'colr': 'white', 'mode': '512GB', 'title': '买不起'}
```

#### 删除操作

```python
"""
字典的删除操作
pop(k) 弹出指定键对应的元素并将弹出的元素返回
del dict[k]：删除指定的键值对，不返回数据
清空字典：dict.clear()
"""
dict4 = {"name": "iphone 14", "price": 41999, "colr": "white"}
# 使用pop进行删除键；返回值为键对应的值
remove_value = dict4.pop("name")
print(dict4) # {'price': 41999, 'colr': 'white'}
print(remove_value) # iphone 14

# del：删除指定的键值对，不返回数据
del dict4["price"]
print(dict4) # {'colr': 'white'}

# clear()清空字典
dict4.clear()
print(dict4) # {}
```

#### 补充

```python
"""
len()获取字典中的元数个数
"""
dict5 = {"name": "iphone 14", "price": 41999, "colr": "white"}
# len获取字典中键值对的个数
print(len(dict5))
```

### 字典综合练习

```python
"""
支持新用户注册(添加)，新用户名和密码注册到字典中
支持老用户登陆(查询)，用户名和密码正确提示登陆成功
主程序通过循环询问，进行何种操作，根据用户的选择，执行注册或是登陆操作
"""
data = {"yyh":"123"}
def register():
    while True:
        username = input("请输入用户名：")
        if username in data:
            print("用户名已经存在，请重新输入！")
            continue
        else:
            passwd = input("请输入密码：")
            data[username] = passwd
            print("用户创建成功")
            break
def login():
    while True:
        username = input("请输入登陆用户名：")
        passwd = input("请输入密码：")
        if username not in data:
            print("用户名不存在！请重新输入！")
            continue
        elif data.get(username) != passwd:
            print("密码输入错误，请重新输入！")
            continue
        else:
            print("登陆成功！")
            break
def quey_user_pwd():
    while True:
        username = input("请输入需要查询用户的用户名：")
        if username not in data:
            print("用户名不存在，请重新输入！")
            continue
        else:
            print("用户名：" + username)
            print("密码：" + data[username])
            break
def quey_user_info():
    for k,v in data.items():
        print(f"用户名：{k},密码：{v}")
while True:
    menu = print("""
        欢迎登陆xx后台系统
         1.注册
         2.登陆
         3.查询用户密码
         4.查询系统用户
         5.退出   
    """)
    num = int(input("请选择功能点："))
    if num == 1:
        register()
    elif num == 2:
        login()
    elif num == 3:
        quey_user_pwd()
    elif num == 4:
        quey_user_info()
    elif num == 5:
        print("Bye!!")
        break
    else:
        print("输入有误，请重新选择！")
        continue
```

## 集和

### 概述

> - 用于描述一组信息
> - 集合与元组和列表相似都用于做容器，在内部可以放一些子元素
> - 集合有三特殊特点： `子元素不重复` 、 `子元素必须可哈希`、`无序`。
>
> 可哈希的数据类型：数字、字符串、元组、布尔
>
> 不可哈希的数据类型：字典、列表、集和
>
> [注]：集和与字典的符号相同，但存储的值不同，一个存储键值对，一个存储值

### 集和定义

```python
"""
符号：{}
关键字：set
"""
set1 = {1, 2, 3, '12'}
set2 = {1, (2, 3), 3}
set3 = {1, 3, 2, 9, 8}
print(set3) # {1, 9, 3, 8, 2} 
```

### 常用方法

#### 集和操作符

```python
"""
集和的操作符号
集合支持用 in 和 not in 操作符检查成员
能够通过 len() 检查集合大小
能够使用 for 迭代集合成员
不能取切片，没有键
"""
# in 和 not in 操作
set4 = {1, 2, 3, "可乐", "abc", (1, 2, 'a')}
print("可乐" in set4) # True
print('a' not in set4) # True(a在集和的元组中，不属于集和)
print(4 in set4) # False

# len()操作,返回集和中元素的个数
print(len(set4)) # 6

# 通过遍历打印所有元素
for i in set4:
    print(i)
```

#### 添加元素

```python
"""
添加元素 add()
如果添加集和中存在的元素，则添加无效
"""
set5 = {1, 2, 3, "可乐", "abc", (1, 2, 'a')}
set5.add("娃娃")
set5.add("可乐")
print(set5) # {1, 2, 3, '娃娃', '可乐', (1, 2, 'a'), 'abc'}
```

#### 删除元素

```python
"""
删除元素
remove()
删除的元素不存在则报错：KeyError
"""
set6 = {1, 2, 3, "可乐", "abc", (1, 2, 'a'), True}
set6.remove("可乐")
print(set6) # {1, 2, 3, (1, 2, 'a'), 'abc', True}
```

#### 交集

```python
"""
集和的交集
& 或 intersection()
输出两个集和中相同的数据
"""
set7 = {'a', 'b', 'c', 'd'}
set8 = {'e', 'f', 'a', 'b'}
# 方法一
print(set7 & set8) # {'a', 'b'}
# 方法二
print(set7.intersection(set8)) # {'b', 'a'}
```

#### 并集

```python
"""
集和的并集
| 或 union()
返回两个集和中不重复的数据
"""
set9 = {'a', 'b', 'c', 'd'}
set10 = {'e', 'f', 'a', 'b'}
# 方法一
print(set9 | set10) # {'e', 'a', 'd', 'f', 'c', 'b'}
# 方法二
print(set9.union(set10)) # {'e', 'a', 'd', 'f', 'c', 'b'}
```

#### 差集

```python
"""
集和的差集
- 或 difference()
返回前集和中与后集和中不同的元素
"""
set11 = {'a', 'b', 'c', 'd'}
set12 = {'e', 'f', 'a', 'b'}
# 方法一
print(set11 - set12) # {'d', 'c'}
# 方法二
print(set12.difference(set11)) # {'e', 'f'}
```

#### 集和转换

```python
"""
集和转换
其他任意类型的数据转换集和可以使用set()进行转换，如果有重复数据则自动去重
数字不能进行转换
"""
strs = "qwerwer"
lists = ['a', 'b', 'd', 1, 'a', 1]
tuples = (1, 5, 1, 5)
dicts = {"name": "coke", "age": "21", "title": "han", "name": "coke"}

print(set(strs)) # {'w', 'q', 'r', 'e'}
print(set(lists)) # {'d', 1, 'b', 'a'}
print(set(tuples)) # {1, 5}
print(set(dicts)) # {'age', 'name', 'title'}
print(set(dicts.values())) # {'coke', 'han', '21'}
```

### 练习

```python
"""
取出只有在b.log在独有的记录(差集)
"""
file1 = "a.log"
file2 = "b.log"
with open(file1, mode='r') as f:
    end_file1 = set(f.readlines())
with open(file2, mode='r') as f:
    end_file2 = set(f.readlines())
result = end_file2 - end_file1
for i in result:
    print(i, end='')
```





# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习
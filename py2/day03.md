- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [面向过程](#面向过程)
    - [基本概念](#基本概念)
    - [重心](#重心)
  - [面向对象](#面向对象)
    - [基本概念](#基本概念-1)
    - [重心](#重心-1)
  - [类和对象](#类和对象)
    - [概念](#概念)
    - [类的设计](#类的设计)
      - [类设计1](#类设计1)
  - [面向对象实践](#面向对象实践)
    - [三大特征](#三大特征)
    - [语法结构](#语法结构)
    - [self参数](#self参数)
    - [init初始化方法](#init初始化方法)
    - [练习1：编写游戏人物](#练习1编写游戏人物)
  - [继承](#继承)
    - [概念](#概念-1)
    - [继承的语法](#继承的语法)
    - [练习2：Animal](#练习2animal)
    - [练习3：游戏人物](#练习3游戏人物)
  - [多重继承](#多重继承)
  - [类特殊方法](#类特殊方法)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)


# 学习目标

OOP基础

OOP进阶

# 课堂笔记（命令）

# 课堂笔记（文本）

## 面向过程

### 基本概念

> 面向过程是一种以事件为中心的编程思想，编程的时候把解决问题的步骤分析出来，然后用函数把这些步骤实现，在一步一步的具体步骤中再按顺序调用函数。

### 重心

> 怎么去做？
>
> 1. 把完成某一个需求的 `所有步骤` `从头到尾` 逐步实现
> 2. 根据开发需求，将某些 **功能独立** 的代码 **封装** 成一个又一个 **函数**
> 3. 最后完成的代码，就是顺序地调用 **不同的函数**
>
> 特点：
>
> 1. 注重 **步骤与过程**，不注重职责分工
> 2. 如果需求复杂，代码会变得很复杂
> 3. **开发复杂项目，没有固定的套路，开发难度很大！**
>
> 注重过程：怎么去做，所有事情都自己去做

## 面向对象

### 基本概念

> 面向对象是一种编程范式，它将程序中的数据和操作封装在一个对象中，从而使得程序更加模块化、可重用和易于维护。在面向对象编程中，对象是类的实例化，类是一种抽象的数据类型，它定义了一组属性和方法。
>
> 面向对象编程的核心思想是封装、继承和多态。封装是将数据和操作封装在一个对象中，隐藏对象的内部实现细节，只暴露必要的接口给外部访问。继承是通过继承父类的属性和方法，来创建一个新的子类，子类可以重写父类的属性和方法，从而实现更加灵活的功能。多态是指同一个方法可以根据不同对象的实现方式，实现不同的行为。

### 重心

> 1. 在完成某一个需求前，首先确定 **职责** —— **要做的事情（方法）**
> 2. 根据 **职责** 确定不同的 **对象**，在 **对象** 内部封装不同的 **方法**（多个）**和属性**
> 3. 最后完成的代码，就是顺序地让 **不同的对象** 调用 **不同的方法**
>
> 特点：
>
> 1. 注重 **对象和职责**，不同的对象承担不同的职责
> 2. 更加适合应对复杂的需求变化，**是专门应对复杂项目开发，提供的固定套路**
> 3. **需要在面向过程基础上，再学习一些面向对象的语法**

## 类和对象

### 概念

> - **类** 是对一群具有 **相同 特征** 或者 **行为** 的事物的一个统称，是抽象的，**不能直接使用**
>   - **特征** 被称为 **属性**
>   - **行为** 被称为 **方法**。
> - **对象** 是 **由类创建出来的一个具体存在**，可以直接使用
> - 由 **哪一个类** 创建出来的 **对象**，就拥有在 **哪一个类** 中定义的：
>   - 属性
>   - 方法
>
> - **类是模板**，**对象** 是根据 **类** 这个模板创建出来的，应该 **先有类，再有对象**
> - **类** 只有一个，而 **对象** 可以有很多个
>   - **不同的对象** 之间 **属性** 可能会各不相同
> - **类** 中定义了什么 **属性和方法**，**对象** 中就有什么属性和方法，**不可能多，也不可能少**

### 类的设计

> 1. **类名** 这类事物的名字，**满足大驼峰命名法**
> 2. **属性** 这类事物具有什么样的特征
> 3. **方法** 这类事物具有什么样的行为
>
> 大驼峰命名法：
>
> 1. 每一个单词的首字母大写
> 2. 单词与单词之间没有下划线
>
> 属性和方法的确定：
>
> - 对 **对象的特征描述**，通常可以定义成 **属性**
> - **对象具有的行为**（动词），通常可以定义成 **方法**

#### 类设计1

```python
"""
小明 今年 18 岁，身高 1.75，每天早上 跑 完步，会去 吃 东西
小美 今年 17 岁，身高 1.65，小美不跑步，小美喜欢 吃 东西
"""
# 创建人这个类
class Person:
    def __init__(self, name, age, height): # 设置他的属性
        self.name = name
        self.age = age
        self.height = height
# 设置他的方法
    def info(self):
        print(f"{self.name}今年{self.age}岁，身高{self.height}")

    def run(self):
        print(f"{self.name} 正在跑步")

    def eat(self):
        print(f"{self.name} 正在吃东西")


xm = Person("小明", 18, 1.75) # 创建出xm对象，并设置初始属性
xm.info() # 调用方法
xm.run()
xm.eat()
xiaomei = Person("小美", 17, 1.65)
```

## 面向对象实践

### 三大特征

> 1. **封装** 根据 **职责** 将 **属性** 和 **方法** **封装** 到一个抽象的 **类** 中
> 2. **继承** **实现代码的重用**，相同的代码不需要重复的编写
> 3. **多态** 不同的对象调用相同的方法，产生不同的执行结果，**增加代码的灵活度**

### 语法结构

> - **方法** 的定义格式和之前学习过的 **函数** 的定义几乎一样

```python
class 类名:

    def 方法1(self, 参数列表):
        pass
  
    def 方法2(self, 参数列表):
        pass
```

### self参数

> - 在类封装的方法内部，`self` 就表示 **当前调用方法的对象自己**
> - **调用方法时**，不需要传递 `self` 参数
> - **在方法内部**
>   - 可以通过 `self.` **访问对象的属性**
>   - 也可以通过 `self.` **调用其他的对象方法**
> - 在 **类的外部**，通过 `变量名.` 访问对象的 **属性和方法**
> - 在 **类封装的方法中**，通过 `self.` 访问对象的 **属性和方法**

### init初始化方法

> - 在日常开发中，不推荐在 **类的外部** 给对象增加属性
>   - 如果**在运行时，没有找到属性，程序会报错**
> - 对象应该包含有哪些属性，应该 **封装在类的内部**
> - 当使用 `类名()` 创建对象时，会 **自动** 执行以下操作：
>   1. 为对象在内存中 **分配空间** —— 创建对象
>   2. 为对象的属性 **设置初始值** —— 初始化方法(`init`)
> - 这个 **初始化方法** 就是 `__init__` 方法，`__init__` 是对象的**内置方法**

```python
class Animal:
    def __init__(self,color,size):
        print("对象初始化")
        self.color = color
        self.size = size
     def info(self):
        pring(f"颜色{self.color}，尺寸{self.size}")
dog = Animal("red","大")
dog.info()
对象初始化
颜色red，尺寸大
```

### 练习1：编写游戏人物

```python
"""
需求：
创建游戏角色类
游戏人物角色拥有名字、武器等属性
游戏人物具有攻击的方法
武器通过武器类实现
"""
class Role:
    def __init__(self, name, weapon): # __init__指定每一个对象独有的属性
        self.name = name #  self 为实例本身的名称
        self.weapon = weapon

    def attack(self, target):
        print(f"我是{self.name}，我使用{self.weapon.name},攻击{target}，掉了{self.weapon.power}滴血")


class Weapon:
    def __init__(self, name, power):
        self.name = name
        self.power = power

wq = Weapon("电机拳套", 500) # 根据武器类实例一个对象wq
lb = Role("电机小子", wq)# 将武器对象wq，作为角色对象lb的武器属性
print(lb.weapon.name, lb.weapon.power) # 电机拳套 500
lb.attack("大大怪") # 我是电机小子，我使用电机拳套,攻击大大怪，掉了500滴血
```

> - 类被定义后，目标就是要把它当成一个模块来使用，并把这些对象嵌入到你的代码中去
> - 组合就是让不同的类混合并加入到其它类中来增加功能和代码重用性
> - 可以在一个大点的类中创建其它类的实例，实现一些其它属性和方法来增强原来的类对象

## 继承

### 概念

> - **继承的概念**：**子类** 拥有 **父类** 的所有 **方法** 和 **属性**

### 继承的语法

```python
class 类名(父类名)
	pass
```

>- **子类** 继承自 **父类**，可以直接 **享受** 父类中已经封装好的方法，不需要再次开发
>- **子类** 中应该根据 **职责**，封装 **子类特有的** **属性和方法**

### 练习2：Animal

```python
"""
- 创建一个动物类：animal
    动物共有的方法：sleep、run、eat、bark
- 创建两个动物类：Dog、Cat
	具体动物有自己的属性：name，color
    动物的共有方法：bark 叫
    重新父类的bark方法
    Dog：汪汪叫
    Cat：喵喵叫
    重新父类eat方法
- 创建一个神犬
    特殊属性：会飞
"""


class Animal:
    def __init__(self, name):
        self.name = name

    def sleep(self):
        print(f"{self.name}正在睡觉")

    def run(self):
        print(f"{self.run}正在跑")

    def eat(self):
        print(f"{self.name}正在吃东西")

    def bark(self):
        print("正在叫")


class Dog(Animal):
    def __init__(self, color, name):
        # 子类中编写特有的属性时，需要先调用父类的构造方法
        super().__init__(name)
        self.color = color

    # 重新父类的bark方法
    def bark(self):
        print(f"{self.color}的{self.name}正在汪汪叫")

    # 重新父类的eat方法
    def eat(self):
        print("正在啃大棒骨头")

class Cat(Animal):
    def __init__(self, color, name):
        # 子类中编写特有的属性时，需要先调用父类的构造方法
        super().__init__(name)
        self.color = color

    def bark(self):
        print(f"{self.name}正在喵喵叫")


class GodDog(Dog):

    def fly(self):
        print("神犬会飞！！")


jinmao = Dog("黄色", "金毛犬")
jinmao.bark() # 黄色的金毛犬正在汪汪叫
jinmao.eat() # 正在啃大棒骨头

jumao = Cat("橘黄色", "狸猫")
jumao.sleep() # 狸猫正在睡觉
jumao.bark() # 狸猫正在喵喵叫

```

### 练习3：游戏人物

```python
class Role:
    def __init__(self, name, weapon):
        self.name = name
        self.weapon = weapon

    def say(self):
        print(f"我是{self.name}，我的武器是{self.weapon}")

# 继承父类Role的所有属性和方法
class Warrior(Role):
    # 调用父类的属性，并编写特有的属性
    def __init__(self, name, weapon, ride):
        super().__init__(name, weapon)
        self.ride = ride
 	# 添加特有的方法
    def attack(self, target):
        print(f"{self.name}驾驭{self.ride}赶往战场中！")
        print(f"{self.name}正在使用{self.weapon}攻击{target}")


class Mege(Role):
    def attack(self, target):
        print(f"{self.name}正在使用{self.weapon}攻击{target}")


dema = Warrior("德玛", "大宝剑", "御剑")
abm = Mege("奥巴马", "双枪")
dema.say()
dema.attack("酒桶")

abm.attack("EZ")


我是德玛，我的武器是大宝剑
德玛驾驭御剑赶往战场中！
德玛正在使用大宝剑攻击酒桶
奥巴马正在使用双枪攻击EZ
```

## 多重继承

> - Python 允许多重继承，即：一个类可以是多个父类的子类，子类可以拥有所有父类的属性
> - 在使用方法时，python有自己的查找顺序：**自下向上**，**自左向右**

```python
class A:
    def fun1(self):
        print("A func")

class B:
    def fun2(self):
        print("B func")

class C(A, B):
    def fun3(self):
        print("C func")
c = C()
c.fun1() # A func
c.fun2() # B func
c.fun3() # C func
```

```python
# 调用父类中相同方法
class A:
    def fun1(self):
        print("A func")
        
    def fun4(self):
        print("A func4")
class B:
    def fun2(self):
        print("B func")

    def fun4(self):
        print("B func4")

class C(A, B): # 继承父类谁在前，就首先调用谁的相同方法 fun4()方法
    def fun3(self):
        print("C func")
        
#class C(B, A): # 当B类写在前时，c.fun4()则打印   B func4
 #  def fun3(self):
  #      print("C func")
        
c = C()
c.fun1()
c.fun2()
c.fun3()
c.fun4() # A func4;根据多层继承的查找顺序来：自左向右进行查询
```

## 类特殊方法

> - 在 Python 中，所有以 “\_\_” 双下划线包起来的方法，都统称为 “Magic Method”
> - 如果对象实现了这些魔法方法中的某一个，那么这个方法就会在特殊的情况下被 Python 所调用
> - \_\_init \_\_方法：实例化类实例时默认会调用的方法
> - \_\_ str \_\_ 方法：打印/显示实例时调用方法，返回字符串
> - \_\_call \_\_方法：用于创建可调用的实例

```python
# 创建新的python文件books.py，魔法方法 __str__，__call__方法的使用
class Book:  # 创建类Book, 定义魔法方法，实现对书籍信息的打印
    def __init__(self, title, author):  # 定义__init__方法，获取书籍的信息【默认自动调用】
        self.title = title
        self.author = author
    #
    def __str__(self):  # 定义__str__方法, 必须返回一个字符串
        return f"书名：{self.title},作者是{self.author}"

    def __call__(self):  # 用于创建可调用的实例，直接作为方法调用
        print("__call__方法被调用了")


if __name__ == '__main__':
    pybook = Book('三只小猪', 'Beryl')  # 抽象出对象pybook
    print(pybook)  # 打印对象，不写“__str__  <__main__.Book object at 0x7f894254bf70>”;写后： 书名：三只小猪,作者是Beryl
    pybook()  # 把对象当作函数调用的时候，自动调用

```



# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习
- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
  - [正则表达式](#正则表达式)
    - [`^`](#)
    - [`$`](#-1)
    - [`[]`](#-2)
    - [`[^]`](#-3)
    - [`.*`](#-4)
    - [`\{n,m}\`](#nm)
    - [`\(\)`](#-5)
    - [`\w`](#w)
    - [`\s`](#s)
    - [`\d`](#d)
  - [扩展正则表达式](#扩展正则表达式)
    - [`+`](#-6)
    - [`？`](#-7)
    - [`{n，m}`](#nm-1)
    - [`()`](#-8)
    - [`|`](#-9)
    - [`\b`](#b)
  - [案例](#案例)
  - [grep补充](#grep补充)
    - [参数](#参数)
  - [sed流式编辑器](#sed流式编辑器)
    - [使用方式](#使用方式)
    - [条件](#条件)
    - [选项](#选项)
      - [`-n`](#-n)
      - [`-i`](#-i)
      - [`-r`](#-r)
    - [指令](#指令)
      - [`p`](#p)
      - [`d`](#d-1)
      - [`s`](#s-1)
      - [`i`](#i)
      - [`a`](#a)
      - [`c`](#c)
- [快捷键](#快捷键)
- [问题](#问题)
  - [简述egrep -q](#简述egrep--q)
  - [编写正则匹配11位手机号](#编写正则匹配11位手机号)
  - [如何使用sed提取文本中的偶数行](#如何使用sed提取文本中的偶数行)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)





# 学习目标

正则表达式

sed基本用法

sed文本块处理

# 课堂笔记（命令）

# 课堂笔记（文本）

## 正则表达式

[注]：

使用正则时，尽量使用引号将表达式引起来

grep查找的必须是文本文件

```sh
]#head -5 /etc/passwd > user # 准备素材
```

### `^` 

匹配前字符开头

```sh
]#grep '^root' user # 匹配user中以root开头的行
```

### `$`

匹配前字符结尾

```sh
]#grep 't$' user #匹配以$结尾的行
```

### `[]`

**集合**

匹配集合中任意单个字符，包含就算

```sh
]#grep "[rot]" user # 匹配单个字符
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin

]#grep "[a-z]" user # 匹配啊包含a-z的所有小写字母
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin

]#grep "[0-9a-Z]" # 匹配数字0-9和所有大小写字母a-z
```

### `[^]`

**集合去反**

```sh
]#grep "[^0-9]" user # 匹配所有数字以外的内容
```

**案例：匹配用户输入的密码是否是数字**

```sh
#!/bin/bash
read -p "please enter pwd: " p
echo $p | greo -q "[^0-9]" 
if [ $? -eq 0 ];then
	echo "input error"
else
	echo ok
fi
```

>  代码解释：由于grep查找的只能是文本文件，所以不能直接写grep "`[`^0-9`]`",所以使用echo喊出来进行查询即可，若不用echo，那就将内容写入文件里面，再查询;
>
> grep -q  静默查询，不输出
>

### `.*`

```sh
# . 匹配任意单个字符
]#grep "." user # 不写匹配字符，则输出所有内容，因为字符都是有单个字符组成
]#greo "r..t" user # 找rt之间有2个字符的行
]#greo "r.t" user # 找rt之间有1个字符的行，没有匹配内，就无输出
# * 匹配前一个字符任意次数，不允许单独使用

#.* 匹配任意字符的任意次数
```

### `\{n,m}\`

匹配前一个字符n到m次

```sh
]# grep 'ro\{1,\}' user  # 匹配o字符1到n次
root:x:0:0:root:/root:/bin/bash 

]# grep 'ro\{1\}' user  # 匹配0字符1次
root:x:0:0:root:/root:/bin/bash
```

### `\(\)`

组合为整体，保留

```sh
]# grep '\(root\)' user 
root:x:0:0:root:/root:/bin/bash

# 组合使用，匹配前一个整体出现2次
]# grep '\(2:\)\{2\}' test.txt 
daemon:x:2:2:daemon:/sbin:/sbin/nologin
```

### `\w`

匹配字母、数字、下划线，\W 非字母、数字、下划线

```sh
]#grep 'ro\w' user # 匹配ro后为字母数字下划线的行
root:x:0:0:root:/root:/bin/bash
```

### `\s`

匹配空格，tab键;\S，非空格，tab

```sh
]#grep '\s' user | wc -l# 找出所有包含空格的行
2
```

### `\d`

匹配数字，[0-9];但兼容性较差;\D非数字行

```sh
]#grep -P '\d' user  # 找出所有包含数字的行
```

## 扩展正则表达式

> 可以加上 -E选项使用
>
> 或使用egrep

### `+`

**最少匹配一次**  匹配前一个字符次数(1次或n次)

```sh
]# egrep 'ro+' test.txt 
root:x:0:0:root:/root:/bin/bash

]# egrep 'r+' test.txt 
root:x:0:0:root:/root:/bin/bash
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

### `？`

**最多匹配一次** 匹配前一个字符次数（1次或0次）

```sh
]# egrep 'r?' test.txt 
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

### `{n，m}`

**匹配前一个字符n到m次**

```sh
]# egrep 'r{1,2}' test.txt # 匹配r出现1次或2次的行
root:x:0:0:root:/root:/bin/bash
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin

]# egrep 'ro{1,2}' test.txt 
root:x:0:0:root:/root:/bin/bas
```

### `()` 

**组合为整体-保留**

```sh
]# egrep '(ro){1,2}' test.txt # 匹配ro出现1,2次的行
root:x:0:0:root:/root:/bin/bash
```

### `|`

**或者**

```sh
]# egrep 'ro|sb' test.txt  #匹配包含ro或sb的行
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

### `\b`

**单词边界** 不允许出现字母数字下划线

```sh
]# egrep '\broot\b' test.txt  # 查找root两边不允许出现字没有数字下划线
root:x:0:0:root:/root:/bin/bash
]#egrep '\broot' # 查找root左边不允许出现字母数字下划线
root:x:0:0:root:/root:/bin/bash
```

## 案例

1. 匹配IP地址

   ```sh
   第一种方式：^(25[0-5]\.|2[0-4][0-9]\.|1?[0-9]?[0-9]\.){3}(25[0-5]|2[0-4][0-9]|1?[0-9]?[0*9])$
   
   第二种方式：^(25[0-5]\.|2[0-4]\d\.|1?\d?\d\.){3}(25[0-5]|2[0-4]\d|1?\d)$
   ```

   代码解释

   > ^` 表示匹配字符串的开始。
   >
   > (25[0-5]\.|2[0-4][0-9]\.|1?[0-9]?[0-9]\.) 这一部分表示匹配IP地址的前三个字段，每个字段的取值范围是0到255。其中：
   >
   > `25[0-5]\.` 匹配250到255之间的数字加上一个点。
   >
   > `2[0-4][0-9]\.` 匹配200到249之间的数字加上一个点。
   >
   > `1?[0-9]?[0-9]\.` 匹配0到199之间的数字加上一个点。
   >
   > `{3}` 表示前面的模式需要出现3次，即匹配三个字段。
   >
   > `(25[0-5]|2[0-4][0-9]|1?[0-9]?[0*9])` 这一部分表示匹配IP地址的最后一个字段，取值范围同样是0到255。
   >
   > `$` 表示匹配字符串的结尾。

## grep补充

### 参数

| 选项 | 含义                                          |
| ---- | --------------------------------------------- |
| -E   | == egrep 支持扩展正则匹配                     |
| -A   | after -A5 匹配你要的内容并且显示接下来的五行  |
| -B   | before -B5 匹配你要的内容并且显示接上面的五行 |
| -C   | context -C5 匹配你要内容的上下五行            |
| -c   | 统计出现了多少行 类似 wc -l                   |
| -v   | 取反，排除(行)                                |
| -n   | 显示行号                                      |
| -i   | 忽略大小写                                    |
| -w   | 精确匹配                                      |
| -q   | 静默输出，屏蔽不显示内容                      |

```sh
# -E # 直接写egrep 方便
[root@web3 ~]# grep 'ro+' /etc/passwd
[root@web3 ~]# grep  -E 'ro+' /etc/passwd
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
# -A
[root@web3 ~]# seq 10 | grep -A2 3 
3
4
5
# -B
[root@web3 ~]# seq 10 | grep -B2 3
1
2
3
# -C
[root@web3 ~]# seq 10 | grep -C2 3
1
2
3
4
5
# -c
[root@web3 ~]# grep 'root' /etc/passwd
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
[root@web3 ~]# grep -c 'root' /etc/passwd
2
# -v
[root@web3 ~]# grep '^root' /etc/passwd | wc -l
1
[root@web3 ~]# grep -v '^root' /etc/passwd | wc -l
22
# -n
[root@web3 ~]# grep -n 'root' /etc/passwd
1:root:x:0:0:root:/root:/bin/bash
10:operator:x:11:0:operator:/root:/sbin/nologin
# -i
[root@web3 ~]# grep -i 'ROOT' /etc/passwd
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
# -w
[root@web3 ~]# grep '22' /etc/services | wc -l
443
[root@web3 ~]# grep -w '22' /etc/services | wc -l
5
```

## sed流式编辑器

**逐行处理**

### 使用方式

```
# 两种使用方式，指令必须有
1、 前置指令 | sed 选项 条件 指令
2、 sed 选项 '条件 指令' 文档
```

### 条件

> 行号：需要处理那一行
>
> /字符串/：需要处理字符串那一行
>
> 数字：指定特定行，如 `1` 表示第1行，`10` 表示第10行。

> `$`：指定最后一行。
>
> `/pattern/`：匹配包含指定模式的行，如 `/foo/` 匹配包含 "foo" 的行。
>
> `addr1, addr2`：指定从 `addr1` 到 `addr2` 之间的行（包括 `addr1` 和 `addr2`），如 `5,10` 表示从第5行到第10行。
>
> `.`：指定当前行。

### 选项

#### `-n`

屏蔽默认输出

```sh
]#sed 'p' user   
root:x:0:0:root:/root:/bin/bash
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin

]# sed -n 'p' user #  未加上-n，会将处理过的所有内容标准输出出来，即上面为初始行，下方为被修改内容行
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin

# sed -nr '/^root|^bin/p' test.txt # 使用扩展正则
```

**案例：**将/bin/bash 换成 /sbin/sh

```sh
# 使用转移字符
]#sed 's/\/bin\/bash/\/sbin\/sh/' user
# 更改提示符，提示符可以任意！# %
]#sed 's#/bin/bash#/sbin/sh#' user
```

#### `-i`

**修改源文件,只有加上-i以上进行操作的内容才生效**

#### `-r`

支持扩展正则，未加r默认使用常用正则模式

```sh
]#sed -n '/^root|^bin/p' # 输出为空
]#sed -nr '/^root|^bin/p' # 输出root开头或则bin开头内容
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
```

**案例：**配置一个httpd网站服务，且端口更改为82

```sh
#!/bin/bash
setenforce 0                            #关闭selinux
yum -y install httpd &> /dev/null        #安装网站
echo  "sed-test~~~" > /var/www/html/index.html        #定义默认页
sed  -i  '/^Listen 80/s/0/2/'  /etc/httpd/conf/httpd.conf #修改配置文件，将监听端口修改为82
systemctl restart httpd                #开服务
systemctl enable httpd    
```

### 指令

#### `p`

内容输出

```sh
]#sed -n '1p'  user# 输出第一行
]#sed -n '2,4p' user# 输出2到4行
]# sed -n '1p;5p' user  # 输出第1行和第5行
]#sed -n '3,+1p' user#输出第三行以及后面一行，及第四行，类似3,4
]#sed -n '1~2p' user # 从第一行，每隔两行输出一次，从第二行开始每个两行输出一次，即输出当前文件的奇数行
]#sed -n '/^root/p' user # 输出以root开头的行
]#sed -n '/root/p' user # 输出包含root的行
]#sed -n '1!p' user # 输出除了第1行的内容
]#sed -n '$p' user # 输出最后一行
]#sed -n '=' user # 输出行号
]#sed -n '$=' user # 输出最后一行的行号
```

#### `d`

内容删除

```sh
]#sed '1d' user # 删除第一行内容，将显示除了第四行所有内容
]#sed '3,5d' user # 删除3,5行内容
]#sed '/root/d' user # 删除所有包含root的行
]#sed '/root/!d' user # 删除不包含root的所有行
]#sed '/^a/d' user #删除以a开头的行
]#sed '$d' user #删除最后一行
]#sed '/^$/d' user # 删除所有空行
```

#### `s`

替换（修改细节）

```sh
]# vim  shu.txt #新建素材
2017 2011 2018
2017 2017 2024
2017 2017 2017
]#sed 's/2017/6666' shu.txt # 把所有行的第一个2017换成6666
]#sed 's/2017/6666/2'shu.txt # 把所有行的第二个2017换成6666
]#sed '1s/2017/6666/'shu.txt # 把第一行的第1个2017换成6666
]#sed '3s/2017/6666/3'shu.txt # 把第三行的第3个2017换成6666
]#sed 's/2017/6666/g'shu.txt # 把所有行的2017换成6666
]#sed '/2024/s/2017/6666/g'shu.txt # 找所有含有2024的行，将2017换成6666

# 替换第几个字符的操作
]#sed 's/.//4' test.txt # 将每行的第四个字符替换为4，.表示任意一个字符，//表示代替换模式，4表示替换的文本
```

#### `i`

行上

```sh
# 行上/下添加内容
]#sed '2i 666' user # 在第2行行上添加一行666
```

#### `a`

行下

```sh
]#sed '1a  666' user # 在第1行行下添加一行666
```

#### `c`

整行替换

```sh
# 替换整行内容
]#sed 'c 666' # 将所有行替换为666
]#sed '$c 666' # 将最后一行替换为666

# 替换特殊使用  将内容调换，类似复制粘贴
]#vim abc
25 yyh   （数字  空格  字母）
355 xiaohang
25 xiaocheng
# 反向引用
]#sed -r 's/([0-9]+)(\s+)([a-z]+)/\3\2\1/' abc 
yyh 52
xiaohang 32
xiaocheng 95
# \3\2\1  ：代表里面第几个元素
# ([0-9]+)(\s+)([a-z]+)
```

# 快捷键

# 问题

## 简述egrep -q

> 静默输出

## 编写正则匹配11位手机号

```
^1[0-9]{10}$
```

## 如何使用sed提取文本中的偶数行

```
sed '2~2p' xx
```



# 补充



# 今日总结

1. ^,$,[],[^],`.`,*,`\{\}`,+,?,{},(),\b,|
2. sed 格式：sed 选项 条件、指令  操作文件
   1. 指令  p、d、s、i、a、c
   2. 条件  /操作条件/
   3. 选项 -n、-i、-r

# 昨日复习
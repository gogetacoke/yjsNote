- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
- [快捷键](#快捷键)
- [问题](#问题)
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

```
]#head -5 /etc/passwd > user # 准备素材
```

### `^` 

匹配前字符开头

```
]#grep '^root' user # 匹配user中以root开头的行
```

### `$`

匹配前字符结尾

```
]#grep 't$' user #匹配以$结尾的行
```

### `[]`

**集合**

匹配单个字符，包含就算

```
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

```
]#grep "[^0-9]" user # 匹配所有数字以外的内容
```

**案例：匹配用户输入的密码是否是数字**

```
#!/bin/bash
read -p "please enter pwd: " p
echo $p | greo -q "[^0-9]" 
if [ $? -eq 0 ];then
	echo "input error"
else
	echo ok
fi
```

代码解释：由于grep查找的只能是文本文件，所以不能直接写grep "`[`^0-9`]`",所以使用echo喊出来进行查询即可，若不用echo，那就将内容写入文件里面，再查询;

grep -q  静默查询，不输出

### `.*`

```
# . 匹配任意单个字符
]#grep "." user # 不写匹配字符，则输出所有内容，因为字符都是有单个字符组成
]#greo "r..t" user # 找rt之间有2个字符的行
]#greo "r.t" user # 找rt之间有1个字符的行，没有匹配内，就无输出
# * 匹配前一个字符任意次数，不允许单独使用

#.* 匹配任意字符的任意次数
```

### `\{n,m}\`



```

```

### `\(\)`

```

```

### `\w`

匹配字母、数字、下划线

```
]#grep 'ro\w' user
```

### `\s`

匹配空格，tab键

```
]#grep '\s' user # 找出所有包含空格的行
```

### `\d`

匹配数字，[0-9];但兼容性较差

```
]#grep -P '\d' user  # 找出所哟包含数字的行
```

## 扩展正则表达式

### `+`

**最少匹配一次**

### `？`

**最多匹配一次**

### `{n，m}`

**匹配前一个字n到m次**

### `()` 

**组合为整体-保留**

### `|`

**或者**

### `\b`

**单词边界**

## 案例

1. 匹配IP地址

   ```
   ^(25[0-5]\.|2[0-4][0-9]\.|1?[0-9]?[0-9]\.){3}(25[0-5]|2[0-4][0-9]|1?[0-9]?[0*9])$
   ^(25[0-5]\.|2[0-4]\d\.|1?\d?\d\.){3}(25[0-5]|2[0-4]\d|1?\d)$
   ```

2. 阿斯顿

3. 阿斯顿

## sed流式编辑器

**逐行处理**

### 使用方式

```
# 两种使用方式，指令必须有
1、 前置指令 | sed 选项 条件 指令
2、 sed 选项 条件 直连 文档
```

### 指令

1. p   显示
2. d    删除（针对一行内容删除）
3. s     替换（修改细节）
4. a    行下追加
5. i      行上添加
6. c     替换整行

### 条件

行号：需要处理那一行

/字符串/：需要处理字符串那一行

### 选项

#### `-n`

屏蔽默认输出

```
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
]#sed -n '1p'  user# 输出第一行
]#sed -n '2,4p' user# 输出2到4行
]# sed -n '1p;5p' user  # 输出第1行和第5行
]#sed -n '3,+1p' user#输出第三行以及后面一行，及第四行，类似3,4
]#sed -n '1~2p' user # 从第一行，每个两行输出一次，从第二行开始每个两行输出一次，即输出当前文件的奇数行
]#sed -n '/^root/p' user # 输出以root开头的行
]#sed -n '/root/p' user # 输出包含root的行
]#sed -n '1!p' user # 输出除了第1行的内容
]#sed -n '$p' user # 输出最后一行
]#sed -n '=' user # 输出行号
```

```
#删除内容
]#sed '1d' user # 删除第一行内容，将显示除了第四行所有内容
]#sed '3,5d' user # 删除3,5行内容
]#sed '/root/d' user # 删除所有包含xml的行
]#sed '/root/!d' user # 删除不包含xml的所有行
]#sed '/^a/d' user #删除以a开头的行
]#sed '$d' user #删除最后一行
]#sed '/^$d/' user # 删除所有空行
```

``` 
# 替换内容
]# vim  shu.txt #新建素材
2017 2011 2018
2017 2017 2024
2017 2017 2017
]#sed 's/2017/6666' shu.txt # 把所有行的第一个2017换成6666
]#sed 's/2017/6666/2'shu.txt # 把所有行的第二个2017换成6666
]#sed '1s/2017/6666/'shu.txt # 把第一行的第1个2017换成6666
]#sed '3s/2017/6666/3'shu.txt # 把第三行的第3个2017换成6666
]#sed 's/2017/6666/g'shu.txt # 把所有行的017换成6666
]#sed '/2024/s/2017/6666/g'shu.txt # 找所有含有2024的行，将2017换成6666
```

```
# 行上/下添加内容
]#sed '2i 666' user # 在第2行行上添加一行666
]#sed '1a 666' user # 在第1行行下添加一行666
```

```
# 替换整行内容
]#sed 'c 666' # 将所有行替换为666
]#sed '$c 666' # 将最后一行替换为666

# 替换特殊使用  将内容调换，类似复制粘贴
]#vim abc
25 yyh   （数字  空格  字母）
355 xiaohang
25 xiaocheng
]#sed -r 's/([0-9]+)(\s+)([a-z]+)/\3\2\1/' abc 
yyh 52
xiaohang 32
xiaocheng 95
# \3\2\1  ：代表里面第几个元素
# ([0-9]+)(\s+)([a-z]+)
```

**案例：**将/bin/bash 换成 /sbin/sh

```
# 使用转移字符
]#sed 's/\/bin\/bash/\/sbin\/sh/' user
# 更改提示符，提示符可以任意！# %
]#sed 's#/bin/bash#/sbin/sh#' user
```

#### `-i`

**修改源文件,只有加上-i以上进行操作的内容才生效**

#### `-r`

支持扩展正则，未加r默认使用常用正则模式

```
]#sed -n '/^root|^bin/p' # 输出为空
]#sed -nr '/^root|^bin/p' # 输出root开头或则bin开头内容
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
```

**案例：**配置一个httpd网站服务，且端口更改为82

```
#!/bin/bash
setenforce 0                            #关闭selinux
yum -y install httpd &> /dev/null        #安装网站
echo  "sed-test~~~" > /var/www/html/index.html        #定义默认页
sed  -i  '/^Listen 80/s/0/2/'  /etc/httpd/conf/httpd.conf #修改配置文件，将监听端口修改为82
systemctl restart httpd                #开服务
systemctl enable httpd    
```



# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习
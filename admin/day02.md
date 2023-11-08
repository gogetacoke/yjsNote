[toc]

# 学习目标

命令行基础

目录及文件管理

文本内容操作

# 课堂笔记（命令）

which firefox

> 查找程序运行所在位置

cat --help

> 任何一个命令加上 --help课堂查看当前命令的所有帮助及选项信息

cat -n /etc/passwd

> 将passwd文本信息输出并显示行号

ls -l /root

> 长格式显示，显示详细属性错的

ls -h /root

> 提供容易读的容量单位（k、m等）

ls -d /root

>  显示目录本（而不是内容）身的属性

ls -A  /root

> 显示所有内容，包含隐藏数据

ls -R /root

> 递归显示目录内容

mount

> 作用：让目录成为设备的访问点
>
> 显示正在挂载的设备关系

mount /dev/cdrom /dvd

> 挂载操作，让目录成为设备访问点，类似windos插入U盘
>
> 以上操作：创建一个空目录，通过挂载命令将目录设置为访问点

umount /dvd

> 卸载类似windos弹出U盘

*（可以一个或多个）

>  通配符，针对不确定文档名词，以特殊字符表示
>
> 例如：显示opt目录下所有a开头的文件或目录
>
> ls /opt/a*
>
> 其余使用： ls /opt/a*.txt
>
> 针对所匹配的是目录使用通配符时：ls -d /opt/a*

？（必须要有一个）

> 单个字符匹配
>
> 例如：
>
> ls /dev/tty？
>
> ls /dev/??tab

[]

>  多个字符或连续范围中的一个，若无则忽略
>
> 例如：ls /dev/tty[0-9]

{}

> 多组不同的字符串，全匹配
>
> 例如：ls /dev/tty{12,45,35}

alias hn=‘hostname’

unalias hn

>   alias：用于给命令起别名，临时定义
>
> 直接输入alias查看系统已存在别名
>
> ​	unalias： 删除定义的别名
>
> 别名优先级高于系统本身存在命令
>
> 永久定义别名
>
> 1. 前往用户家目录，下面以root用户为例
> 2. cd /root
> 3. 修改目录下./bashrc文件
> 4. vim ./bashrc
> 5. 添加需要别名的命令
> 6. alias hn=‘hostname’
> 7. 进入末尾模式保存并退出
> 8. 关闭终端，重新打开即生效

mkdri -p /opt/work/apps

> [-p]：连同父目录一同创建
>
> 在opt中创建work目录再在work目录中创建apps目录

rm /opt/hh

rm -r /opt/hh

rm -rf /opt/hh

> rm 删除文件或目录，且有提示是否删除
>
> [-r] 递归删除目录
>
> [-rf] 强制删除，不提示

mv /opt/1.txt /opt/nsd

> mv：剪切移动，需要两个参数：源数据  目标目录
>
> 将源数据剪切到目标目录中
>
> mv：进行重命名，在目标目录中修改源数据名称
>
> 例如：mv /opt/1.txt /opt/nsd/2.txt

cp /etc/passwd /opt

cp -r /home /opt

\cp -r /home /opt

cp /mnt/1.txt /opt/2.txt

cp -r /root /etc/passwd /opt/cab

cp /etc/hosts .

> cp：复制
>
> [-r]：递归，复制目录
>
> [`\`]：强制覆盖，临时取消别名，针对目录下已经存在源文件，还需进复制
>
> cp：还能在复制中进行重命名
>
> cp：复制可以支持两个以上的参数，永远把最后一个参数作为目标，其他的所有的参数都作为源数据
>
> .：复制与一个点进行连用，将数据复制到当前路径下，点表示当前目录

grep -i Root /etc/passwd

grep -v root /etc/passwd

grep ^root /etc/passwd

grep root$ /etc/passwd

grep ^$ /etc/passwd

> [-i]：忽略大小写进行过滤
>
> [-v]：取反匹配，不包括
>
> [^]：以字符串root开头
>
> [$]：以字符串root结尾
>
> [^$]：过滤空行
>
> 以上选项可以配合使用例如：显示passwd文本中不为空行的所有内容
>
> grep -v ^$ /etc/passwd



# 课堂笔记（文本）

1. Liunx命令执行依赖于解释器

   用户-》解释器（shell）-》内核-》硬件

2. 代码补全功能TAB

   if(tab)(tab) #列出以if开头的命令

3. 光驱设备

   sr0

   所在位置： /dev/sr0   或则 /dev/cdrom

4. 所有命令运行格式

   命令 [选项] 参数

5.  vim编辑时出现交换文件该怎么办

   进入文件所在目录，将其删除

# 快捷键

1. ctrl+c

   结束正在运行的命令

2. alt+.与esc+.

   粘贴上一个命令的**参数**

3. ctrl+u

   从光标处清空至行首

4. ctrl+k

   从光标处清空至行尾

5. home键

   从光标处快速到行首

6. end键

   从光标处快速到行尾

7. ctrl + w

   往回删除一个单词（以空格界定）


# 问题

+ Linux默认的解释器位置

  /bin/bash

+ ～代表什么含义

  用户家目录，存放用户个性化信息的目录

+ 管理员的家目录在那里

  /root

+ 如何去往普通用户家目录

  cd ～/yyh  （系统已经存在这个用户）

  cd /home（查看系统所有的普通用户）


# 总结

1. 在Linux中以.开头的文件或目录将会被隐藏，使用ls -A命令可进行查看

2. Linux中系统中默认的解释器是哪个程序？

/bin/bash

3. 如何结束当前正在运行的命令，快捷键是？

  ctrl+c

4.  如何快速去往用户root的家目录？

  cd /root

5. ls常用选项有哪些？不用写作用

  -l  长格式显示

  -d 显示目录本身，不包括内容

  -h 显示更易读的单位

  -A 显示所有内容，包括隐藏内容

  -R 递归显示

6. 如何查看Linux系统具体版本？

  cat /etc/redhat-release

7. 实现hn相当于运行hostname，如何实现？

  alias hn=‘hostname’

8. 强制删除一个文件夹以及其内容的命令是？

  rm-rf

9.  cp命令如果源数据是一个目录，需要加什么选项？

  cp -r

10. vim编辑器有几种工作模式？分别是什么？

  命令模式、插入模式、末尾模式

11. 过滤/etc/passwd文件内容，不包含root的行，命令是？

   grep -v root /etc/passwd

12. 将/opt重新命名成/student，具体命令是？

   mv /opt /student
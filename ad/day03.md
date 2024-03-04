- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
- [快捷键](#快捷键)
- [问题](#问题)
- [总结](#总结)


# 学习目标

归档及压缩

重定向（重新定向命令的输出）与管道操作

find精确查找

vim高级使用

# 课堂笔记（命令）

打包

tar -zcf /root/di.tar.gz /etc/passwd /etc/shells /home

解压

tar -xf /root/di.tar.gz /nsd10

高级使用

tar -zcf /root/er.tat.gz -C /etc/ passwd shells

tar -zcf /root/san.tar.gz -C /etc/ passwd

> **tar命令进行压缩**
>
> 命令各式：tar 选项 /路径/压缩包名.tar.压缩或解压方式  /被压缩的文件
>
> [-z]：gzip压缩格式
>
> [-j]：bzip2压缩格式
>
> [-J]：xz压缩格式
>
> [-c]：进行打包
>
> [-f]：制定压缩包的名字，必须放在最后
>
> **tar命令进行解压**
>
> 命令格式：tar 选项 /路径/压缩包名 选项2 /释放的位置
>
> [-x]：解压
>
> [-t]：查看tar包内容
>
> [-C]：指定路径（选项2）(打包：目录与文件带空格只打包文件  解包：指定解压路径)
>
> **高级使用**（目的：取消打包中的路径）
>
> 命令格式：tar 选项 /路径 选项2 /路径/ （空格）文件

head -2 /etc/passwd > /opt/2.txt

hostname >> /opt/2.txt

`>` /opt/2.txt

> 作用：重新定向命令的输出操作，屏幕将不会显示输出内容
>
> `>`：覆盖重定向
>
> `>>`：追加重定向
>
> `>`前不加任何命令则是清空文件内容

echo 123

echo 123 > /opt/1.txt

> 作用：输出echo后写的内容，文本复读机

head -3 /etc/passwd `|` tail -1

> ``|``：管道操作，是将`|`前面的输出作为后面的参数来进行的
>
> 上面命令：将passwd中前三行的第三行输出出来
>
> mount cp mv 三个命令不支持管道操作
>
> 管道操作也不支持交互式操作
>
> 例：显示8-12行
>
> head -12 /etc/passwd `|` tail -5
>
> 例：将12-25行内容写入opt下2.txt文本中
>
> head -25 /etc/passwd `|` tail -13 > /opt/2.txt
>
> 例：输出/etc/login.defs中有效信息 ,去除#开头，去除空行
>
> grep -v ^# /etc/login.defs `|` grep -v ^$ 
>
> **过滤命令的输出**
>
> 例：输出ifconfig中含有127的文本内容
>
> ifconfig `|` grep 127
>
> 例：输出ifconfig使用上下键进行查阅
>
> ifconfig `|` less

find /etc -type d

> find精确查找
>
> 格式：find 目录 条件
>
> 上诉命令：查找etc中所有目录
>
> **[-type]按照类型查找**
>
> - [d]：查找类型为目录，包括隐藏文件
> - [f]：查找类型为文件
> - [l]：查找类型为快捷键方式
>
> **[-name]按照名字查找**
>
> 例：查找etc下所有隐藏文件（配合通配符使用）
>
> find /etc -name ‘.*’
>
> 例：配合wc或cat -n进行数量统计
>
> find /etc -name ‘*tab’ `|` wc -l
>
> find /etc -name ‘*tab’ `|` cat -n
>
> **两个条件联合使用**
>
> find /mnt -name ‘cbd*’ -type f 
>
> 上诉代码：查找mnt目录下以cbd开头的所有目录
>
> [-o]：或则，两则满足其一
>
> 使用场景：目录下存在cbd文件和cbd目录通过以下命令满足其一就输出
>
> find /mnt -name ‘cbd*’ **-o** -type f
>
> **[-size]使用+（大于）或-（小于）输出文件大小**
>
> find /boot -size +10k
>
> 写范围：
>
> find /boot -size +10k -size -50k
>
> 单位：k、M、G
>
> **[-user]查找用户（按数据的所有者查询）**
>
> find / -user yyh
>
> **[-mtime]：按照时间进行查找**
>
> -mtime +90 90天前的数据
>
> -mtime -90 90天之内的数据
>
> find /dev -mtime +90
>
> **高级使用**
>
> [注]：处理find找到的数据，每查找的一个就传递一次
>
> 格式：find  [范围]  [条件]  -exec  处理命令  {}   \;
>
> -exec额外操作的开始
>
> {}  永远表示前面find查找的结果
>
> \;  额外操作的结束
>
> 例：在boot下找到大于10M的数据复制到opt下
>
> find /boot -size +10M -size +10M -exec cp {} /opt \;

wc -l /etc/passwd 

>统计当前文件共有多少行，通常配合-l使用
>
>配合管道使用
>
>例：查找boot目录下共有多少个文件
>
>find /boot -type f `|` wc -l

# 课堂笔记（文本）

1. 进入计算器

   bc
   
   输入quit进行退出
   
   配合echo加`|`使用：echo 25*5 `|` bc
   
2. vim高级操作

   **命令行操作**

   | 操作类型       | 按键指令                               | 用途                                                         |
   | -------------- | -------------------------------------- | ------------------------------------------------------------ |
   | 移动光标       | 键盘上下左右                           | 进行上下左右移动                                             |
   | 光标行内体跳转 | Home键<br />End键                      | 跳转到行首<br />跳转到行尾                                   |
   | 全文翻页       | PgUp键、PgDn键                         | 向上翻页、向下翻页                                           |
   | 光标行间跳转   | 1G或gg<br />G                          | 跳转到文件的首行<br />跳转到文件的末尾行                     |
   | 复制           | yy、#yy（#代表数字）                   | 复制光标处的一行、#行                                        |
   | 粘贴           | p、P                                   | 粘贴到光标之后、之前                                         |
   | 删除           | x或Deliet<br />dd、#dd<br />d^<br />d$ | 删除光标出的单个字符<br />删除光标处的一行、#行<br />从光标处之前删除至行首<br />从光标出删除到行尾 |
   | 文本查找       | /word<br />n、N                        | 向后查找字符串‘word’<br />跳至后、前的一个结果               |
   | 撤销编辑       | u<br />U<br />ctrl+r                   | 撤销最近的一次操作<br />撤销对当前行的所有操作<br />取消前一次撤销操作 |
   | 保存退出       | ZZ                                     | 保存修改并退出                                               |

    末行模式

   | 操作类型       | 命令             | 用途                       |
   | -------------- | ---------------- | -------------------------- |
   | 保存           | :w               | 保存当前文件               |
   | 强制不保存退出 | :q!              | 放弃已有更改强制退出       |
   | 另存           | :w /路径/文件名  | 另存为其他文件             |
   | 多文件读入     | :r /路径/文件    | 读入其他文件内容           |
   | 行内替换       | :s /old/new      | 替换当前行的第一个old为new |
   |                | :s /old/new/g    | 替换当前行所有old为new     |
   | 区域内替换     | :n,m s/old/new/g | 替换n-m行所有的old         |
   |                | :% /old/new/g    | 替换全文old为new           |
   | 编辑器设置     | :set nu`|`nonu \| 显示\`\|`不显示行号        ||
   |                | :set ai|`noai | 启用\`\|`不启用自动缩进    ||

   自动缩进：光标自动与上一行对齐 

   vimdiff：

   > 作用：文件内容对比
   >
   > 格式：vimdiff file1 file2
   >
   > 命令模式下ctrl+w，然后左右键移11光标
   >
   > 末行模式wqa保存并退出

# 快捷键



# 问题

1. Linux 压缩格式有那些

   压缩速度，不是绝对，只是在大部分情况下是

   gzip------》.gz 最快，压缩空间比较大

   bzip2-----》.bz2 中

   xz-----》.xz  慢，压缩空间比较小

2. Linux中有效信息

   去除以#开头的注释行和去除空行
   
3. vim设置永久开关功能位置

   vim /root/.vimrc

   [注]：

   初始没有这个这个，需要手动创建

   没一行只能写一个命令
   
4. 请简述一条LInux命令行的一般组成格式

# 总结

1. 将/root/backup.tar.gz压缩包，释放到/opt目录下请写出该命令

   tar -xf /root/backup.tar.gz -C /opt

2. 利用find查找处/etc目录下，所有以".conf"结尾的数据，请写出该命令

   find /etc -name '*.conf'

3. 利用find查找处/etc目录下，只要是目录即可，请写出该命令

   find /etc -type d

4. 如何将/boot目录大于10M的数据，快速拷贝到/opt目录下，请写出该命

   find /boot -size +10M -exec cp {} /opt \;

5. 统计/var目录三个月之前数据的个数，如何操作?

   find /var mtime +90

6. 显示/etc/login.txt文件有效信息，请写出该命令

   grep -v ^# /etc/login.txt `|`grep -v ^$

7. vim编辑器，如何实现，复制光标所在的一行？

   yy

8. vim编辑器，将全文的man替换为MAN如何操作？

   :%s/man/MAN/g

9. vim编辑器中如何开启行号功能？

   :set nu

10. 将"hello world"快速写入到，/opt/cbd.txt文件的最后一行，如何操作？

    echo 'hello word' >> /opt/cbd.txt

11. 显示ifconfig命令输出的前两行，如何操作？

    ifconfig `|` head -2


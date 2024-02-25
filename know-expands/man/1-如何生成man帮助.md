# 创建教程

1. **创建man页面文件**： man页面通常以纯文本格式存储，每个页面都有自己的编号对应不同的手册部分（例如1代表用户命令，2代表系统调用，3代表库函数等）。创建一个新的man页面文件，例如为命令`mycommand`创建一个man页面。

   Bash

   ```bash
   # 创建并编辑名为mycommand.1的文件
   ]#vim mycommand.1
   ```

2. **遵循man页面格式**： man页面由一系列节（section）组成，包括名称节（TH）、描述节（SH）、选项节（SS）、示例节（EX）等。下面是一个简单的模板：

   ```sh
   .TH MYSCRIPT 1 "日期" "版本号" "脚本名称"
   .SH 名称
   myscript - 我的脚本的简短描述
   
   .SH SYNOPSIS
   .B myscript
   [\fI选项\fR] [\fI参数\fR]
   
   .SH 描述
   这里写脚本的详细描述。
   
   .SH 选项
   这里列出脚本的各种选项及其说明。
   
   .SH 参数
   这里列出脚本使用的参数及其说明。
   
   .SH 示例
   这里列出一些脚本的示例用法。
   
   .SH 作者
   作者的姓名和联系信息。
   
   ```

3. **使用groff宏语言**： man页面是用troff（或其扩展版本groff）宏语言编写的，`.TH`、`.SH` 等都是troff的宏命令。

4. **安装man页面**： 编写完成后，将man页面文件放到对应的man手册分区目录下。例如对于命令手册（man1），路径通常是 `/usr/share/man/man1/`。然后更新man数据库：

   ```bash
   ]#gzip mycommand.1
   ]#cp  mycommand.1.gz /usr/share/man/man1/
   ]#mandb
   ```

5. **验证man页面**： 完成上述步骤后，可以使用`man`命令查看新创建的帮助文档：

   ```bash
   ]#man mycommand
   ```

**[注]**：实际操作中可能需要root权限才能安装man页面，具体取决于系统的配置。此外，大型项目通常会包含一个Makefile或者构建脚本来自动化这个过程。

# 案例

> 创建一个基于虚拟机创建的man帮助信息

## man页面文件

```sh
]#vim t.1
.TH create_vm 1 "2024-01-29" "1.0" "自动创建虚拟机"
.SH 名称
create_vm - 自动创建虚拟机

.SH SYNOPSIS
.B create_vm
[\fI选项\fR] [\fI虚拟机名称\fR]

.SH 描述
create_vm 是一个用于自动创建虚拟机的命令。它可以根据指定的参数创建一个新的虚拟机实例。

.SH 选项
这里列出命令的各种选项及其说明。

.TP
\fB-h\fR, \fB--help\fR
显示命令的帮助信息。

.TP
\fB-c\fR, \fB--cpu\fR \fINUM\fR
指定虚拟机的 CPU 核心数。NUM 为一个整数值。

.TP
\fB-m\fR, \fB--memory\fR \fISIZE\fR
指定虚拟机的内存大小。SIZE 可以使用 GB、MB 或 KB 作为单位进行指定。

.TP
\fB-d\fR, \fB--disk\fR \fISIZE\fR
指定虚拟机的磁盘大小。SIZE 可以使用 GB、MB 或 KB 作为单位进行指定。

.SH 参数
这里列出命令使用的参数及其说明。

.TP
\fI虚拟机名称\fR
指定要创建的虚拟机的名称。

.SH 示例
这里列出一些命令的示例用法。

.TP
命令的用法示例1：
.PP
\fBcreate_vm -c 2 -m 4GB -d 50GB my_vm\fR

.TP
命令的用法示例2：
.PP
\fBcreate_vm --cpu 4 --memory 8GB --disk 100GB another_vm\fR

.SH 作者
作者的姓名和联系信息。
```

## 安装man页面

```sh
# 将创建的t.1打包成gz包拷贝到man手册分区
]#gzip t.1 # 原始文件将被压缩成一个新的.gz文件
]#cp t.1.gz /usr/share/man/man1/

# 更新man数据库;
]#mandb
]#ls /usr/share/man/man1/t.1.gz
/usr/share/man/man1/t.1.gz
```

> 系统中的man命令会自动识别并处理`.gz`压缩过的man页面文件
>
> 压缩成`.gz`格式的目地是为了节省磁盘空间和加快 man 页面的检索速度

## 查看结果

```sh
]#man t
create_vm(1)             自动创建虚拟机             create_vm(1)

名称
       create_vm - 自动创建虚拟机

SYNOPSIS
       create_vm [选项] [虚拟机名称]

描述
       create_vm
       是一个用于自动创建虚拟机的命令。它可以根据指定的参数创建一
个新的虚拟机实例。

选项
       这里列出命令的各种选项及其说明。

       -h, --help
              显示命令的帮助信息。

       -c, --cpu NUM
              指定虚拟机的 CPU 核心数。NUM 为一个整数值。
                     -m, --memory SIZE
              指定虚拟机的内存大小。SIZE  可以使用  GB、MB 或 KB
              作为单位进行指定。

       -d, --disk SIZE
              指定虚拟机的磁盘大小。SIZE 可以使用 GB、MB  或  KB
              作为单位进行指定。

参数
       这里列出命令使用的参数及其说明。

       虚拟机名称
              指定要创建的虚拟机的名称。

示例
       这里列出一些命令的示例用法。

       命令的用法示例1：

       create_vm -c 2 -m 4GB -d 50GB my_vm
       
       命令的用法示例2：

       create_vm --cpu 4 --memory 8GB --disk 100GB another_vm

作者
       YYH

1.0                        2024-01-29               create_vm(1)     
```


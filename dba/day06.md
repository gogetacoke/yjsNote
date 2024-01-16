- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
  - [数据备份与恢复(完全)](#数据备份与恢复完全)
    - [基本概论](#基本概论)
    - [物理备份](#物理备份)
      - [冷备份](#冷备份)
      - [恢复](#恢复)
    - [逻辑备份](#逻辑备份)
      - [热备份](#热备份)
      - [恢复](#恢复-1)
    - [备份缺点总结](#备份缺点总结)
      - [测试锁表](#测试锁表)
  - [增量备份](#增量备份)
    - [软件安装](#软件安装)
- [课堂笔记（文本）](#课堂笔记文本)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)


# 学习目标

数据备份与恢复

binlog日志

# 课堂笔记（命令）

## 数据备份与恢复(完全)

### 基本概论

> 备份数据的范式：
>
> + 数据库服务的状态分：
>   + 热备份：mysql服务处于开启状态
>   + 冷备份 ：mysql服务处于停止状态
> + 根据使用的工具分：
>   + 物理备份：在mysql服务器上将/var/lib/mysql 目录拷贝到一个文件夹或进行打包（尽量关闭服务再进行：cp -rp、tar）
>   + 逻辑备份：在命令行使用mysqldump

### 物理备份

#### 冷备份

```sh
# 使用cp拷贝备份
]#systemctl stop mysqld
]#cp -rp /var/lib/mysql /opt/mysql.bak   # -p保留用户权限复制

# 使用tar进行打包备份
]#cd /var/lib/mysql
]#tar -zcf /opt/mysql.tar.gz ./*
```

#### 恢复

```sh
# 删除数据所有数据
]#rm -rf /var/lib/mysql/*

# 数据恢复
]#cp -rp /opt/mysql.bak /var/lib/mysql/
]#systemctl start mysqld
```

### 逻辑备份

> + 备份的是sql命令
> + 完全备份

#### 热备份

```sh
"完全备份---所有数据库"
]#mkdir /bakdir # 用与存储备份文件
]#mysqldump -uroot -pyyh -A > /bakdir/allbak.sql

"完全备份---某一个库,某多个库"
]#mysqldump -uroot -pyyh -B tarena > /bakdir/tarena.sql
]#mysqldump -uroot -pyyh -B tarena db3 > /bakdir/tarena.sql

"完全备份---某一个库中的某一张表,多张表"
]#mysqldump -uroot -pyyh  tarena salary > /bakdir/tarena_salary.sql
]#mysqldump -uroot -pyyh  tarena salary employees > /bakdir/tarena_salary.sql
```

#### 恢复

> 数据库恢复方法：
>
> 1. sehll命令行恢复
> 2. mysql命令行恢复

```shell
# 删除原有和数据
delete from tarena.salary;

# 命令行恢复
mysql -uroot -pyy < /bakdir/tarena_salary.sql
# mysql命令行恢复(如果恢复表，需要提前进入表所在的库)
mysql>source /bakdir/tarena_salary.sql
```

### 备份缺点总结

> Mysqldump 备份和恢复数据时会锁表，锁表期间无法对标做写访问，mysqldump适合备份数据量比较小的数据在数据库服务器访问量少时候备份。

#### 测试锁表

```sql
"格式"
lock tables 库.标 write;
```

```sql
"
测试50机器锁住tarena.user表
51机器进行访问
解锁时看51机器的反应
"
[root@mysql50 ~]#mysql -uroot -pyyh
mysql>lock tables tarena.user write;

[root@mysql50 ~]#mysql -uwebuser -p123456
mysql>select * from tarena.user; 
# 此时命令行将停止


[root@mysql50 ~]#mysql -uroot -pyyh
mysql>unlock tables; # 当在50机器上执行这个命令后，51机器紧接着就能查询出来
```

## 增量备份

### 软件安装

```sh
# 准备软件包---50机器与51机器需要同时安装
percona-xtrabackup-8.0.26-18-Linux-x86_64.glibc2.12-minimal.tar.gz
# 安装依赖
yum -y install perl-DBD-MySQL
```



# 课堂笔记（文本）



# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习
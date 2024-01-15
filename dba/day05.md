- [学习目标](#学习目标)
- [课堂笔记（命令）](#课堂笔记命令)
- [课堂笔记（文本）](#课堂笔记文本)
- [快捷键](#快捷键)
- [问题](#问题)
- [补充](#补充)
- [今日总结](#今日总结)
- [昨日复习](#昨日复习)

# 学习目标

表头高级约束

MySQL索引

用户管理

# 课堂笔记（命令）



# 课堂笔记（文本）

## 主键

![](../pic/dba/d5-1.png)

### 创建主键

> 由于添加主键，不允许赋null值，创建成功后，Null为NO 不允许为空	

```sql
"第一种格式"
mysql> create table db3.t4(
    -> id int,
    -> name char(3),
    -> idCar char(18),
    -> primary key(idCar)
    -> );
    
mysql> desc db3.t4;
+-------+----------+------+-----+---------+-------+
| Field | Type     | Null | Key | Default | Extra |
+-------+----------+------+-----+---------+-------+
| id    | int      | YES  |     | NULL    |       |
| name  | char(3)  | YES  |     | NULL    |       |
| idCar | char(18) | NO   | PRI | NULL    |       |
+-------+----------+------+-----+---------+-------+
3 rows in set (0.01 sec)
 
 
 
"第二种格式"
mysql> create table db3.t5(
    -> id int,
    -> name char(3),
    -> idCar char(18) primary key
    -> );
    
mysql> desc db3.t5;
+-------+----------+------+-----+---------+-------+
| Field | Type     | Null | Key | Default | Extra |
+-------+----------+------+-----+---------+-------+
| id    | int      | YES  |     | NULL    |       |
| name  | char(3)  | YES  |     | NULL    |       |
| idCar | char(18) | NO   | PRI | NULL    |       |
+-------+----------+------+-----+---------+-------+
3 rows in set (0.00 sec)

```

### 删除主键

> [注]：删除主键后，不能为空将不会改变，需要手动修改

```sql
mysql> alter table db3.t4 drop primary key;
Query OK, 0 rows affected (1.31 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc db3.t4;
+-------+----------+------+-----+---------+-------+
| Field | Type     | Null | Key | Default | Extra |
+-------+----------+------+-----+---------+-------+
| id    | int      | YES  |     | NULL    |       |
| name  | char(3)  | YES  |     | NULL    |       |
| idCar | char(18) | NO   |     | NULL    |       |
+-------+----------+------+-----+---------+-------+
3 rows in set (0.00 sec)

"手动再修改可以为空"
mysql> alter table db3.t4 modify idCar char(18);
Query OK, 0 rows affected (1.34 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc db3.t4;
+-------+----------+------+-----+---------+-------+
| Field | Type     | Null | Key | Default | Extra |
+-------+----------+------+-----+---------+-------+
| id    | int      | YES  |     | NULL    |       |
| name  | char(3)  | YES  |     | NULL    |       |
| idCar | char(18) | YES  |     | NULL    |       |
+-------+----------+------+-----+---------+-------+
3 rows in set (0.00 sec)



mysql> select * from db3.t4;
+------+------+------------+
| id   | name | idCar      |
+------+------+------------+
|    1 | bob  | 511322     |
|    1 | ab   | 5113221999 |
+------+------+------------+
2 rows in set (0.00 sec)
```

```sql
"已有字段-》添加主键: 一张表中只能有一个主键"
mysql> alter table db3.t4 add primary key(idCar);
Query OK, 0 rows affected (1.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc db3.t4;
+-------+----------+------+-----+---------+-------+
| Field | Type     | Null | Key | Default | Extra |
+-------+----------+------+-----+---------+-------+
| id    | int      | YES  |     | NULL    |       |
| name  | char(3)  | YES  |     | NULL    |       |
| idCar | char(18) | NO   | PRI | NULL    |       |
+-------+----------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```

### 验证主键

```sql
mysql> insert into db3.t4 values(1,"bob","511322");
Query OK, 1 row affected (0.07 sec)

mysql> insert into db3.t4 values(1,"coke","511322");
ERROR 1406 (22001): Data too long for column 'name' at row 1

mysql> insert into db3.t4 values(1,"ab","5113221999");
Query OK, 1 row affected (0.11 sec)

mysql> insert into db3.t4 values(1,"ab",null);
ERROR 1048 (23000): Column 'idCar' cannot be null
 
```

## 复合主键

> + 多个表头一起做主键
> + 复合主键的值不允许同时重复

### 创建

```sql
mysql> create table db3.t6( 
    ip varchar(15), 
    port char(2), 
    status enum("deny","allow"),
    primary key(ip,port));
Query OK, 0 rows affected (0.49 sec)

mysql> desc db3.t6;
+--------+----------------------+------+-----+---------+-------+
| Field  | Type                 | Null | Key | Default | Extra |
+--------+----------------------+------+-----+---------+-------+
| ip     | varchar(15)          | NO   | PRI | NULL    |       |
| port   | char(2)              | NO   | PRI | NULL    |       |
| status | enum('deny','allow') | YES  |     | NULL    |       |
+--------+----------------------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```

### 删除

```sql
mysql> alter table db3.t6 drop primary key;
Query OK, 0 rows affected (1.00 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc db3.t6;
+--------+----------------------+------+-----+---------+-------+
| Field  | Type                 | Null | Key | Default | Extra |
+--------+----------------------+------+-----+---------+-------+
| ip     | varchar(15)          | NO   |     | NULL    |       |
| port   | char(2)              | NO   |     | NULL    |       |
| status | enum('deny','allow') | YES  |     | NULL    |       |
+--------+----------------------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```

### 添加

```sql
mysql> alter table db3.t6 add primary key(ip,port);
Query OK, 0 rows affected (0.74 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc db3.t6;
+--------+----------------------+------+-----+---------+-------+
| Field  | Type                 | Null | Key | Default | Extra |
+--------+----------------------+------+-----+---------+-------+
| ip     | varchar(15)          | NO   | PRI | NULL    |       |
| port   | char(2)              | NO   | PRI | NULL    |       |
| status | enum('deny','allow') | YES  |     | NULL    |       |
+--------+----------------------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```

### 验证

```sql
mysql> insert into db3.t6 values("1.1.1.1","80","deny");
Query OK, 1 row affected (0.07 sec)

"两个主键的值相同者报错"
mysql> insert into db3.t6 values("1.1.1.1","80","allow");
ERROR 1062 (23000): Duplicate entry '1.1.1.1-80' for key 't6.PRIMARY'

mysql> insert into db3.t6 values("1.1.1.1","22","allow");
Query OK, 1 row affected (0.08 sec)

mysql> insert into db3.t6 values("2.1.1.1","22","allow");
Query OK, 1 row affected (0.08 sec)

mysql> select * from db3.t6;
+---------+------+--------+
| ip      | port | status |
+---------+------+--------+
| 1.1.1.1 | 22   | allow  |
| 1.1.1.1 | 80   | deny   |
| 2.1.1.1 | 22   | allow  |
+---------+------+--------+
3 rows in set (0.00 sec)
```

## 主键自增长

> + 表头要有自增长 表头必须有主键设置才可以
> + 通过表头自加1计算结果赋值

### 创建

```sql
mysql> create table db3.t7(
    -> id int primary key auto_increment,
    -> name char(3),
    -> age int
    -> );
mysql> desc t7;
+-------+---------+------+-----+---------+----------------+
| Field | Type    | Null | Key | Default | Extra          |
+-------+---------+------+-----+---------+----------------+
| id    | int     | NO   | PRI | NULL    | auto_increment |
| name  | char(3) | YES  |     | NULL    |                |
| age   | int     | YES  |     | NULL    |                |
+-------+---------+------+-----+---------+----------------+
3 rows in set (0.00 sec)
```

### 测试

```sql
mysql> insert into db3.t7(name,age) values("bob",15);
Query OK, 1 row affected (0.08 sec                         
mysql> insert into db3.t7(name,age) values("jay",20);
Query OK, 1 row affected (0.08 sec)

mysql> select * from db3.t7;
+----+------+------+
| id | name | age  |
+----+------+------+
|  1 | bob  |   15 |
|  2 | jay  |   20 |
+----+------+------+
2 rows in set (0.00 sec)                          
```

### 扩展1-续行

> + 根据表中最后一行的序号进行自增长

```sql
mysql> insert into db3.t7 values(5,"jay",20);
Query OK, 1 row affected (0.05 sec)

mysql> select * from db3.t7;
+----+------+------+
| id | name | age  |
+----+------+------+
|  1 | bob  |   15 |
|  2 | jay  |   20 |
|  5 | jay  |   20 |
+----+------+------+
3 rows in set (0.00 sec)

mysql> insert into db3.t7(name,age) values("cok",35);
Query OK, 1 row affected (0.09 sec)

mysql> select * from db3.t7;
+----+------+------+
| id | name | age  |
+----+------+------+
|  1 | bob  |   15 |
|  2 | jay  |   20 |
|  5 | jay  |   20 |
|  6 | cok  |   35 |
+----+------+------+
4 rows in set (0.00 sec)
```

### 扩展2-删除表数据

> + delete from 删除自增长列，将继续编号
> + truncate table 删除自增长列，将从1开始

```sql
"delete-删除"
mysql> delete from db3.t7;
Query OK, 4 rows affected (0.20 sec)

mysql> select * from db3.t7;
Empty set (0.00 sec)

mysql> insert into db3.t7(name,age) values("bob",16),("yyh",22);
Query OK, 2 rows affected (0.25 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> select * from db3.t7;
+----+------+------+
| id | name | age  |
+----+------+------+
|  7 | bob  |   16 |
|  8 | yyh  |   22 |
+----+------+------+
2 rows in set (0.00 sec)
```

```sql
"truncate 删除"
mysql> truncate table db3.t7;
Query OK, 0 rows affected (0.60 sec)

mysql> insert into db3.t7(name,age) values("bob",16),("yyh",22);
Query OK, 2 rows affected (0.08 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> select * from db3.t7;
+----+------+------+
| id | name | age  |
+----+------+------+
|  1 | bob  |   16 |
|  2 | yyh  |   22 |
+----+------+------+
2 rows in set (0.00 sec)

```

### 扩展3-主键删除

> 删除带有主键的自增长：进行删除时应该**先删除自增长**，**再删除主键**

```sql
mysql> alter table db3.t7 drop primary key;
ERROR 1075 (42000): Incorrect table definition; there can be only one auto column and it must be defined as a key

"使用修改表将自增长"
mysql> alter table db3.t7 modify id int;
Query OK, 2 rows affected (1.05 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> desc db3.t7;
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
| id    | int     | NO   | PRI | NULL    |       |
| name  | char(3) | YES  |     | NULL    |       |
| age   | int     | YES  |     | NULL    |       |
+-------+---------+------+-----+---------+-------+
3 rows in set (0.00 sec)

mysql> alter table db3.t7 drop primary key;
Query OK, 2 rows affected (0.79 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> desc db3.t7;
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
| id    | int     | NO   |     | NULL    |       |
| name  | char(3) | YES  |     | NULL    |       |
| age   | int     | YES  |     | NULL    |       |
+-------+---------+------+-----+---------+-------+
3 rows in set (0.00 sec)

```

### 案例1

> 给表user添加主键自增长，使得给每一行添加一个id

```sql
"未修改前，希望获得第三行数据
1.先查出所有数据
2.找出第三行特殊名称
3.使用where条件
"
mysql> select * from user where name="daemon";
+--------+----------+------+------+---------+---------+---------------+
| name   | password | uid  | gid  | comment | homedir | shell         |
+--------+----------+------+------+---------+---------+---------------+
| daemon | x        |    2 |    2 | daemon  | /sbin   | /sbin/nologin |
+--------+----------+------+------+---------+---------+---------------+
1 row in set (0.00 sec)

"修改后，添加一个id设置自增长,放在第一行"
mysql> alter table db3.user add id int primary key auto_increment first;
mysql> select * from user where id=3;
+----+--------+----------+------+------+---------+---------+---------------+
| id | name   | password | uid  | gid  | comment | homedir | shell         |
+----+--------+----------+------+------+---------+---------+---------------+
|  3 | daemon | x        |    2 |    2 | daemon  | /sbin   | /sbin/nologin |
+----+--------+----------+------+------+---------+---------+---------------+
1 row in set (0.00 sec)
```

## 外键

> 使用规则：
>
> + 表存储引擎必须是innodb(mysql8 默认为该引擎)
> + 表头数据类型要一致
> + 被参照表头必须要是索引类型的一种
>
> 作用：
>
> + 插入记录时，表头值再另一个表的表头值范围内选择

### 创建外键

```sql
"创建测试数据-员工表yg_tab"
mysql> create table db3.yg_tab(
    -> yg_id int primary key auto_increment,
    -> name char(4)
    -> );
    
"基于员工表创建工资表gz_tab，创建外键"
mysql> create table db3.gz_tab(
    -> gz_id int,
    -> pay float,
    -> foreign key(gz_id) references db3.yg_tab(yg_id) on update cascade on delete cascade);
Query OK, 0 rows affected (0.71 sec)
```

### 查询表信息

> 查询外键信息

```sql
mysql> show create table db3.gz_tab\G
*************************** 1. row ***************************
Table: gz_tab
Create Table: CREATE TABLE `gz_tab` (
  `gz_id` int DEFAULT NULL,
  `pay` float DEFAULT NULL,
  KEY `gz_id` (`gz_id`),
  CONSTRAINT `gz_tab_ibfk_1` FOREIGN KEY (`gz_id`) REFERENCES `yg_tab` (`yg_id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
1 row in set (0.00 sec)
```

### 删除外键

> + 存在外键的表，无法进行删除，需要先删除外键信息才能删除
> + 根据索引名进行删除

```sql
"案例1--->员工表中的yg_id与工资表gz_id存在外键关系"
mysql> drop table db3.yg_tab;
ERROR 3730 (HY000): Cannot drop table 'yg_tab' referenced by a foreign key constraint 'gz_tab_ibfk_1' on table 'gz_tab'

"案例2--->删除员工表中的数据"
mysql> truncate table db3.yg_tab;
ERROR 1701 (42000): Cannot truncate a table referenced in a foreign key constraint (`db3`.`gz_tab`, CONSTRAINT `gz_tab_ibfk_1`)
```

```sql
"需要先在gz_tab表中删除外键关系，才能进行删除yg_tab表或表数据"
mysql> alter table db3.gz_tab drop foreign key gz_tab_ibfk_1;
"查看删除后的信息"
mysql> show create table db3.gz_tab\G
*************************** 1. row ***************************
       Table: gz_tab
Create Table: CREATE TABLE `gz_tab` (
  `gz_id` int DEFAULT NULL,
  `pay` float DEFAULT NULL,
  KEY `gz_id` (`gz_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
1 row in set (0.00 sec)
```

**[注]：若要删除员工表的数据，还需删除工资表的数据，以防出错**

### 添加外键

> + 创建表后未添加外键时使用，或修改时

```sql
mysql> alter table db3.gz_tab add foreign key(gz_id) references db3.yg_tab(yg_id) on update cascade on delete cascade;


mysql> show create table db3.gz_tab\G
*************************** 1. row ***************************
       Table: gz_tab
Create Table: CREATE TABLE `gz_tab` (
  `gz_id` int DEFAULT NULL,
  `pay` float DEFAULT NULL,
  KEY `gz_id` (`gz_id`),
  CONSTRAINT `gz_tab_ibfk_1` FOREIGN KEY (`gz_id`) REFERENCES `yg_tab` (`yg_id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

### 验证

**外键字段的值必须在参考表字段值范围内**

```sql
mysql> insert into db3.yg_tab(name) values("bob"),("tom"),("yyh");
mysql> insert into db3.gz_tab values(1,300),(2,800),(3,1000);

mysql> select yg_tab.*,pay from yg_tab inner join gz_tab on yg_id=gz_id;
+-------+------+------+
| yg_id | name | pay  |
+-------+------+------+
|     1 | bob  |  300 |
|     2 | tom  |  800 |
|     3 | yyh  | 1000 |
+-------+------+------+
```

**外键字段的值必须*不*在参考表字段值范围内**

```sql
mysql> select * from yg_tab;
+-------+------+
| yg_id | name |
+-------+------+
|     1 | bob  |
|     2 | tom  |
|     3 | yyh  |
+-------+------+

"当在工资表中插入不存在的员工id的数据时，由上方查处员工id没有4"
mysql> insert into db3.gz_tab values(4,1200);
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`db3`.`gz_tab`, CONSTRAINT `gz_tab_ibfk_1` FOREIGN KEY (`gz_id`) REFERENCES `yg_tab` (`yg_id`) ON DELETE CASCADE ON UPDATE CASCADE)

"在yg_tab中加入4号员工即可解决"
mysql> insert into db3.yg_tab(name) values("xh");

mysql> select * from db3.yg_tab;
+-------+------+
| yg_id | name |
+-------+------+
|     1 | bob  |
|     2 | tom  |
|     3 | yyh  |
|     4 | xh   |
+-------+------+
mysql> insert into db3.gz_tab values(4,1200);

mysql> select * from gz_tab;
+-------+------+
| gz_id | pay  |
+-------+------+
|     1 |  300 |
|     2 |  800 |
|     3 | 1000 |
|     4 | 1200 |
+-------+------+
4 rows in set (0.00 sec)
```

**验证同步更新**

> + 当父表(yg_tab)发生变化后子表(gz_tab)也跟着变化

```sql
"修改员工表中的3号员工id为6"
mysql> update db3.yg_tab set yg_id=6 where yg_id=3;
mysql> select * from db3.yg_tab;
+-------+------+
| yg_id | name |
+-------+------+
|     1 | bob  |
|     2 | tom  |
|     4 | xh   |
|     6 | yyh  |
+-------+------+

"查询工资表中该员工id是否更新"
mysql> select * from gz_tab;
+-------+------+
| gz_id | pay  |
+-------+------+
|     1 |  300 |
|     2 |  800 |
|     6 | 1000 |
|     4 | 1200 |
+-------+------+
```

**验证同步删除**

> + 员工表删除信息后，工资表同步更新删除

```sql
"在yg_tab中删除yg_id=6，查看工资表gz_tab中是否同步删除"
mysql> delete from db3.yg_tab where yg_id=6;

mysql> select * from gz_tab;
+-------+------+
| gz_id | pay  |
+-------+------+
|     1 |  300 |
|     2 |  800 |
|     4 | 1200 |
+-------+------+
```

### 优化

> 根据上方配置后**存在以下问题：**
>
> + 没有员工编号的也能进行发工资
>
>   ```sql
>   mysql> insert into db3.gz_tab values(null,1800);
>   mysql> select * from gz_tab;
>   +-------+------+
>   | gz_id | pay  |
>   +-------+------+
>   |     1 |  300 |
>   |     2 |  800 |
>   |     4 | 1200 |
>   |  NULL | 1800 |
>   +-------+------+
>   ```
>
> + 可重复给员工发工资
>
>   ```sql
>   mysql>insert into db3.gz_tab values(1,300);
>   mysql> select * from gz_tab;
>   +-------+------+
>   | gz_id | pay  |
>   +-------+------+
>   |     1 |  300 |
>   |     2 |  800 |
>   |     4 | 1200 |
>   |  NULL | 1800 |
>   |     1 |  300 |
>   +-------+------+
>   ```

**解决：在工资表中的gz_id加上主键**

```sql
"存在错误数据，先删除全部数据"
mysql> delete from db3.gz_tab;
"未添加主键时表结构"
mysql> desc db3.gz_tab;                                                           +-------+-------+------+-----+---------+-------+
| Field | Type  | Null | Key | Default | Extra |
+-------+-------+------+-----+---------+-------+
| gz_id | int   | YES  | MUL | NULL    |       |
| pay   | float | YES  |     | NULL    |       |
+-------+-------+------+-----+---------+-------+
"添加主键"
mysql> alter table db3.gz_tab add primary key(gz_id);
mysql> desc db3.gz_tab;
+-------+-------+------+-----+---------+-------+
| Field | Type  | Null | Key | Default | Extra |
+-------+-------+------+-----+---------+-------+
| gz_id | int   | NO   | PRI | NULL    |       |
| pay   | float | YES  |     | NULL    |       |
+-------+-------+------+-----+---------+-------+
2 rows in set (0.00 sec)

"插入数据并验证"
mysql> insert into db3.gz_tab values(1,300);
mysql> select *  from gz_tab;
+-------+------+
| gz_id | pay  |
+-------+------+
|     1 |  300 |
+-------+------+
mysql> insert into db3.gz_tab values(1,5000);
ERROR 1062 (23000): Duplicate entry '1' for key 'gz_tab.PRIMARY'
mysql> insert into db3.gz_tab values(null,5000);
ERROR 1048 (23000): Column 'gz_id' cannot be null
```

> + 观察表结构，在设置了外键时Key为MUL，加上主键后变为PRI
> + 在进行删除时，需先删除外键，才能删除主键(不能为空还需手动设置)

# 快捷键



# 问题



# 补充



# 今日总结



# 昨日复习
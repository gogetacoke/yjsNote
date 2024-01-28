# MySQL调优

## 复杂的查询语句

### 启用慢查询日志

> 保存耗时较长查询命令
>
> /var/lib/mysql/主机名-slow.log  慢查询日志存放地

```perl
"
slow-query-log 启动慢查询日志
long-query-time 设置查询时间(执行一条查询语句如果在设置的时间内没有返回，将会记录这条日志)
"
# 开启慢查询日志
[root@mysql50 ~]# vim /etc/my.cnf.d/mysql-server.cnf 
[mysqld]
slow-query-log
long-query-time=0.3 # 设置查询时间大于0.3就像该条日志记录在慢查询日志中
[root@mysql50 ~]#systemctl restart mysqld # 重启生效配置
# 查询数据库目录是否有该文件
[root@mysql50 ~]# wc -l /var/lib/mysql/mysql50-slow.log
3 /var/lib/mysql/mysql50-slow.log 
```

```perl
"
测试，使用sleep函数进行测试1s与5s
"
[root@mysql50 ~]#mysql -uroot -p123
mysql> select sleep(1), "yyh";
+----------+-----+
| sleep(1) | yyh |
+----------+-----+
|        0 | yyh |
+----------+-----+
1 row in set (1.00 sec)

mysql> select sleep(5), "china";
+----------+-------+
| sleep(5) | china |
+----------+-------+
|        0 | china |
+----------+-------+
1 row in set (5.00 sec)

"查询慢查询日志"
[root@mysql50 ~]# cat /var/lib/mysql/mysql50-slow.log
......
SET timestamp=1706398329;
select sleep(1), "yyh";
# Time: 2024-01-27T23:32:24.798957Z 
# User@Host: root[root] @ localhost []  Id:     8
# Query_time: 5.000200  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
SET timestamp=1706398339;
select sleep(5), "china";
"使用命令统计慢查询命令"
[root@mysql50 ~]# mysqldumpslow /var/lib/mysql/mysql50-slow.log 
Reading mysql slow query log from /var/lib/mysql/mysql50-slow.log
Count: 2  Time=3.00s (6s)  Lock=0.00s (0s)  Rows=1.0 (2), root[root]@localhost
  select sleep(N), "S"
# 根据结果看到总共有两条慢查询日志，可通过管道导出整理发给开发人员
[root@mysql50 ~]# mysqldumpslow /var/lib/mysql/mysql50-slow.log > sql.txt
# 整理结果如下
[root@mysql50 ~]# cat sql.txt 
  select sleep(N), "S"
```


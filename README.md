mysql运维之服务器日志。（east_sun参考文档）
============================

概要：
---

MySQL日志类型分为***1.服务器日志（server_log）***和***2.事务日志（transaction_log）***。
其中在实际运维中，经常使用的服器日志有：服务器错误日志，慢查询日志和综合查询日志。事务日志经常使用的是：存储引擎事务日志，二进制日志。

**1.1服务器日志之服务器错误日志(error_log)：**

服务器错误日志主要记录MySQL运行过程中的Error、Warning、Note等信息，系统出错或者某条记录出问题可以查看Error日志。

```
mysql> SHOW GLOBAL VARIABLES LIKE 'log_error' ;＃＃此句SQL出了服务器错误日志的具体位置：
+---------------+----------------------------------------+
| Variable_name | Value                                  |
+---------------+----------------------------------------+
| log_error     | /usr/local/mysql/data/mysqld.local.err |
+---------------+----------------------------------------+
1 row in set (0.00 sec)

```


然后进入log_error文件用less 读取内容，可以看到Error、Warning、Note等信息，需要强调一点，
log_error里面不斗全是错误信息，同时用户使用记录实例启动中重要信息和配置参数log_error文档信息。
当DBA遇到无法启动mysql时候，需要第一时间想到查看log_error。


**1.2服务器日志之慢查询日志（log_slow）：**

	服务器日志分类下日志的慢日志查询主要用途是用来记录在MySQL中响应时间超过阀值的语句，
  具体指运行时间超过long_query_time设置值，默认值为10秒,同时可以这个参数是精确到微秒
  的（microseconds）。MySQL官方文档中(https://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html)，
  慢查询日志解释主要如下：
	
	
	The slow query log consists of SQL statements that took more than 
  long_query_time seconds to execute and required at least min_examined_row_limit 
  rows to be examined. The minimum and default values of long_query_time are 
  0 and 10, respectively. The value can be specified to a resolution of microseconds.
  For logging to a file, times are written including the microseconds part. For logging
  to tables, only integer times are written; the microseconds part is ignored.


	
```
mysql> SHOW GLOBAL VARIABLES LIKE '%slow%'; **##查看slow_log相关日志**
+---------------------------+--------------------------------------------------------+
| Variable_name             | Value                                                  |
+---------------------------+--------------------------------------------------------+
| log_slow_admin_statements | OFF                                                    |
| log_slow_slave_statements | OFF                                                    |
| slow_launch_time          | 2                                                      |
| slow_query_log            | OFF                                                    |
| slow_query_log_file       | /usr/local/mysql/data/louhaomiaodeMacBook-Pro-slow.log |
+---------------------------+--------------------------------------------------------+
5 rows in set (0.01 sec)

mysql> SET GLOBAL slow_query_log = 1 ; **##开启错误日志查询功能（默认一般关闭）**
Query OK, 0 rows affected (0.02 sec)

mysql> SHOW GLOBAL VARIABLES LIKE '%slow%'; **##查看slow_log相关日志**
+---------------------------+--------------------------------------------------------+
| Variable_name             | Value                                                  |
+---------------------------+--------------------------------------------------------+
| log_slow_admin_statements | OFF                                                    |
| log_slow_slave_statements | OFF                                                    |
| slow_launch_time          | 2                                                      |
| slow_query_log            | ON                                                     |
| slow_query_log_file       | /usr/local/mysql/data/louhaomiaodeMacBook-Pro-slow.log |
+---------------------------+--------------------------------------------------------+
5 rows in set (0.01 sec)
mysql> SHOW GLOBAL VARIABLES LIKE'%long%'; ##默认为十秒
+----------------------------------------------------------+-----------+
| Variable_name                                            | Value     |
+----------------------------------------------------------+-----------+
| long_query_time                                          | 10.000000 |
| performance_schema_events_stages_history_long_size       | 1000      |
| performance_schema_events_statements_history_long_size   | 1000      |
| performance_schema_events_transactions_history_long_size | 1000      |
| performance_schema_events_waits_history_long_size        | 1000      |
+----------------------------------------------------------+-----------+
5 rows in set (0.00 sec)

mysql> SET GLOBAL long_query_time = 5 ;##一般线上业务设为五秒。
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW GLOBAL VARIABLES LIKE'%long%';
+----------------------------------------------------------+----------+
| Variable_name                                            | Value    |
+----------------------------------------------------------+----------+
| long_query_time                                          | 5.000000 |
| performance_schema_events_stages_history_long_size       | 1000     |
| performance_schema_events_statements_history_long_size   | 1000     |
| performance_schema_events_transactions_history_long_size | 1000     |
| performance_schema_events_waits_history_long_size        | 1000     |
+----------------------------------------------------------+----------+
5 rows in set (0.01 sec)

```


# 1. 日志

|日志||
|:-|:-|
|事务日志 |transaction log|
|错误日志 |error log|
|通用日志 |general log|
|慢查询日志 |slow query log|
|二进制日志 |binary log|
|中继日志 |reley log|

# 2. 事务日志 transaction log

事务日志的写入类型为“追加”，因此其操作为“顺序 IO” ；通常也被称为：预写式日志 write ahead logging。

事务日志文件： ib_logfile0，ib_logfile1。

事务日志：事务型存储引擎自行管理和使用，建议和数据文件分开存放。
- redo log
- undo log

## 2.1. Innodb事务日志相关配置

```
show variables like '%innodb_log%';

innodb_log_file_size 5242880    ##每个日志文件大小
innodb_log_files_in_group 2     ##日志组成员个数
innodb_log_group_home_dir ./    ##事务文件路径
innodb_flush_log_at_trx_commit  ##默认为1
```
```
[mysqld]
innodb_log_group_hone_dir = /data/logs

mkdir /data/logs
chown mysql.mysql /data/logs
```

## 2.2. innodb_flush_log_at_trx_commit  

设置为1，同时 sync_binlog = 1 表示最高级别的容错。  

innodb_use_global_flush_log_at_trx_commit的值确定是否可以使用SET语句重置此变量。

||描述|
|:-|:-|
|1|默认情况下，日志缓冲区将写入日志文件，并在每次事务后执行刷新到磁盘。这是完全遵守ACID特性。|
|0|提交时没有任何操作；而是每秒执行一次日志缓冲区写入和刷新。这样可以提供更好的性能，但服务器崩溃可以清除最后一秒的事务。|
|2|每次提交后都会写入日志缓冲区，但每秒都会进行一次刷新。性能比 0 略好一些，但操作系统或停电可能导致最后一秒的交易丢失。|
|3|模拟 MariaDB 5.5组提交（每组提交3个同步），此项 MariaDB 10.0支持。|


事务日志优化  
此处有图


# 3. 错误日志

错误日志内容
- mysqld 启动和关闭过程中输出的事件信息。
- mysqld 运行中产生的错误信息。
- event scheduler 运行一个 event 时产生的日志信息。
- 在主从复制架构中的从服务器上启动从服务器线程时产生的信息。

错误日志相关配置
```
SHOW GLOBAL VARIABLES LIKE 'log_error'


## 错误文件路径
[mysql_safe]
log_error=/PATH/TO/LOG_ERROR_FILE

## 是否记录警告信息至错误日志文件
## 默认值1
log_warnings=1|0    

```

# 4. 通用日志

通用日志：记录对数据库的通用操作，包括错误的SQL语句。
- 文件：file，默认值
- 表：table

通用日志相关设置
```
## 开关
general_log=ON|OFF

## 文件格式
general_log_file=HOSTNAME.log

## 存储方式
log_output=TABLE|FILE|NONE
```
```
set globle log_output = table
show tables general_log;
show table status like 'general_log'\G


查看 argument 字段
```

# 5. 慢查询日志

慢查询日志：记录执行查询时长超出指定时长的操作。

|慢查询日志相关设置|描述|
|:-|:-|
|slow_query_log=ON\|OFF |开启或关闭慢查询。|
|long_query_time=N |慢查询的阀值，单位秒。|
|slow_query_log_file=HOSTNAME-slow.log |慢查询日志文件。|
|log_slow_filter = |admin,filesort,filesort_on_disk,full_join,full_scan,query_cache,query_cache_miss,tmp_table,tmp_table_on_disk。<br>上述查询类型且查询时长超过 long_query_time，则记录日志。|
|log_queries_not_using_indexes=ON |不使用索引或使用全索引扫描，不论是否达到慢查询阈值的语句是否记录日志，默认OFF，即不记录。|
|log_slow_rate_limit = 1 |多少次查询才记录，mariadb 特有。|
|log_slow_verbosity= Query_plan,explain |记录内容。|
|log_slow_queries = OFF  |同 slow_query_log，新版已废弃。|

```
[mysqld]
slow_query_log
log_queries_not_using_indexes
## 建议启用


select @@long_query_time;
```

```
select @@profiling ;
set profiling = on;


show profiles;
## 查看一下编号
show profiles for query 5;
## 查看查询的哪个阶段
```


# 6. 中继日志：relay log

主从复制架构中，从服务器用于保存从主服务器的二进制日志中读取的事件。

# 7. 二进制日志

二进制日志
- 记录导致数据改变或潜在导致数据改变的SQL语句。
- 记录已提交的日志。

日志内容不依赖于存储引擎类型。

功能：通过“重放”日志文件中的事件来生成数据副本。

注意：建议二进制日志和数据文件分开存放。

## 7.1. 二进制日志记录格式

二进制日志记录三种格式

|二进制日志记录格式|描述|
|:-|:-|
|statement|基于“语句”，记录语句，默认模式。|
|row|基于“行”，记录数据，日志量较大。|
|mixed|混合模式，让系统自行判定该基于哪种方式进行。|

格式配置
```
show variables like 'binlog_format';
```

## 7.2. 二进制日志文件的构成

有两类文件
- 日志文件：mysql|mariadb-bin。文件名后缀，二进制格式。  
如：mariadb-bin.000001
- 索引文件：mysql|mariadb-bin.index，文本格式。

## 7.3. 二进制日志相关的服务器变量

|二进制日志相关的服务器变量|描述|
|:-|:-|
sql_log_bin=ON\|OFF|是否记录二进制日志，默认ON
|log_bin=/PATH/BIN_LOG_FILE|指定文件位置；默认OFF，表示不启用二进制日志功能，上述两项都开启才可。|
|binlog_format=STATEMENT\|ROW\|MIXED|二进制日志记录的格式，默认 STATEMENT。
|max_binlog_size=1073741824|单个二进制日志文件的最大体积，到达最大值会自动滚动，默认为1G。<br>说明：文件达到上限时的大小未必为指定的精确值|
|sync_binlog=1\|0|设定是否启动二进制日志即时同步磁盘功能，默认0，由操作系统负责同步日志到磁盘。|
|expire_logs_days=N|二进制日志可以自动删除的天数。 默认为0，即不自动删除二进制日志。|

```
set sql_log_bin = OFF  ## session级别禁用
## 可以临时禁用，可用于大批量倒入数据、数据恢复。
```
```
[mysqld]
log_bin				##默认文件前缀mariadb-bin
log_bin=/data/logbin/mysql-bin
```

## 7.4. 二进制日志相关配置

查看 mariadb 自行管理使用中的二进制日志文件列表，及大小。
```
SHOW {BINARY | MASTER} LOGS
```
查看使用中的二进制日志文件
```
SHOW MASTER STATUS
```
查看二进制文件中的指定内容
```
SHOW BINLOG EVENTS [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count]
show binlog events in 'mysql-bin.000001' from 6516 limit 2,3
```
切换日志
```
FLUSH LOGS;
```

## 7.5. mysqlbinlog

二进制日志的客户端命令工具。

### 7.5.1. 命令格式

```
mysqlbinlog [OPTIONS] log_file …
```

### 7.5.2. 选项

- --start-position=# 指定开始位置
- --stop-position=#
- --start-datetime=
- --stop-datetime=  
- --base64-output[=name]
- -v -vvv

时间格式：YYYY-MM-DD hh:mm:ss

### 7.5.3. 示例

```
mysqlbinlog --start-position=6787 --stop-position=7527 /var/lib/mysql/mariadb-bin.000003 -v

mysqlbinlog --start-datetime="2018-01-30 20:30:10" --stopdatetime="2018-01-30 20:35:22" mariadb-bin.000003 -vvv
```

## 7.6. 二进制日志事件的格式

```
# at 328
# 151105 16:31:40 server id 1 end_log_pos 431 Query thread_id=1 exec_time=0 error_code=0
use `mydb`/*!*/;
SET TIMESTAMP=1446712300/*!*/;
CREATE TABLE tb1 (id int, name char(30))
/*!*/;
```
```
事件发生的日期和时间：151105 16:31:40
事件发生的服务器标识：server id 1
事件的结束位置：end_log_pos 431
事件的类型：Query
事件发生时所在服务器执行此事件的线程的ID：thread_id=1
语句的时间戳与将其写入二进制文件中的时间差：exec_time=0
错误代码：error_code=0
事件内容：
```
GTID：Global Transaction ID，mysql5.6 以 mariadb10 以上版本专属属性：GTID日志

## 7.7. 清除指定二进制日志

清除文件，而不是内容。
```
PURGE { BINARY | MASTER } LOGS
    { TO 'log_name' | BEFORE datetime_expr }
```

示例：
```
PURGE BINARY LOGS TO 'mariadb-bin.000003';  ## 删除3之前的日志
PURGE BINARY LOGS BEFORE '2017-01-23';
PURGE BINARY LOGS BEFORE '2017-03-22 09:25:30';
```
## 7.8. RESET

删除所有二进制日志，index 文件重新记数
```
RESET MASTER [TO #]; 
```
删除所有二进制日志文件，并重新生成日志文件，文件名从#开始记数，默认从1开始，一般是 master 主机第一次启动时执行，
MariaDB10.1.6开始支持“TO #”。

## 7.9. 切换日志文件

```
FLUSH LOGS;
```
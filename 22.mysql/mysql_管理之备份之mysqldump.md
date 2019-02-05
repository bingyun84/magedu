# 1. mysqldump

逻辑备份工具：mysqldump, mydumper, phpMyAdmin

Schema 和数据存储在一起、巨大的SQL语句、单个巨大的备份文件。

mysqldump 工具：客户端命令，通过 mysql 协议连接至 mysql 服务器进行备份。

# 2. 语法

```
mysqldump [OPTIONS] database [tables]
mysqldump [OPTIONS] -B DB1 [DB2 DB3...]
mysqldump [OPTIONS] -A [OPTIONS]
```
参考：
https://dev.mysql.com/doc/refman/5.7/en/mysqldump.html

## 2.1. mysqldump常见选项

|选项|描述|
|:-|:-|
|-A, --all-databases |备份所有数据库，包括 create database 语句。|
|-B, --databases db_name… |指定备份的数据库，包括 create database 语句。|
|-E, --events|备份相关的所有 event scheduler。|
|-R, --routines|备份所有存储过程和自定义函数。|
|--triggers|备份表相关触发器，默认启用，用 --skip-triggers，不备份触发器。|
|--default-character-set=utf8 |指定字符集。|
|--master-data[=#]|此选项须启用二进制日志。<br>1：所备份的数据之前加一条记录为 CHANGE MASTER TO 语句，非注释，不指定#，默认为1。<br>2：记录为注释的 CHANGE MASTER TO 语句。<br>此选项会自动关闭 --lock-tables 功能，自动打开 -x \| --lock-all-tables 功能（除非开启--single-transaction）。|
|-F, --flush-logs |备份前滚动日志，锁定表完成后，执行 flush logs 命令，生成新的二进制日志文件，配合 -A 或 -B 选项时，会导致刷新多次数据库。<br>建议在同一时刻执行转储和日志刷新，可通过和 --single-transaction 或 -x，--master-data 一起使用实现，此时只刷新一次日志。|
|--compact |去掉注释，适合调试，生产不使用。|
|-d, --no-data |只备份表结构。|
|-t, --no-create-info |只备份数据，不备份 create table。|
|-n, --no-create-db |不备份 create database，可被 -A 或 -B 覆盖。|
|--flush-privileges |备份 mysql 或相关时需要使用。|
|-f, --force |忽略 SQL 错误，继续执行。|
|--hex-blob |使用十六进制符号转储二进制列，当有包括 BINARY、VARBINARY、BLOB、BIT 的数据类型的列时使用，避免乱码。|
|-q, --quick |不缓存查询，直接输出，加快备份速度。|

## 2.2. MyISAM备份选项

MyISAM 存储引擎支持温备；不支持热备，所以必须先锁定要备份的库，而后启动备份操作
锁定方法如下：

|选项|描述|
|:-|:-|
|-x, --lock-all-tables|加全局读锁，锁定所有库的所有表。同时加 --singletransaction 或 --lock-tables 选项会关闭此选项功能。<br>注意：数据量大时，可能会导致长时间无法并发访问数据库。|
|-l, --lock-tables|对于需要备份的每个数据库，在启动备份之前分别锁定其所有表，默认为 on，--skip-lock-tables 选项可禁用，对备份 MyISAM 的多个库,可能会造成数据不一致。|

注意：以上选项对 InnoDB 表一样生效，实现温备，但不推荐使用。

## 2.3. InnoDB备份选项： 

支持热备，可用温备但不建议用

|选项|描述|
|:-|:-|
|--single-transaction|此选项 Innodb 中推荐使用，不适用 MyISAM，此选项会开始备份前，先执行 START TRANSACTION 指令开启事务。|

此选项通过在单个事务中转储所有表来创建一致的快照。仅适用于存储在支持多版本控制的存储引擎中的表（目前只有 InnoDB 可以）；转储不保证与其他存储引擎保持一致。在进行单事务转储时，要确保有效的转储文件（正确的表内容和二进制日志位置），没有其他连接应该使用以下语句：ALTER TABLE，DROP TABLE，RENAME TABLE，TRUNCATE TABLE。

此选项和 --lock-tables（此选项隐含提交挂起的事务）选项是相互排斥。

备份大型表时，建议将 --single-transaction 选项和 --quick 结合一起使用。

# 3. 生产环境实战备份策略
## 3.1. InnoDB建议备份策略

```
mysqldump -uroot -A -F -E -R --single-transaction --master-data=1 --
flush-privileges --triggers --default-character-set=utf8 --hex-blob
>$BACKUP/fullbak_$BACKUP_TIME.sql
```

## 3.2. MyISAM建议备份策略

```
mysqldump -uroot -A -F -E -R -x --master-data=1 --flush-privileges --
triggers --default-character-set=utf8 --hex-blob
>$BACKUP/fullbak_$BACKUP_TIME.sql
```
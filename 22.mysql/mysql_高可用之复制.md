# 1. MySQL复制
扩展方式： Scale Up ，Scale Out  
MySQL的扩展
- 读写分离  
复制：每个节点都有相同的数据集。
- - 向外扩展
- - 二进制日志
- - 单向

复制的功用：
- 数据分布
- 负载均衡读
- 备份
- 高可用和故障切换
- MySQL升级测试

一主一从  
一主多从  
主从复制原理  
MySQL垂直分区  
MySQL水平分片（Sharding）  
对应shard中查询相关数据  


主从复制线程：
- 主节点：  
dump Thread：为每个 Slave 的 I/O Thread 启动一个 dump 线程，用于向其发送 binary log events。

- 从节点：  
I/O Thread：向 Master 请求二进制日志事件，并保存于中继日志中。  
SQL Thread：从中继日志中读取日志事件，在本地完成重放。

跟复制功能相关的文件：
- master.info：用于保存 slave 连接至 master 时的相关信息，例如账号、密码、
服务器地址等。
- relay-log.info：保存在当前 slave 节点上已经复制的当前二进制日志和本地 replay log 日志的对应关系。

主从复制特点
- 异步复制
- 主从数据不一致比较常见

复制架构：
- Master/Slave, Master/Master, 环状复制
- 一主多从
- 从服务器还可以再有从服务器
- 一从多主：适用于多个不同数据库

复制需要考虑二进制日志事件记录格式
- STATEMENT（5.0之前）
- ROW（5.1之后，推荐）
- MIXEDMySQL


复制模型  
此处有图



# 2. 主从配置过程

参看官网  
https://mariadb.com/kb/en/library/setting-up-replication/  
https://dev.mysql.com/doc/refman/5.5/en/replication-configuration.html

## 2.1. 主节点配置

(1) 启用二进制日志
```
[mysqld]
log_bin
```

(2) 为当前节点设置一个全局惟一的ID号
```
[mysqld]
server_id=#
log-basename=master                 ## 可选项，设置datadir中日志名称，确保不依赖主机名
```

(3) 创建有复制权限的用户账号
```
GRANT REPLICATION SLAVE ON *.* TO 'repluser'@'HOST' IDENTIFIED BY 'replpass';
```

## 2.2. 从节点配置

(1) 启动中继日志
```
[mysqld]
server_id=#                         ## 为当前节点设置一个全局惟的ID号
read_only=ON                        ## 设置数据库只读
relay_log=relay-log                 ## relay log的文件路径，默认值 hostname-relay-bin
relay_log_index=relay-log.index     ## 默认值 hostname-relay-bin.index
```

(2) 使用有复制权限的用户账号连接至主服务器，并启动复制线程
```
mysql> CHANGE MASTER TO 
MASTER_HOST='host',
MASTER_USER='repluser', 
MASTER_PASSWORD='replpass',
MASTER_LOG_FILE='mariadb-bin.xxxxxx', 
MASTER_LOG_POS=#;

mysql> START SLAVE [IO_THREAD|SQL_THREAD];
```

如果主节点已经运行了一段时间，且有大量数据时，如何配置并启动 slave 节点。
- 通过备份恢复数据至从服务器
- 复制起始位置为备份时，二进制日志文件及其POS

如果要启用级联复制，需要在从服务器启用以下配置
```
[mysqld]
log_bin
log_slave_updates
```

## 2.3. 复制架构中应该注意的问题

1、限制从服务器为只读
```
在从服务器上设置
[mysqld]
read_only=ON
```
注意：此限制对拥有SUPER权限的用户均无效。

阻止所有用户，包括主服务器复制的更新。
```
mysql> FLUSH TABLES WITH READ LOCK;
```

2、 RESET SLAVE  
在从服务器清除 master.info，relay-log.info，relay log，开始新的 relay
log。注意：需要先 STOP SLAVE。  
RESET SLAVE ALL 清除所有从服务器上设置的主服务器同步信息如：PORT，HOST，USER 和 PASSWORD 等。
```
STOP SLAVE
RESET SLAVE
RESET SLAVE ALL
```

3、 sql_slave_skip_counter = N 从服务器忽略几个主服务器的复制事件，global 变量。

4、 如何保证主从复制的事务安全  
参看：https://mariadb.com/kb/en/library/server-system-variables/

在master节点启用参数：
```
sync_binlog=1                       ## 每次写后立即同步二进制日志到磁盘，性能差

如果用到的为InnoDB存储引擎：
innodb_flush_log_at_trx_commit=1    ## 每次事务提交立即同步日志写磁盘
innodb_support_xa=ON                ## 默认值，分布式事务MariaDB10.3.0废除
sync_master_info=#                  ## #次事件后master.info同步到磁盘
```

在slave节点启用服务器选项：
```
skip_slave_start=ON                 ## 不自动启动slave
```

在slave节点启用参数：
```
sync_relay_log=#                    ## #次写后同步relay log到磁盘
sync_relay_log_info=#               ## #次事务后同步relay-log.info到磁盘
```

# 3. 主主复制

主主复制：互为主从。

容易产生的问题：数据不一致；因此慎用。

考虑要点：自动增长id。 
``` 
// 配置一个节点使用奇数 id
auto_increment_offset=1             ## 开始点
auto_increment_increment=2          ## 增长幅度

// 另一个节点使用偶数id
auto_increment_offset=2
auto_increment_increment=2
```
## 3.1. 主主复制的配置步骤：

(1) 各节点使用一个惟一 server_id  
(2) 都启动 binary log 和 relay log  
(3) 创建拥有复制权限的用户账号  
(4) 定义自动增长id字段的数值范围各为奇偶  
(5) 均把对方指定为主节点，并启动复制线程  

# 4. 半同步复制

默认情况下，MySQL的复制功能是异步的，异步复制可以提供最佳的性能，主库把 binlog 日志发送给从库即结束，并不验证从库是否接收完毕。这意味着当主服务器或从服务器端发生故障时，有可能从服务器没有接收到主服务器发送过来的 binlog 日志，这就会造成主服务器和从服务器的数据不一致，甚至在恢复时造成数据的丢失。

## 4.1. 半同步复制实现：

主服务器配置:
```
mysql> INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
mysql> SET GLOBAL rpl_semi_sync_master_enabled=1;
mysql> SET GLOBAL rpl_semi_sync_master_timeout = 1000;                      ## 超时长为1s
mysql> SHOW GLOBAL VARIABLES LIKE '%semi%';
mysql> SHOW GLOBAL STATUS LIKE '%semi%';
```

从服务器配置:
```
mysql> INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';
mysql> SET GLOBAL rpl_semi_sync_slave_enabled=1;
```

# 5. 复制过滤器：

让从节点仅复制指定的数据库，或指定数据库的指定表。

两种实现方式：  
(1) 服务器选项：主服务器仅向二进制日志中记录与特定数据库相关的事件。  
注意：此项和 binlog_format 相关。  
参看：https://mariadb.com/kb/en/library/mysqld-options/#-binlogignore-db
```
binlog_do_db =                      ## 数据库白名单列表，多个数据库需多行实现
binlog_ignore_db =                  ## 数据库黑名单列表
```
问题：基于二进制还原将无法实现；不建议使用。

(2) 从服务器 SQL_THREAD 在 replay 中继日志中的事件时，仅读取与特定数据库(特定表)相关的事件并应用于本地。  
问题：会造成网络及磁盘IO浪费。

从服务器上的复制过滤器相关变量
```
replicate_do_db=                    ## 指定复制库的白名单
replicate_ignore_db=                ## 指定复制库黑名单
replicate_do_table=                 ## 指定复制表的白名单
replicate_ignore_table=             ## 指定复制表的黑名单
replicate_wild_do_table=foo%.bar%   ## 支持通配符
replicate_wild_ignore_table=
```

# 6. MySQL复制加密

基于 SSL 复制：  
在默认的主从复制过程或远程连接到 MySQL/MariaDB 所有的链接通信中的数据都是明文的，外网里访问数据或则复制，存在安全隐患。通过 SSL/TLS 加密的方式进行复制的方法，来进一步提高数据的安全性。

## 6.1. 配置实现：

参看：https://mariadb.com/kb/en/library/replication-with-secureconnections/

- 主服务器开启SSL：[mysqld] 加一行 ssl。  
- 主服务器配置证书和私钥；并且创建一个要求必须使用SSL连接的复制账号。  
- 从服务器使用CHANGER MASTER TO 命令时指明 ssl 相关选项。  

Master服务器配置
```
[mysqld]
log-bin
server_id=1
ssl
ssl-ca=/etc/my.cnf.d/ssl/cacert.pem
ssl-cert=/etc/my.cnf.d/ssl/master.crt
ssl-key=/etc/my.cnf.d/ssl/master.key
```

Slave服务器配置
```
mysql>
CHANGE MASTER TO
MASTER_HOST='MASTERIP',
MASTER_USER='rep',
MASTER_PASSWORD='centos',
MASTER_LOG_FILE='mariadb-bin.000001',
MASTER_LOG_POS=245,
MASTER_SSL=1,
MASTER_SSL_CA = '/etc/my.cnf.d/ssl/cacert.pem',
MASTER_SSL_CERT = '/etc/my.cnf.d/ssl/slave.crt',
MASTER_SSL_KEY = '/etc/my.cnf.d/ssl/slave.key';
```

# 7. 复制的监控和维护

(1) 清理日志
```
PURGE { BINARY | MASTER } LOGS { TO 'log_name' | BEFORE datetime_expr }
RESET MASTER
RESET SLAVE
```

(2) 复制监控
```
SHOW MASTER STATUS
SHOW BINLOG EVENTS
SHOW BINARY LOGS
SHOW SLAVE STATUS
SHOW PROCESSLIST
```

(3) 从服务器是否落后于主服务
```
Seconds_Behind_Master: 0
```

(4) 如何确定主从节点数据是否一致  
- percona-tools

(5) 数据不一致如何修复  
- 删除从数据库，重新复制。
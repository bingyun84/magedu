# 1. 存储引擎
InnoDB support for FULLTEXT indexes is available in MySQL 5.6.4 and later.

# 2. 存储引擎比较
https://docs.oracle.com/cd/E17952_01/mysql-5.5-en/storage-engines.html

## 2.1. MyISAM引擎
### 2.1.1. 特性
- 不支持事务
- 表级锁定
- 读写相互阻塞，写入不能读，读时不能写
- 只缓存索引
- 不支持外键约束
- 不支持聚簇索引
- 读取数据较快，占用资源较少
- 不支持MVCC（多版本并发控制机制）高并发
- 崩溃恢复性较差
- MySQL5.5.5前默认的数据库引擎

MyISAM存储引擎适用场景：只读（或者写较少）、表较小（可以接受长时间进行修复操作）

### 2.1.2. MyISAM引擎文件

|MyISAM引擎文件|描述|
|:-|:-|
|tbl_name.frm |表格式定义|
|tbl_name.MYD |数据文件|
|tbl_name.MYI |索引文件|

## 2.2. InnoDB引擎
### 2.2.1. 特性
- 行级锁
- 支持事务，适合处理大量短期事务
- 读写阻塞与事务隔离级别相关
- 可缓存数据和索引
- 支持聚簇索引
- 崩溃恢复性更好
- 支持MVCC高并发
- 从MySQL5.5后支持全文索引
- 从MySQL5.5.5开始为默认的数据库引擎

### 2.2.2. InnoDB数据库文件  
所有InnoDB表的数据和索引放置于同一个表空间中。表空间文件：datadir定义的目录下。数据文件：ibddata1, ibddata2, ...  

|文件|描述|
|:-|:-|
|tb_name.frm|表格式定义|
|tb_name.ibd|数据文件(存储数据和索引)|  

每个表单独使用一个表空间存储表的数据和索引，需要启用：innodb_file_per_table=ON，(>= MariaDB 5.5)  
参看：https://mariadb.com/kb/en/library/xtradbinnodb-serversystem-variables/#innodb_file_per_table 

## 2.3. 其它存储引擎

|其它存储引擎|描述|
|:-|:-|
| Performance_Schema|Performance_Schema数据库使用。|
| Memory |将所有数据存储在RAM中，以便在需要快速查找参考和其他类似数据的环境中进行快速访问。适用存放临时数据。引擎以前被称为HEAP引擎。|
| MRG_MyISAM|使MySQL DBA或开发人员能够对一系列相同的MyISAM表进行逻辑分组，并将它们作为一个对象引用。适用于VLDB(Very Large Data Base)环境，如数据仓库。|
| Archive |为存储和检索大量很少参考的存档或安全审核信息，只支持 SELECT 和 INSERT 操作；支持行级锁和专用缓存区。|
| Federated联合|用于访问其它远程MySQL服务器一个代理，它通过创建一个到远程MySQL服务器的客户端连接，并将查询传输到远程服务器执行，而后完成数据存取，提供链接单独MySQL服务器的能力，以便从多个物理服务器创建一个逻辑数据库。非常适合分布式或数据集市环境。|
| BDB|可替代InnoDB的事务引擎，支持COMMIT、 ROLLBACK和其他事务特性。|
| Cluster/NDB|MySQL的簇式数据库引擎，尤其适合于具有高性能查找要求的应用程序，这类查找需求还要求具有最高的正常工作时间和可用性。|
| CSV|CSV存储引擎使用逗号分隔值格式将数据存储在文本文件中。可以使用CSV引擎以CSV格式导入和导出其他软件和应用程序之间的数据交换。|
| BLACKHOLE |黑洞存储引擎接受但不存储数据，检索总是返回一个空集。该功能可用于分布式数据库设计，数据自动复制，但不是本地存储。|
| example|“stub”引擎，它什么都不做。可以使用此引擎创建表，但不能将数据存储在其中或从中检索。目的是作为例子来说明如何开始编写新的存储引擎。|

MariaDB支持的其它存储引擎：
- OQGraph
- SphinxSE
- TokuDB
- Cassandra
- CONNECT
- SQUENCE

# 3. 管理存储引擎
查看mysql支持的存储引擎
```
show engines;
```

查看当前默认的存储引擎
```
show variables like '%storage_engine%';
```

设置默认的存储引擎
```
vim /etc/my.conf
[mysqld]
default_storage_engine = InnoDB
```

查看库中所有表使用的存储引擎
```
show table status from db_name;
```

查看库中指定表的存储引擎
```
show table status like ' tb_name ';
show create table tb_name;
```

设置表的存储引擎：
```
CREATE TABLE tb_name(... ) ENGINE=InnoDB;
ALTER TABLE tb_name ENGINE=InnoDB;
```
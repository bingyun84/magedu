# 1. 并发控制

MVCC：多版本并发控制，和事务级别相关。

# 2. 锁

锁粒度：
- 表级锁
- 行级锁

锁类型：
- 读锁：共享锁，只读不可写，多个读互不阻塞。
- 写锁：独占锁，排它锁，一个写锁会阻塞其它读和写锁。

实现
- 存储引擎：自行实现其锁策略和锁粒度。
- 服务器级：实现行锁，表级锁；用户可显式请求。

分类：
- 隐式锁：由存储引擎自动施加锁。
- 显式锁：用户手动请求。

锁策略：在锁粒度及数据安全性寻求的平衡机制。

## 2.1. 显式使用锁

加锁
```
LOCK TABLES 
    tbl_name [[AS] alias] lock_type
    [, tbl_name [[AS] alias] lock_type] ...
```
lock_type：READ，WRITE
```
lock table students read;
lock table students write;
```

解锁
```
UNLOCK TABLES 
```
```
## 退出会话也会解锁
exit
```

关闭正在打开的表（清除查询缓存），通常在备份前加全局读锁。
未指定表名，则对整个实例的表加锁。包括mysql数据库。
可以配合备份（myisam）使用，温备。
```
FLUSH TABLES [tb_name[,...]] [WITH READ LOCK]
```

查询时加写或读锁
```
SELECT clause [FOR UPDATE | LOCK IN SHARE MODE]
```

```
mysql> show processlist;
mysql> kill 4
```

# 3. 事务

事务Transactions：一组原子性的SQL语句，或一个独立工作单元。

事务日志：记录事务信息，实现undo，redo等故障恢复功能。

## 3.1. ACID特性：

|ACID|描述|
|:-|:-|
|原子性，A，atomicity |整个事务中的所有操作要么全部成功执行，要么全部失败后回滚。|
|一致性，C，Consistency |数据库总是从一个一致性状态转换为另一个一致性状态。|
|隔离性，I，Isolation |一个事务所做出的操作在提交之前，是不能为其它事务所见；隔离有多种隔离级别，实现并发。|
|持久性，D，Durability|一旦事务提交，其所做的修改会永久保存于数据库中。|


Transaction生命周期  
此处有图


## 3.2. 管理事务
### 3.2.1. 启动事务

```
BEGIN
BEGIN WORK
START TRANSACTION
```
### 3.2.2. 结束事务

```
## 提交
COMMIT

## 回滚
ROLLBACK
```
注意：只有事务型存储引擎中的 DML 语句方能支持此类操作。

### 3.2.3. 自动提交

```
[mysqld]
autocommit = 1   ## 默认

set autocommit={1|0} 

select @@autocommit;
```
默认为1，为0时设为非自动提交。

建议：显式请求和提交事务，而不要使用“自动提交”功能。

### 3.2.4. 保存点：savepoint

```
## 创建保存点
SAVEPOINT identifier

## 回滚到保存点
ROLLBACK [WORK] TO [SAVEPOINT] identifier

## 释放所有保存点
RELEASE SAVEPOINT identifier
```

## 3.3. 事务隔离级别

事务隔离级别：从上至下更加严格。

|事务隔离级别|描述|
|:-|:-|
| READ UNCOMMITTED |可读取到未提交数据，产生脏读。|
| READ COMMITTED |可读取到提交数据，但未提交数据不可读，产生不可重复读，即可读取到多个提交数据，导致每次读取数据不一致。|
| REPEATABLE READ |可重复读，多次读取数据都一致，产生幻读，即读取过程中，即使有其它提交的事务修改数据，仍只能读取到未修改前的旧数据。此为MySQL默认设置。|
| SERIALIZABILE |可串行化，未提交的读事务阻塞修改事务，或者未提交的修改事务阻塞读事务。导致并发性能差。|

事务隔离级别

|事务隔离级别 |脏读可能性 |不可重复读可能性 |幻读可能性 |加锁读|
|:-|:-:|:-:|:-:|:-:|
|读未提交（read-uncommitted） |是 |是 |是 |否|
|不可重复读（read-committed） |否 |是 |是 |否|
|可重复读（repeatable-read） |否 |否 |是 |否|
|串行化（serializable） |否 |否 |否 |是|

## 3.4. 指定事务隔离级别

服务器变量 tx_isolation 指定，默认为 REPEATABLE-READ，可在 GLOBAL 和 SESSION 级进行设置。
```
SET tx_isolation=''
            READ-UNCOMMITTED
            READ-COMMITTED
            REPEATABLE-READ
            SERIALIZABLE


show variables list 'tx_isolation';
```

服务器选项中指定
```
vim /etc/my.cnf
[mysqld]
transaction-isolation=SERIALIZABLE
```

# 4. 死锁

两个或多个事务在同一资源相互占用，并请求锁定对方占用的资源的状态。

解决方法：避免交叉改表。
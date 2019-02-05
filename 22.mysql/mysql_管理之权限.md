# 1. MySQL权限管理  
# 2. 权限类别

- 管理类  
- 程序类  
- 数据库级别  
- 表级别  
- 字段级别  

## 2.1. 管理类

CREATE TEMPORARY TABLES  
CREATE USER  
FILE  
SUPER  
SHOW DATABASES  
RELOAD  
SHUTDOWN  
REPLICATION SLAVE  
REPLICATION CLIENT  
LOCK TABLES  
PROCESS  

## 2.2. 程序类： FUNCTION、 PROCEDURE、 TRIGGER

CREATE  
ALTER  
DROP  
EXCUTE  

## 2.3. 库和表级别：DATABASE、 TABLE

ALTER  
CREATE  
CREATE VIEW  
DROP  
INDEX  
SHOW VIEW  
GRANT OPTION：能将自己获得的权限转赠给其他用户

## 2.4. 数据操作

SELECT  
INSERT  
DELETE  
UPDATE  

## 2.5. 字段级别

SELECT(col1,col2,...)  
UPDATE(col1,col2,...)  
INSERT(col1,col2,...)  

## 2.6. 所有权限

ALL PRIVILEGES 或 ALL

# 3. 授权

参考：https://dev.mysql.com/doc/refman/5.7/en/grant.html

## 3.1. 语法

```
GRANT priv_type [(column_list)],... ON [object_type] priv_level TO 'user'@'host'
[IDENTIFIED BY 'password'] [WITH GRANT OPTION];
```

priv_type: ALL [PRIVILEGES]

object_type:TABLE | FUNCTION | PROCEDURE

priv_level: \*(所有库) | \*.\* | db_name.\* | db_name.tbl_name | tbl_name(当前库的表) | db_name.routine_name(指定库的函数,存储过程,触发器)

with_option: GRANT OPTION  
| MAX_QUERIES_PER_HOUR count  
| MAX_UPDATES_PER_HOUR count  
| MAX_CONNECTIONS_PER_HOUR count  
| MAX_USER_CONNECTIONS count  

示例：
```
GRANT SELECT (col1), INSERT (col1,col2) ON mydb.mytbl TO 'someuser'@'somehost';
```
```
grant all on hellodb.* to test@'192.168.34.%';


## 创建用户并授权
grant select (name,age),update (age) on hellodb.students to test2@'192.168.34.%' identified by 'centos';

```

# 4. 回收授权

## 4.1. 语法

```
REVOKE priv_type [(column_list)] [, priv_type [(column_list)]] ... ON [object_type] priv_level FROM user [, user] ...
```

示例：
```
REVOKE DELETE ON testdb.* FROM 'testuser'@‘172.16.0.%’;
```

# 5. 查看指定用户获得的授权

```
Help SHOW GRANTS
SHOW GRANTS FOR 'user'@'host';
SHOW GRANTS FOR CURRENT_USER[()];
```

# 6. 注意

MariaDB服务进程启动时会读取mysql库中所有授权表至内存  
(1) GRANT或REVOKE等执行权限操作会保存于系统表中，MariaDB的服务进程通常会自动重读授权表，使之生效。  
(2) 对于不能够或不能及时重读授权表的命令，可手动让MariaDB的服务进程重读授权表：
```
mysql> FLUSH PRIVILEGES;
```
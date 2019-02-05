# 1. DML语句
DML：
- INSERT
- DELETE
- UPDATE

# 2. INSERT：
一次插入一行或多行数据

## 2.1. 语法
```
INSERT [L OW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
[INTO] tbl_name 
    [(col_name,...)]
{VALUES | VALUE} 
    ({expr | DEFAULT},...),(...),...
[ ON DUPLICATE KEY UPDATE       ##如果重复更新之
    col_name=expr
    [, col_name=expr] ... ]

简化写法：
INSERT tbl_name [(col1,...)] VALUES (val1,...), (val21,...)
```
```
INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
[INTO] tbl_name
SET col_name={expr | DEFAULT}, ...
[ ON DUPLICATE KEY UPDATE
    col_name=expr
    [, col_name=expr] ... ]
```
```
INSERT [LOW_PRIORITY | HIGH_PRIORITY] [IGNORE]
[INTO] tbl_name [(col_name,...)]
SELECT ...
[ ON DUPLICATE KEY UPDATE
    col_name=expr
    [, col_name=expr] ... ]
```

# 3. UPDATE：
## 3.1. 语法
```
UPDATE [LOW_PRIORITY] [IGNORE] table_reference
    SET col_name1={expr1|DEFAULT} [, col_name2={expr2|DEFAULT}] ...
[WHERE where_condition]
[ORDER BY ...]
    [LIMIT row_count]
```

## 3.2. 注意：
一定要有限制条件，否则将修改所有行的指定字段。where 条件后 key 为主键，否则也是不能update或delete。

限制条件：
```
WHERE
LIMIT
```

Mysql 选项：
```
-U|--safe-updates| --i-am-a-dummy
```

配置文件
```
[mysql]
safe-updates = 1
```

示例
```
MariaDB [db1]> update  t3 set id=1;
ERROR 1175 (HY000): You are using safe update mode and you tried to update a table without a WHERE that uses a KEY column

MariaDB [db1]> delete from t3;
ERROR 1175 (HY000): You are using safe update mode and you tried to update a table without a WHERE that uses a KEY column

MariaDB [db1]> desc t2;
+-------+------------------+------+-----+---------+----------------+
| Field | Type             | Null | Key | Default | Extra          |
+-------+------------------+------+-----+---------+----------------+
| id    | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| name  | varchar(5)       | YES  |     | NULL    |                |
| sex   | enum('f','m')    | NO   |     | NULL    |                |
+-------+------------------+------+-----+---------+----------------+
3 rows in set (0.00 sec)

MariaDB [db1]> select * from t2;
+----+------+-----+
| id | name | sex |
+----+------+-----+
|  1 | a    | m   |
|  2 | b    | f   |
|  3 | c    | m   |
+----+------+-----+
3 rows in set (0.00 sec)

MariaDB [db1]> delete from t2 where name='b';
ERROR 1175 (HY000): You are using safe update mode and you tried to update a table without a WHERE that uses a KEY column
MariaDB [db1]> delete from t2 where id=2;
Query OK, 1 row affected (0.00 sec)
```
# 4. DELETE:
## 4.1. 语法
```
DELETE [LOW_PRIORITY] [QUICK] [IGNORE] 
FROM tbl_name
[WHERE where_condition]
[ORDER BY ...]
[LIMIT row_count]   ## 可先排序再指定删除的行数
```

## 4.2. 注意：
一定要有限制条件，否则将清空表中的所有数据。

如何限制，见delete

## 4.3. 清空表
```
TRUNCATE TABLE tbl_name; 
```
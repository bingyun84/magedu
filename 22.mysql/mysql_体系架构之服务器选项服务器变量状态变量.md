
# 1. mysqld选项、服务器系统变量和服务器状态变量
## 1.1. 官方
https://dev.mysql.com/doc/refman/5.7/en/mysqld-optiontables.html

https://mariadb.com/kb/en/library/full-list-of-mariadb-optionssystem-and-status-variables/

## 1.2. 注意：
其中有些参数支持运行时修改，会立即生效；有些参数不支持，且只能通过修改配置文件，并重启服务器程序生效。  
有些参数作用域是全局的，且不可改变；有些可以为每个用户提供单独（会话）的设置。

# 2. 管理
## 2.1. 服务器选项
获取mysqld的可用选项列表：
```
mysqld --help --verbose
mysqladmin variables
mysqld --print-defaults     ##获取默认设置
```

设置服务器选项方法：
```
## 在命令行中设置
shell> ./mysqld_safe --skip-name-resolve=1

## 在配置文件my.cnf中设置
skip_name_resolve=1
```

## 2.2. 服务器系统变量
全局和会话两种类型。

获取系统变量
```
mysql> SHOW GLOBAL VARIABLES;
mysql> SHOW [SESSION] VARIABLES;
mysql> SELECT @@VARIABLES;
```

修改服务器变量的值：
```
mysql> help SET
```

```
## 修改全局变量：仅对修改后新创建的会话有效；对已经建立的会话无效。
mysql> SET GLOBAL system_var_name=value;
mysql> SET @@global.system_var_name=value;


## 修改会话变量
mysql> SET [SESSION] system_var_name=value;
mysql> SET @@[session.]system_var_name=value;
```

## 2.3. 服务器状态变量
全局和会话两种

状态变量（只读）：用于保存mysqld运行中的统计数据的变量，不可更改。

获取服务器状态变量
```
mysql> SHOW GLOBAL STATUS;
mysql> SHOW [SESSION] STATUS;
```

# 3. 服务器变量 SQL_MODE

SQL_MODE：对其设置可以完成一些约束检查的工作，可分别进行全局的设置或当前会话的设置。

参看：https://mariadb.com/kb/en/library/sql-mode/

常见MODE:

|常见MODE|描述|
|:-|:-|
| NO_AUTO_CREATE_USER  |禁止GRANT创建密码为空的用户。|
| NO_ZERO_DATE  |在严格模式，不允许使用‘0000-00-00’ 的时间。|
| ONLY_FULL_GROUP_BY  |对于GROUP BY聚合操作，如果在SELECT中的列，没有在GROUP BY中出现，那么将认为这个SQL是不合法的。|
| NO_BACKSLASH_ESCAPES  |反斜杠“\” 作为普通字符而非转义字符。|
| PIPES_AS_CONCAT  |将"\|\|"视为连接操作符而非“或运算符”|

# 1. mysql字符集

数据库、表、字段

# 2. 持久改变

```
## 服务器
## 需要重启服务
vim /mysql/3306/etc/my.cnf
[mysqld]
character-set-server=utf8mb4

service mysqld3306 restart
systemctl restart mariadb


## 客户端
## 即时生效
vim /etc/my.cnf
[client]
default-character-set=utf8mb4
```

# 3. 命令更改字符集

```
set character_set_server=utf8;
set character_set_database=utf8;
set character_set_client=utf8;
set character_set_connection=utf8;
```

```
alter database studentdb character set  utf8;
alter table student character set utf8;
alter table student modify name varchar(20) character set utf8;
```
# 1. 数据库

# 2. 数据库操作

创建数据库：
```
CREATE DATABASE|SCHEMA [IF NOT EXISTS] 'DB_NAME'
    CHARACTER SET 'character set name'
    COLLATE 'collate name'
```
```
show create database db1;
```

删除数据库
```
DROP DATABASE|SCHEMA [IF EXISTS] 'DB_NAME';
```

查看支持所有字符集：
```
SHOW CHARACTER SET;

# cat db.opt 
default-character-set=utf8mb4
default-collation=utf8mb4_general_ci
```

查看支持所有排序规则：
```
SHOW COLLATION;
```

获取命令使用帮助：
```
mysql> HELP KEYWORD;
```

查看数据库列表：
```
mysql> SHOW DATABASES;
```
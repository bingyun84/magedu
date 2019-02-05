# 1. 表
表：二维关系
- 设计表：遵循规范，第三范式。
- 定义：字段，索引  
字段：字段名，字段数据类型，修饰符  
约束，索引：应该创建在经常用作查询条件的字段上。

# 2. 创建表：
```
CREATE TABLE
```
(1) 直接创建
```
CREATE TABLE [IF NOT EXISTS] 'tbl_name' (col1 type1 修饰符, col2
type2 修饰符, ...)
```

(2) 通过查询现存表创建；新表会被直接插入查询而来的数据。  
```
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name
    [(create_definition,...)] [table_options] [partition_options]
    select_statement
```

(3) 通过复制现存的表的表结构创建，但不复制数据。   
```
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name 
    { LIKE old_tbl_name | (LIKE old_tbl_name) }
```
## 2.1. 注意：

- Storage Engine 是指表类型，也即在表创建时指明其使用的存储引擎，同一库中不同表可以使用不同的存储引擎。
- 同一个库中表建议要使用同一种存储引擎类型。

## 2.2. 字段信息
col type1  
- PRIMARY KEY(col1,...)  
- INDEX(col1, ...)  
- UNIQUE KEY(col1, ...)  

## 2.3. 表选项：
- ENGINE [=] engine_name  
SHOW ENGINES    #查看支持的engine类型  
- ROW_FORMAT [=] {DEFAULT|DYNAMIC|FIXED|COMPRESSED|REDUNDANT|COMPACT}

## 2.4. 修饰符
针对所有类型：

|修饰符|描述|
|:-|:-|
| NULL |数据列可包含NULL值|
| NOT NULL |数据列不允许包含NULL值|
| DEFAULT |默认值|
| PRIMARY KEY |主键|
| UNIQUE KEY |唯一键|
| CHARACTER SET name |指定一个字符集|

针对数值型

|修饰符|描述|
|:-|:-|
| AUTO_INCREMENT |自动递增，适用于整数类型|
| UNSIGNED |无符号|

## 2.5. 示例
```
CREATE TABLE students (id int UNSIGNED NOT NULL PRIMARY KEY,name VARCHAR（20）NOT NULL,age tinyint UNSIGNED);

DESC students;

CREATE TABLE students2 (id int UNSIGNED NOT NULL ,name VARCHAR(20) NOT NULL,age tinyint UNSIGNED,PRIMARY KEY(id,name));
```

# 3. 获取帮助：
```
mysql> HELP CREATE TABLE;
```
# 4. 表操作
```
## 查看所有的引擎：
SHOW ENGINES


## 查看表：
SHOW TABLES [FROM db_name]


## 查看表结构：
DESC [db_name.]tb_name
SHOW COLUMNS FROM [db_name.]tb_name


## 删除表：
DROP TABLE [IF EXISTS] tb_name


## 查看表创建命令：
SHOW CREATE TABLE tbl_name


## 查看表状态：
SHOW TABLE STATUS LIKE 'tbl_name’


## 查看库中所有表状态：
SHOW TABLE STATUS FROM db_name
```

示例
```
mysql> desc student;
mysql> select columns from student; 	# 同上
show create table student
| stutdents | CREATE TABLE `students` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(10) NOT NULL,
  `sex` enum('f','m') DEFAULT 'm',
  `age` tinyint(3) unsigned DEFAULT NULL,
  `phone` char(11) DEFAULT NULL,
  `address` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 |


// \G 竖着看
mysql> show table status like 'student'\G
*************************** 1. row ***************************
           Name: students
         Engine: InnoDB
        Version: 10
     Row_format: Dynamic
           Rows: 0
 Avg_row_length: 0
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: 1
    Create_time: 2018-11-29 01:16:14
    Update_time: NULL
     Check_time: NULL
      Collation: utf8mb4_general_ci
       Checksum: NULL
 Create_options: 
        Comment: 
1 row in set (0.00 sec)

ERROR: No query specified
```

# 5. 表组件
```
ALTER TABLE 'tbl_name'
```

## 5.1. 字段  
```
## 添加字段：add
ADD col1 data_type [FIRST|AFTER col_name]


## 删除字段：drop   


## 修改字段：   
## alter（默认值）, change（字段名）, modify（字段属性）   
```

## 5.2. 索引:
```
## 添加索引：add index  
## 删除索引：drop index  


## 查看表上的索引：
SHOW INDEXES FROM [db_name.]tbl_name;
```
## 5.3. 查看帮助：
```
Help ALTER TABLE
```
## 5.4. 修改表示例
```
ALTER TABLE students RENAME s1;
ALTER TABLE s1 ADD phone varchar(11) AFTER name;
ALTER TABLE s1 MODIFY phone int;
ALTER TABLE s1 CHANGE COLUMN phone mobile char(11);
ALTER TABLE s1 DROP COLUMN mobile;
Help ALTER TABLE    ## 查看帮助修改表示例
ALTER TABLE students ADD gender ENUM('m','f')
ALETR TABLE students CHANGE id sid int UNSIGNED NOT NULL PRIMARY KEY;
ALTER TABLE students ADD UNIQUE KEY(name);
ALTER TABLE students ADD INDEX(age);
DESC students;
SHOW INDEXES FROM students;
ALTER TABLE students DROP age;
```
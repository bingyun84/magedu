# 1. 触发器

触发器的执行不是由程序调用，也不是由手工启动，而是由事件来触发、激活从而实现执行。

# 2. 语法
## 2.1. 创建触发器

```
CREATE
    [DEFINER = { user | CURRENT_USER }]
TRIGGER trigger_name
    trigger_time trigger_event
ON tbl_name FOR EACH ROW
    trigger_body
```
说明：

|元素|描述|
|:-|:-|
| trigger_name|触发器的名称。|
| trigger_time|{ BEFORE \| AFTER }，表示在事件之前或之后触发。|
| trigger_event|{ INSERT \|UPDATE \| DELETE }，触发的具体事件。|
| tbl_name|该触发器作用在表名。|

## 2.2. 删除触发器

```
DROP TRIGGER trigger_name;
```

# 3. 查看触发器

```
SHOW TRIGGERS
```

查询系统表 information_schema.triggers 的方式。指定查询条件，查看指定的触发器信息。
```
mysql> USE information_schema;
Database changed
mysql> SELECT * FROM triggers WHERE
trigger_name='trigger_student_count_insert';
```

# 4. 示例

创建触发器，在向学生表INSERT数据时，学生数增加，DELETE学生时，学生数减少。
```
## 准备表
CREATE TABLE student_info (
    stu_id INT(11) NOT NULL AUTO_INCREMENT,
    stu_name VARCHAR(255) DEFAULT NULL,
    PRIMARY KEY (stu_id)
);
CREATE TABLE student_count (
    student_count INT(11) DEFAULT 0
);
INSERT INTO student_count VALUES(0);


## 创建触发器
CREATE TRIGGER trigger_student_count_insert
AFTER INSERT
ON student_info FOR EACH ROW
UPDATE student_count SET student_count=student_count+1;

CREATE TRIGGER trigger_student_count_delete
AFTER DELETE
ON student_info FOR EACH ROW
UPDATE student_count SET student_count=student_count-1;
```
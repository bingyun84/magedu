# 1. 函数

函数：系统函数和自定义函数

# 2. 系统函数

https://dev.mysql.com/doc/refman/5.7/en/func-op-summaryref.html

# 3. 自定义函数 (user-defined function UDF)

保存在mysql.proc表中。

## 3.1. 语法

创建UDF
```
CREATE [AGGREGATE] FUNCTION function_name(parameter_name type,[parameter_name type,...]) RETURNS {STRING|INTEGER|REAL}
    runtime_body
```

说明：参数和返回值个数

||个数|
|:-|:-|
|参数|不限|
|返回值|1个|

创建无参UDF
```
CREATE FUNCTION simpleFun() RETURNS VARCHAR(20) RETURN "Hello World!”;
```
创建有参数UDF
```
DELIMITER //
CREATE FUNCTION deleteById(uid SMALLINT UNSIGNED) RETURNS VARCHAR(20)
BEGIN
    DELETE FROM students WHERE stuid = uid;
    RETURN (SELECT COUNT(stuid) FROM students);
END//
DELIMITER ;
```

## 3.2. 管理函数

查看函数列表：
```
SHOW FUNCTION STATUS;
```

查看函数定义
```
SHOW CREATE FUNCTION function_name
```

删除UDF
```
DROP FUNCTION function_name
```

调用自定义函数语法
```
SELECT function_name(parameter_value,...)
```

# 4. 局部变量
## 4.1. 自定义函数中定义局部变量语法

```
DECLARE 变量1[,变量2,... ]变量类型 [DEFAULT 默认值]
```
说明：局部变量的作用范围是在 BEGIN...END 程序中，而且定义局部变量语句必须在 BEGIN...END 的第一行定义。

示例：
```
DELIMITER //
CREATE FUNCTION addTwoNumber(x SMALLINT UNSIGNED, Y SMALLINT UNSIGNED) RETURNS SMALLINT
BEGIN
    DECLARE a, b SMALLINT UNSIGNED;
    SET a = x, b = y;
    RETURN a+b;
END//
DELIMITER ;
```
```
set @a=10;
select @a;
```

## 4.2. 为变量赋值语法
```
SET parameter_name = value[,parameter_name = value...]
```
```
SELECT INTO parameter_name
```

示例:
```
...
DECLARE x int;
SELECT COUNT(id) FROM tdb_name INTO x;
RETURN x;
END//
```
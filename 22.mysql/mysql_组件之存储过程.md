# 1. 存储过程
存储过程优势
- 存储过程把经常使用的SQL语句或业务逻辑封装起来，预编译保存在数据库中，当需要时从数据库中直接调用，省去了编译的过程。
- 提高了运行速度。
- 同时降低网络数据传输量。

存储过程保存在mysql.proc表中。

# 2. 语法
## 2.1. 创建存储过程
```
CREATE PROCEDURE sp_name ([ proc_parameter [,proc_parameter ...]])
    routime_body
```

### 2.1.1. proc_parameter 的构成

```
[IN|OUT|INOUT] parameter_name type
```

||描述|
|:-|:-|
|IN|输入参数|
|OUT|输出参数|
|INOUT|输入输出参数|
|param_name|参数名称|
|type|参数类型|

## 2.2. 存储过程修改  

ALTER语句修改存储过程只能修改存储过程的注释等无关紧要的东西，不能修改存储过程体，所以要修改存储过程，方法就是删除重建。

## 2.3. 删除存储过程

```
DROP PROCEDURE [IF EXISTS] sp_name
```

# 3. 查看及调用

查看存储过程列表
```
SHOW PROCEDURE STATUS;
```

查看存储过程定义
```
SHOW CREATE PROCEDURE sp_name
```

调用存储过程
```
CALL sp_name ([ proc_parameter [,proc_parameter ...]])
```
```
CALL sp_name
```
说明:当无参时,可以省略"()",当有参数时,不可省略"()”


# 4. 存储过程示例   

创建无参存储过程
```
delimiter //
CREATE PROCEDURE showTime()
BEGIN
    SELECT now();
END//
delimiter ;


CALL showTime;
```

创建含参存储过程：只有一个IN参数
```
delimiter //
CREATE PROCEDURE selectById(IN uid SMALLINT UNSIGNED)
BEGIN
    SELECT * FROM students WHERE stuid = uid;
END//
delimiter ;


call selectById(2);
```

示例
```
delimiter //
CREATE PROCEDURE dorepeat(n INT)
BEGIN
    SET @i = 0;
    SET @sum = 0;
    REPEAT SET @sum = @sum+@i; SET @i = @i + 1;
    UNTIL @i > n END REPEAT;
END//
delimiter ;


CALL dorepeat(100);
SELECT @sum;
```

创建含参存储过程：包含IN参数和OUT参数
```
delimiter //
CREATE PROCEDURE deleteById(IN uid SMALLINT UNSIGNED, OUT num SMALLINT UNSIGNED)
BEGIN
    DELETE FROM students WHERE stuid = uid;
    SELECT row_count() into num;
END//
delimiter ;


call deleteById(2,@Line);
SELECT @Line;
```
说明：创建存储过程deleteById，包含一个IN参数和一个OUT参数。调用时，传入删除的ID和保存被修改的行数值的用户变量@Line，select @Line;输出被影响行数。

# 5. 流程控制
存储过程和函数中可以使用流程控制来控制语句的执行。

|流程控制语句|描述|
|:-|:-|
| IF|用来进行条件判断。根据是否满足条件，执行不同语句。|
| CASE|用来进行条件判断，可实现比IF语句更复杂的条件判断。|
| LOOP|重复执行特定的语句，实现一个简单的循环。|
| LEAVE|用于跳出循环控制。|
| ITERATE|跳出本次循环，然后直接进入下一次循环。|
| REPEAT|有条件控制的循环语句。当满足特定条件时，就会跳出循环语句。|
| WHILE|有条件控制的循环语句。|
# 1. DQL语句
# 2. select
## 2.1. 语法
```
SELECT
    [ALL | DISTINCT | DISTINCTROW ]
    [SQL_CACHE | SQL_NO_CACHE]
    select_expr [, select_expr ...]
[FROM table_references
[WHERE where_condition]
[GROUP BY {col_name | expr | position}
    [ASC | DESC], ... [WITH ROLLUP]]
[HAVING where_condition]
[ORDER BY {col_name | expr | position}
    [ASC | DESC], ...]
[LIMIT {[offset,] row_count | row_count OFFSET offset}]
[FOR UPDATE | LOCK IN SHARE MODE]
```

## 2.2. 字段显示可以使用别名：
```
col1 AS alias1, col2 AS alias2, ...
```

## 2.3. DISTINCT 
去除重复列
```
SELECT DISTINCT gender FROM students;
```

## 2.4. WHERE子句：
指明过滤条件以实现“选择”的功能：
### 2.4.1. 过滤条件：
布尔型表达式
### 2.4.2. 算术操作符：
+，-，*，/，%
### 2.4.3. 比较操作符：  
- =，<=>（相等或都为空），<>，!=(非标准SQL)，>，>=，<，<=  
- BETWEEN min_num AND max_num  
- IN (element1, element2, ...)  
- IS NULL  
- IS NOT NULL  

### 2.4.4. LIKE:

|通配符|描述|
|:-|:-|
| % |任意长度的任意字符|
| _ |任意单个字符|

### 2.4.5. RLIKE：
正则表达式，索引失效，不建议使用。

### 2.4.6. REGEXP：
匹配字符串可用正则表达式书写模式，同上。

### 2.4.7. 逻辑操作符：
- NOT
- AND
- OR
- XOR

## 2.5. GROUP
根据指定的条件把查询结果进行“分组”以用于做“聚合”运算。
avg(), max(), min(), count(), sum()

select 只能出现分组字段和聚合函数。

## 2.6. HAVING
对分组聚合运算后的结果指定过滤条件。

## 2.7. ORDER BY
根据指定的字段对查询结果进行排序。
- 升序：ASC
- 降序：DESC

加减号“-”，null就排到后面去了
```
order by -classid desc;		
```

## 2.8. LIMIT
对查询的结果进行输出行数数量限制。
```
LIMIT [[offset,]row_count]
```


## 2.9. FOR UPDATE
对查询结果中的数据请求施加“锁”。
写锁，独占或排它锁，只有一个读和写。

## 2.10. LOCK IN SHARE MODE: 
对查询结果中的数据请求施加“锁”。
读锁，共享锁，同时多个读。

# 3. select 执行顺序

|步骤|描述|
|:-|:-|
|from  |
|where 		|#过滤行  |
|group by 	|#分组  |
|having		|#分组过滤  |
|order by	|#排序  |
|select		|#选择列，投影  |
|limit		|#过滤行  |

# 4. 示例
```
DESC students;
INSERT INTO students VALUES(1,'tom'，'m'),(2,'alice','f');
INSERT INTO students(id,name) VALUES(3,'jack'),(4,'allen');
SELECT * FROM students WHERE id < 3;
SELECT * FROM students WHERE gender='m';
SELECT * FROM students WHERE gender IS NULL;
SELECT * FROM students WHERE gender IS NOT NULL;
SELECT * FROM students ORDER BY name DESC LIMIT 2;
SELECT * FROM students ORDER BY name DESC LIMIT 1,2;
SELECT * FROM students WHERE id >=2 and id <=4
SELECT * FROM students WHERE BETWEEN 2 AND 4
SELECT * FROM students WHERE name LIKE ‘t%’
SELECT * FROM students WHERE name RLIKE '.*[lo].*';
SELECT id stuid,name as stuname FROM students
```

# 5. SQL JOINS多表查询
## 5.1. 交叉连接：
笛卡尔乘积，每一行都与另外表的所有行连接，结果=M*N。
```
TABLE1 cross join TABLE2
```

## 5.2. 内连接：
- 等值连接：让表之间的字段以“等值”建立连接关系；
- 不等值连接
- 自然连接：去掉重复列的等值连接
- 自连接

## 5.3. 外连接：
### 5.3.1. 左外连接：
```
FROM tb1 LEFT JOIN tb2 ON tb1.col=tb2.col
```
```
## 标准用法
select <select_list>
from a
    left outer join b
    on a.key = b.key

## 变种
select <select_list>
from a
    left outer join b
    on a.key = b.key
where b.key is null
```

关键点：b.key is null，过滤掉了a、b内连接部分。

### 5.3.2. 右外连接
```
FROM tb1 RIGHT JOIN tb2 ON tb1.col=tb2.col
```
```
## 标准用法
select <select_list>
from a
    right outer join b
    on a.key = b.key

## 变种
select <select_list>
from a
    right outer join b
    on a.key = b.key
where a.key is null
```
关键点：a.key is null，过滤掉了a、b内连接部分。

### 5.3.3. full outer join
mysql目前不支持
```
left outer join
union
right outer join
```

# 6. 子查询：
在查询语句嵌套着查询语句，性能较差。
基于某语句的查询结果再次进行的查询。

## 6.1. 用在 WHERE 子句中的子查询
用于比较表达式中的子查询；子查询仅能返回单个值。
```
SELECT Name
    ,Age 
FROM students 
WHERE Age > (SELECT avg(Age) FROM students);
```

### 6.1.1. 用于 IN 中的子查询：
子查询应该单键查询并返回一个或多个值从构成列表。
```
SELECT Name
    ,Age 
FROM students 
WHERE Age IN (SELECT Age FROM teachers);
```
### 6.1.2. 用于 EXISTS

## 6.2. 用于 FROM 子句中的子查询
```
SELECT tb_alias.col1,... 
FROM (SELECT clause) AS tb_alias 
WHERE Clause;
```
示例：
```
SELECT s.aage,s.ClassID 
FROM (SELECT avg(Age) AS aage,ClassID
        FROM students 
        WHERE ClassID IS NOT NULL GROUP BY ClassID
    ) AS s
WHERE s.aage>30;
```

# 7. 联合查询：UNION
```
SELECT Name,Age FROM students 
UNION 
SELECT Name,Age FROM teachers;
```

# 8. 练习
导入 hellodb.sql 生成数据库。
1) 在 students 表中，查询年龄大于25岁，且为男性的同学的名字和年龄
2) 以 ClassID 为分组依据，显示每组的平均年龄
3) 显示第2题中平均年龄大于30的分组及平均年龄
4) 显示以 L 开头的名字的同学的信息
5) 显示 TeacherID 非空的同学的相关信息
6) 以年龄排序后，显示年龄最大的前10位同学的信息
7) 查询年龄大于等于20岁，小于等于25岁的同学的信息

导入 hellodb.sql，实现下面要求
1. 以 ClassID 分组，显示每班的同学的人数
1. 以 Gender 分组，显示其年龄之和
1. 以 ClassID 分组，显示其平均年龄大于25的班级
1. 以 Gender 分组，显示各组中年龄大于25的学员的年龄之和
1. 显示前5位同学的姓名、课程及成绩
1. 显示其成绩高于80的同学的名称及课程
1. 取每位同学各门课的平均成绩，显示成绩前三名的同学的姓名和平均成绩
1. 显示每门课程课程名称及学习了这门课的同学的个数
1. 显示其年龄大于平均年龄的同学的名字
1. 显示其学习的课程为第1、2，4或第7门课的同学的名字
1. 显示其成员数最少为3个的班级的同学中年龄大于同班同学平均年龄的同学
1. 统计各班级中年龄大于全校同学平均年龄的同学
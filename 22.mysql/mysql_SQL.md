# 1. SQL语言
# 2. SQL语言的兴起
- 20世纪70年代，IBM开发出SQL，用于DB2
- 1981年，IBM推出SQL/DS数据库
- 业内标准微软和Sybase的T-SQL，Oracle的PL/SQL
- SQL作为关系型数据库所使用的标准语言，最初是基于IBM的实现在1986年被批准的。 1987年，“国际标准化组织(ISO)” 把ANSI(美国国家标准化组织)SQL作为国际标准。

# 3. SQL：ANSI SQL
SQL-1986, SQL-1989, SQL-1992, SQL-1999, SQL-2003，SQL-2008, SQL-2011

# 4. SQL语言规范
- 在数据库系统中，SQL语句不区分大小写(建议用大写)
- SQL语句可单行或多行书写，以“;” 结尾
- 关键词不能跨多行或简写
- 用空格和缩进来提高语句的可读性
- 子句通常位于独立行，便于编辑，提高可读性
- 注释：  
SQL标准：  
--多行注释：/\*注释内容\*/   
--单行注释：-- 注释内容 注意有空格  
MySQL注释：  
--\# 

# 5. SQL语句分类：
- DDL: Data Defination Language 数据定义语言  
CREATE，DROP，ALTER  
- DML: Data Manipulation Language 数据操纵语言  
INSERT，DELETE，UPDATE  
- DCL：Data Control Language 数据控制语言  
GRANT，REVOKE，COMMIT，ROLLBACK  
- DQL：Data Query Language 数据查询语言  
SELECT

# 6. SQL语句构成
- Keyword组成clause
- 多条clause组成语句

示例：
```
SELECT *            ## SELECT子句
FROM products       ## FROM子句
WHERE price>400     ## WHERE子句
```
说明：一组SQL语句，由三个子句构成。SELECT、FROM 和 WHERE 是关键字

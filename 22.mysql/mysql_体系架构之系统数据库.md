# 1. MySQL中的系统数据库
## 1.1. mysql数据库
mysql数据库是mysql的核心数据库，类似于Sql Server中的master库，主要负责存储数据库的用户、权限设置、关键字等mysql自己需要使用的控制和管理信息。

## 1.2. performance_schema数据库
MySQL 5.5开始新增的数据库，主要用于收集数据库服务器性能参数,库里表的存储引擎均为PERFORMANCE_SCHEMA，用户不能创建存储引擎为PERFORMANCE_SCHEMA的表。

## 1.3. information_schema数据库
MySQL 5.0之后产生的，一个虚拟数据库，物理上并不存在。information_schema数据库类似与“数据字典”，提供了访问数据库元数据的方式，即数据的数据。比如数据库名或表名，列类型，访问权限（更加细化的访问方式）。
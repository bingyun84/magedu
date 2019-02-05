# 1. Mysql 客户端程序

# 2. mysql使用模式：
交互式模式，可运行命令有两类：客户端命令和服务器端命令。 

## 2.1. 客户端命令：  

```
mysql> help show
```

|命令|描述|
|:-|:-|
|\h|help|
|\u|use|
|\s|status|
|\\!|system|  

## 2.2. 服务器端命令：  
SQL语句， 需要语句结束符，默认为分号“;”  

# 3. mysql客户端可用选项：
|选项|描述|
|:-|:-|
|-A, --no-auto-rehash |禁止补全|
|-u, --user= |用户名，默认为root|
|-h, --host= |服务器主机，默认为localhost|
|-p, --passowrd= |用户密码，建议使用-p，默认为空密码|
|-P, --port= |服务器端口|
|-S, --socket= |指定连接socket文件路径|
|-D, --database= |指定默认数据库|
|-C, --compress |启用压缩|
|-e “SQL“ |执行SQL命令|
|-V, --version |显示版本|
|-v, --verbose |显示详细信息|
|--print-defaults |获取程序默认使用的配置|

# 4. 执行命令
运行 mysql 命令：默认空密码登录。

登录系统：
```
mysql -uroot -p
```

客户端命令：本地执行
```
mysql> help
```

每个命令都完整形式和简写格式
```
mysql> status 或 \s
```

使用 -e 选项执行命令
```
mysql -e 'show databases'
```

服务端命令：通过 mysql 协议发往服务器执行并取回结果。每个命令末尾都必须使用命令结束符号，默认为分号。
```
mysql> SELECT VERSION();
```

脚本模式：
```
mysql -uUSERNAME -pPASSWORD < /path/somefile.sql

mysql> source /path/from/somefile.sql
```

# 5. 示例

修改提示符
```
## 使用命令
mysql> prompt mysql-->
mysql> prompt \u@[\D]

## 使用选项
mysql --prompt="\u@[\D]"

## 配置文件
## /etc/my.cnf.d/mysql-clients.cnf
[mysql]
prompt="\u@[\D]"
```

```
## 显示当前数据库
mysql> select database();

## 切换数据库
mysql> use mysql

## 查看当前用户
mysql> select user();    
mysql> SELECT User,Host,Password FROM user;
```
# 1. 服务器端配置
服务器端(mysqld)：工作特性有多种配置方式
1. 命令行选项
1. 配置文件

# 2. 配置文件：
类ini格式。集中式的配置，能够为mysql的各应用程序提供配置信息。
```
[mysqld]
[mysqld_safe]
[mysqld_multi]
[mysql]
[mysqldump]
[server]
[client]
```
格式：parameter = value  
说明：_和- 相同  

|值|描述|
|:-|:-|
|1，ON，TRUE |真|
|0，OFF，FALSE |假|
省略掉值默认为真。

# 3. 配置文件顺序
后面覆盖前面的配置文件，顺序如下：

|配置文件|描述|
|:-|:-|
|/etc/my.cnf |Global选项|
|/etc/mysql/my.cnf| Global选项|
|SYSCONFDIR/my.cnf |Global选项|
|$MYSQL_HOME/my.cnf |Server-specific 选项|
|--defaults-extra-file=path||
|~/.my.cnf |User-specific 选项|


# 4. MariaDB配置
侦听3306/tcp端口可以在绑定有一个或全部接口IP上。
```
vim /etc/my.cnf
[mysqld]
skip-networking=1
```
关闭网络连接，只侦听本地客户端， 所有和服务器的交互都通过一个socket实
现，socket的配置存放在/var/lib/mysql/mysql.sock，可在/etc/my.cnf修改。

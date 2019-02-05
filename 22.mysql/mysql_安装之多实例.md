# 1. 杂章
数据库程序程序文件目录  
/app/mysql


# 2. 准备数据库文件目录
```
mkdir -pv /mysql/{3306,3307,3308}/{data,etc,socket,log.pid}

chown -R mysql.mysql /mysql/
```
目录结构
```
[root@mysql ~]# tree /mysql
/mysql
├── 3306
│   ├── data
│   ├── etc
│   ├── log
│   ├── pid
│   └── socket
├── 3307
│   ├── data
│   ├── etc
│   ├── log
│   ├── pid
│   └── socket
└── 3308
    ├── data
    ├── etc
    ├── log
    ├── pid
    └── socket
```

# 3. 生成数据库文件
```
cd /app/mysql/
scripts/mysql_install_db --user=mysql --datadir=/mysql/3306/data
scripts/mysql_install_db --user=mysql --datadir=/mysql/3307/data
scripts/mysql_install_db --user=mysql --datadir=/mysql/3308/data
```

# 4. 配置文件
```
vim /mysql/3306/etc/my.cnf
[client]
port	= 3306
socket	= /mysql/3306/socket/mysql.sock


[mysqld]
port	= 3306
datadir	= /mysql/3306/data/
socket	= /mysql/3306/socket/mysql.sock


[mysqld_safe]
log-error	= /mysql/3306/log/mariadb.log
pid-file	= /mysql/3306/pid/mariadb.pid
```
```
## cp 到3307 3308
cp my.cnf ../../3307/etc/
cp my.cnf ../../3308/etc/



## 修改内容
vim my.cnf
%s/3306/3308

## or

sed -i 's/3306/3307/' /mysql/3307/etc/my.cnf 
sed -i 's/3306/3308/' /mysql/3308/etc/my.cnf 
```



# 5. 启动脚本
脚本内容
```
vim /mysql/3306/mysqld

#!/bin/bash
#chkconfig: 345 80 2
port=3306
mysql_user="root"
mysql_pwd="centos"
cmd_path="/app/mysql/bin"
mysql_basedir="/mysql"
mysql_sock="${mysql_basedir}/${port}/socket/mysql.sock"

function_start_mysql()
{
    if [ ! -e "$mysql_sock" ];then
      printf "Starting MySQL...\n"
      ${cmd_path}/mysqld_safe --defaults-file=${mysql_basedir}/${port}/etc/my.cnf  &> /dev/null  &
    else
      printf "MySQL is running...\n"
      exit
    fi
}


function_stop_mysql()
{
    if [ ! -e "$mysql_sock" ];then
       printf "MySQL is stopped...\n"
       exit
    else
       printf "Stoping MySQL...\n"
       ${cmd_path}/mysqladmin -u ${mysql_user} -p${mysql_pwd} -S ${mysql_sock} shutdown
   fi
}


function_restart_mysql()
{
    printf "Restarting MySQL...\n"
    function_stop_mysql
    sleep 2
    function_start_mysql
}

case $1 in
start)
    function_start_mysql
;;
stop)
    function_stop_mysql
;;
restart)
    function_restart_mysql
;;
*)
    printf "Usage: ${mysql_basedir}/${port}/bin/mysqld {start|stop|restart}\n"
esac
```

```
## 复制
cd /mysql/3306
cp mysqld ../3308
cp mysqld ../3307

## 修改端口
vim ../3307/mysqld
vim ../3308/mysqld
```
改文件属主、属组为 mysql
```
chown mysql.mysql mysqld 
chown mysql.mysql ../3307/mysqld 
chown mysql.mysql ../3308/mysqld 
```
增加执行权限
```
chmod a+x mysqld
chmod a+x ../3307/mysqld
chmod a+x ../3308/mysqld

## or

chmod 700 mysqld
chmod 700 ../3307/mysqld 
chmod 700 ../3308/mysqld 
```
```
## 启动
./mysqld start

## 停止服务
./mysql stop
```

# 6. 注册服务
```
cp /mysql/3306/mysqld /etc/init.d/mysqld3306
cp /mysql/3307/mysqld /etc/init.d/mysqld3307
cp /mysql/3308/mysqld /etc/init.d/mysqld3308

chkconfig --add mysqld3306
chkconfig --add mysqld3307
chkconfig --add mysqld3308

service mysqld3306 start
service mysqld3307 start
service mysqld3308 start
```


# 7. 安全脚本，后面要加socket文件以示区别
安全加固
```
cd /app/mysql/bin
mysql_secure_installtion -S /mysql/3306/socket/mysql.sock
mysql_secure_installtion -S /mysql/3307/socket/mysql.sock
mysql_secure_installtion -S /mysql/3308/socket/mysql.sock
```
客户端连接
```
mysql -uroot -pcentos -h127.0.0.1 -P 3308 
mysql -uroot -pcentos -S /mysql/3308/socket/mysql.sock
```
看下结果
```
select @@port;
status
```
```
MariaDB [(none)]> select user,host,password,authentication_string from mysql.user;
+------+-----------+-------------------------------------------------------------------+
| user | host      | password                                  | authentication_string |
+------+-----------+-------------------------------------------+-----------------------+
| root | localhost | *128977E278358FF80A246B5046F51043A2B1FCED |                       |
| root | 127.0.0.1 | *128977E278358FF80A246B5046F51043A2B1FCED |                       |
| root | ::1       | *128977E278358FF80A246B5046F51043A2B1FCED |                       |
+------+-----------+-------------------------------------------+-----------------------+
3 rows in set (0.00 sec)
```



# 8. mysql_multi
只能同数据库软件版本多实例

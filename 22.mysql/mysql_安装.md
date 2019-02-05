# 1. Mariadb安装方式：
1. 源代码：编译安装。  
2. 二进制格式的程序包：展开至特定路径，并经过简单配置后即可使用。  
3. 程序包管理器管理的程序包。  
CentOS 安装光盘  
项目官方：https://downloads.mariadb.org/mariadb/repositories/  
国内镜像：https://mirrors.tuna.tsinghua.edu.cn/mariadb/mariadbx.y.z/yum/centos/7/x86_64/  

# 2. RPM包安装MySQL
## 2.1. centos 7：安装光盘直接提供  
mariadb-server 服务器包  
mariadb 客户端工具包  
mariadb-libs
```
yum install mariadb-server
```

centos 6
```
yum install mysql-server
```

## 2.2. 服务
```
systemctl start mariadb
systemctl enable mariadb
```

## 2.3. 提高安全性
```
/usr/bin/mysql_secure_installation

# 设置数据库管理员root口令
# 禁止root远程登录
# 删除anonymous用户帐号
# 删除test数据库
```

# 3. 通用二进制格式安装过程
## 3.1. 准备用户
```
groupadd -r -g 306 mysql
useradd -r -g mysql -u 306 -s /sbin/nologin -d /data/mysql -c "MariaDB User" mysql
```

## 3.2. 准备数据目录，建议使用逻辑卷
```
mkdir /data/mysql
chown mysql:mysql /data/mysql
```
或者
```
install -d /data/mysql -o mysql -g mysql
```
## 3.3. 准备二进制程序
```
tar xf mariadb-VERSION-linux-x86_64.tar.gz -C /usr/local
cd /usr/local
ln -sv mariadb-VERSION mysql
chown -R root:mysql /usr/local/mysql/
```
```
## 解压缩
tar xf mariadb-10.2.19-linux-x86_64.tar.gz -C /usr/local/

cd /usr/local/
ll mariadb-10.2.19-linux-x86_64
ln -s /usr/local/mariadb-10.2.19-linux-x86_64/ /usr/local/mysql

ls /usr/local/mysql
## 目录属性有问题，1021 1004
chown -R root.root /usr/local/mysql/
```

## 3.4. 准备配置文件
```
mkdir /etc/mysql/
cp support-files/my-large.cnf /etc/mysql/my.cnf
[mysqld]中添加三个选项：
datadir = /data/mysql
innodb_file_per_table = on          ## 默认值就是on了。
skip_name_resolve = on              ## 禁止主机名解析，建议使用
socket=/data/mysql/mysql.sock       ## 模板默认值/tmp/mysql.sock。修改后，下一步创建数据库文件会报错。


## 数据库启动后可以测试
show variables like 'innodb_file_per_table';
show @@innodb_file_per_table;
```
## 3.5. 创建数据库文件
```
cd /usr/local/mysql/
./scripts/mysql_install_db --datadir=/data/mysql --user=mysql
## 注意：必须此目录，此脚本要去找./bin下的文件：
```
## 3.6. 准备服务脚本，并启动服务
```
cp /usr/local/mysql/support-files/mysql.server /etc/rc.d/init.d/mysqld

chkconfig --add mysqld
service mysqld start
```
## 3.7. PATH路径
```
echo 'PATH=/usr/local/mysql/bin:$PATH' > /etc/profile.d/mysql.sh
. /etc/profile.d/mysql.sh

## 程序都可以用了
```
## 3.8. 安全初始化
```
/usr/local/mysql/bin/mysql_secure_installation

## 看下效果
mysql
select user,host,password,authentication_string from user;
```

# 4. 源码编译安装mariadb
## 4.1. 安装包
```
yum install bison bison-devel zlib-devel libcurl-devel libarchive-devel boostdevel gcc gcc-c++ cmake ncurses-devel gnutls-devel libxml2-devel openssldevel libevent-devel libaio-devel openssl-devel
```

## 4.2. 做准备用户和数据目录
```
useradd -r -s /sbin/nologin -d /data/mysql/ mysql
mkdir /data/mysql
chown mysql.mysql /data/mysql
tar xvf mariadb-10.2.18.tar.gz
```
## 4.3. cmake 编译安装
cmake 的重要特性之一是其独立于源码(out-of-source)的编译功能，即编译工作可以在另一个指定的目录中而非源码目录中进行，这可以保证源码目录不受任何一次编译的影响，因此在同一个源码树上可以进行多次不同的编译，如针对于不同平台编译。

编译选项：https://dev.mysql.com/doc/refman/5.7/en/source-configuration-options.html
```
cd mariadb-10.2.18/
cmake . \
-DCMAKE_INSTALL_PREFIX=/app/mysql \
-DMYSQL_DATADIR=/data/mysql/ \
-DSYSCONFDIR=/etc \
-DMYSQL_USER=mysql \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITHOUT_MROONGA_STORAGE_ENGINE=1 \
-DWITH_DEBUG=0 \
-DWITH_READLINE=1 \
-DWITH_SSL=system \
-DWITH_ZLIB=system \
-DWITH_LIBWRAP=0 \
-DENABLED_LOCAL_INFILE=1 \
-DMYSQL_UNIX_ADDR=/data/mysql/mysql.sock \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci


make && make install
```
提示：如果出错，执行
```
rm -f CMakeCache.txt
```

## 4.4. 准备环境变量
```
echo 'PATH=/app/mysql/bin:$PATH' > /etc/profile.d/mysql.sh
. /etc/profile.d/mysql.sh
```
## 4.5. 生成数据库文件
```
cd /app/mysql/
scripts/mysql_install_db --datadir=/data/mysql/ --user=mysql
```
## 4.6. 准备配置文件
```
cp /app/mysql/support-files/my-huge.cnf /etc/my.cnf

[mysqld]
datadir = /data/mysql
```
## 4.7. 准备启动脚本
```
cp /app/mysql/support-files/mysql.server /etc/init.d/mysqld
```
## 4.8. 启动服务
```
chkconfig --add mysqld
service mysqld start
```

## 4.9. 安全初始化
```
/usr/local/mysql/bin/mysql_secure_installation
```
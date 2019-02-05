# 1. Mysql 用户账号

## 1.1. 格式
mysql 用户账号由两部分组成：
'USERNAME'@'HOST'

## 1.2. HOST
HOST 限制此用户可通过哪些远程主机连接 mysql 服务器。IP地址或Network。
支持使用通配符：  

|通配符|描述|
|:-|:-|
|% |匹配任意长度的任意字符<br>172.16.0.0/255.255.0.0 或 172.16.%.%|
|_ |匹配任意单个字符|


# 2. 用户管理

## 2.1. 创建用户：CREATE USER

```
CREATE USER 'USERNAME'@'HOST' [IDENTIFIED BY 'password']；
```
```
create user test@'172.16.34.%' identified by 'centos';
```

默认权限：USAGE

## 2.2. 用户重命名：RENAME USER

```
RENAME USER old_user_name TO new_user_name;
```

## 2.3. 删除用户：

```
DROP USER 'USERNAME'@'HOST‘
```

示例：删除默认的空用户
```
DROP USER ''@'localhost';
```

## 2.4. 修改密码：

```
mysql> SET PASSWORD FOR 'user'@'host' = PASSWORD('password');
```
```
mysql> UPDATE mysql.user SET password=PASSWORD('password') WHERE clause;
mysql> FLUSH PRIVILEGES;    ## 此方法需要执行该指令才能生效
```
```
mysqladmin -u root -poldpass password 'newpass'
```

# 3. 忘记管理员密码的解决办法：

(1) 启动mysqld进程时，为其使用如下选项：    
```
--skip-grant-tables    
--skip-networking  
```
或者  
```
[mysqld]  
skip-grant-tables	#跳过授权文件  
skip-networking		#跳过网络，防止用户从网络访问  
```
(2) 使用 UPDATE 命令修改管理员密码。  
```
update mysql.user set password=password('centos') where userr='root';
```
(3) 关闭 mysqld 进程，移除上述两个选项，重启 mysqld。
# 1. MySQL读写分离

读写分离应用：
- mysql-proxy：Oracle，https://downloads.mysql.com/archives/proxy/
- Atlas：Qihoo，https://github.com/Qihoo360/Atlas/blob/master/README_ZH.md
- dbproxy：美团，https://github.com/Meituan-Dianping/DBProxy
- Cetus：网易乐得，https://github.com/Lede-Inc/cetus
- Amoeba：https://sourceforge.net/projects/amoeba/
- Cobar：阿里巴巴，Amoeba的升级版
- Mycat：基于Cobar， http://www.mycat.io/
- ProxySQL：https://proxysql.com/

# 2. ProxySQL

ProxySQL：MySQ L中间件，两个版本：官方版和 percona 版，percona 版是基于官方版基础上修改，C++语言开发，轻量级但性能优异(支持处理千亿级数据)，具有中间件所需的绝大多数功能，包括：
- 多种方式的的读/写分离
- 定制基于用户、基于schema、基于语句的规则对SQL语句进行路由
- 缓存查询结果
- 后端节点监控

官方站点：https://proxysql.com/  
官方手册：https://github.com/sysown/proxysql/wiki

## 2.1. ProxySQL安装

基于 YUM 仓库安装
```
cat <<EOF | tee /etc/yum.repos.d/proxysql.repo
[proxysql_repo]
name= ProxySQL YUM repository
baseurl=http://repo.proxysql.com/ProxySQL/proxysql-1.4.x/centos/\$releasever
gpgcheck=1
gpgkey=http://repo.proxysql.com/ProxySQL/repo_pub_key
EOF
```

基于RPM下载安装：https://github.com/sysown/proxysql/releases

## 2.2. ProxySQL组成

- 服务脚本：/etc/init.d/proxysql
- 配置文件：/etc/proxysql.cnf
- 主程序：/usr/bin/proxysql

## 2.3. ProxySQL实现读写分离
### 2.3.1. 准备

实现读写分离前，先实现主从复制。注意：slave 节点需要设置 read_only=1。

### 2.3.2. 启动ProxySQL

```
service proxysql start
```
启动后会监听两个端口，默认为6032（ProxySQL 的管理端口）和6033（ProxySQL
 对外提供服务的端口）。

### 2.3.3. 连接ProxySQL

使用mysql客户端连接到 ProxySQL 的管理接口6032，默认管理员用户和密码都是admin。
```
mysql -uadmin -padmin -P6032 -h127.0.0.1
```
说明：在 main 和 monitor 数据库中的表，runtime_ 开头的是运行时的配置，不能修改，只能修改非 runtime_ 表，修改后必须执行 LOAD … TO RUNTIME 才能加载到 RUNTIME 生效，执行 save … to disk 将配置持久化保存到磁盘。

### 2.3.4. 向 ProxySQL 中添加 MySQL 节点

以下操作不需要 use main 也可成功。
```
MySQL [(none)]> select * from mysql_servers;

MySQL [(none)]> insert into mysql_servers(hostgroup_id, hostname, port, comment) values(10, '192.168.6.17', 3306, 'write group');
MySQL [(none)]> insert into mysql_servers(hostgroup_id, hostname, port, comment) values(10, '192.168.6.27', 3306, 'read group');

MySQL [(none)]> load mysql servers to runtime;
MySQL [(none)]> save mysql servers to disk;
```

### 2.3.5. 添加监控后端节点的用户
因为 ProxySQL 需要通过每个节点的 read_only 值来自动调整它们是属于读组还是写组。

(1) 在 master 上执行：
```
mysql> grant replication client on *.* to monitor@'192.168.6.%' identified by 'centos';
```

(2) ProxySQL 上配置监控
```
set mysql-monitor_username='monitor';
set mysql-monitor_password='centos';
```
加载到 RUNTIME，并保存到 disk。
```
load mysql_variables to runtime;
save mysql_variables to disk;
```

### 2.3.6. monitor 库

监控模块的指标保存在 monitor 库的 log 表中。

查看监控连接是否正常的 (对connect指标的监控)：(如果 connect_error 的结果为 NULL 则表示正常)。
```
mysql> select * from mysql_server_connect_log;
```

查看监控心跳信息 (对ping指标的监控)：
```
mysql> select * from mysql_server_ping_log;
```

查看 read_only 和 replication_lag 的监控日志
```
mysql> select * from mysql_server_read_only_log;
mysql> select * from mysql_server_replication_lag_log;
```

### 2.3.7. 设置分组信息

需要修改的是 main 库中的 mysql_replication_hostgroups 表，该表有3个字段：writer_hostgroup，reader_hostgroup，comment。

指定写组的id为10，读组的id为20
```
insert into mysql_replication_hostgroups values(10, 20, "test");
```

将 mysql_replication_hostgroups 表的修改加载到 RUNTIME 生效
```
load mysql servers to runtime;
save mysql servers to disk;
```

Monitor 模块监控后端的 read_only 值，按照 read_only 的值将节点自动移动到读/写组。
```
select hostgroup_id, hostname, port, status, weight from mysql_servers;
+--------------+--------------+------+--------+--------+
| hostgroup_id | hostname | port | status | weight |
+--------------+--------------+------+--------+--------+
| 10 | 192.168.6.17 | 3306 | ONLINE | 1 |
| 20 | 192.168.6.27 | 3306 | ONLINE | 1 |
```

### 2.3.8. 配置发送SQL语句的用户

(1) 在master节点上执行
```
grant all on *.* to sqluser@'192.168.6.%' identified by 'centos';
```

(2) 在ProxySQL配置  
将用户 sqluser 添加到 mysql_users 表中，default_hostgroup 默认组设置为写组10，当读写分离的路由规则不符合时，会访问默认组的数据库。
```
insert into mysql_users(username, password, default_hostgroup) values ('sqluser', 'centos', 10);
load mysql users to runtime;
save mysql users to disk;
```

### 2.3.9. 使用 root 用户和 sqluser 用户测试是否能路由到默认的10写组实现读、写数据
```
mysql -usqluser -pcentos -P6033 -h127.0.0.1 -e 'select @@server_id'
mysql -usqluser -pcentos -P6033 -h127.0.0.1 -e 'create database testdb'
mysql -usqluser -pcentos -P6033 -h127.0.0.1 -e 'use testdb;create table t(id int)'
```

### 2.3.10. 配置路由规则，实现读写分离

与规则有关的表：mysql_query_rules 和 mysql_query_rules_fast_routing，后
者是前者的扩展表，1.4.7之后支持。

插入路由规则：  
将select语句分离到20的读组，select语句中有一个特殊语句 SELECT...FOR UPDATE 它会申请写锁，应路由到10的写组。
```
insert into mysql_query_rules \
(rule_id, active, match_digest, destination_hostgroup, apply) VALUES \
(1, 1, '^SELECT.*FOR UPDATE$', 10, 1), (2, 1, '^SELECT', 20, 1);

load mysql query rules to runtime;
save mysql query rules to disk;
```
注意：因 ProxySQL 根据 rule_id 顺序进行规则匹配，select ... for update 规则的 rule_id 必须要小于普通的 select 规则的 rule_id。

测试读操作是否路由给20的读组
```
mysql -usqluser -pcentos -P6033 -h127.0.0.1 -e 'select @@server_id'
```

测试写操作，以事务方式进行测试
```
mysql -usqluser -pcentos -P6033 -h127.0.0.1 \
-e 'start transaction;select @@server_id;commit;select @@server_id'
mysql -usqluser -pcentos -P6033 -h127.0.0.1 -e 'insert testdb.t values (1)'
mysql -usqluser -pcentos -P6033 -h127.0.0.1 -e 'select id from testdb.t'
```

路由的信息：  
查询stats库中的stats_mysql_query_digest表
```
SELECT hostgroup hg, sum_time, count_star, digest_text 
FROM stats_mysql_query_digest 
ORDER BY sum_time DESC;
```
# 1. MySQL高可用

MMM：Multi-Master Replication Manager for MySQL  
Mysql主主复制管理器是一套灵活的脚本程序，基于perl实现，用来对 mysql replication 进行监控和故障迁移，并能管理 mysql Master-Master 复制的配置（同一时间只有一个节点是可写的）。  
官网：http://www.mysql-mmm.org  
https://code.google.com/archive/p/mysql-master-master/downloads

MHA：Master High Availability  
对主节点进行监控，可实现自动故障转移至其它从节点；通过提升某一从节点为新的主节点，基于主从复制实现，还需要客户端配合实现，目前MHA主要支持一主多从的架构，要搭建MHA，要求一个复制集群中必须最少有三台数据库服务器，一主二从，即一台充当 master，一台充当备用 master，另外一台充当从库，出于机器成本的考虑，淘宝进行了改造，目前淘宝 TMHA 已经支持一主一从。  
官网：https://code.google.com/archive/p/mysql-master-ha/

Galera Cluster：wsrep（MySQL extended with the Write Set Replication）  
通过 wsrep 协议在全局实现复制；任何一节点都可读写，不需要主从复制，实现多主读写。

GR（Group Replication）
MySQL 官方提供的组复制技术（MySQL 5.7.17引入的技术），基于原生复制技术 Paxos 算法。

# 1.1. MHA


MHA集群架构  
MHA  
此处有图



## 1.2. MHA工作原理

1 从宕机崩溃的 master 保存二进制日志事件（binlog events）。  
2 识别含有最新更新的 slave。  
3 应用差异的中继日志（relay log）到其他的 slave。  
4 应用从 master 保存的二进制日志事件（binlog events）。  
5 提升一个 slave 为新的 master。  
6 使其他的 slave 连接新的 master 进行复制。

## 1.3. 软件

MHA 软件由两部分组成，Manager 工具包和 Node 工具包。

Manager 工具包主要包括以下几个工具：

|工具|描述|
|:-|:-|
|masterha_check_ssh |检查MHA的 SSH 配置状况|
|masterha_check_repl |检查 MySQL 复制状况|
|masterha_manger |启动 MHA|
|masterha_check_status |检测当前 MHA 运行状态|
|masterha_master_monitor |检测 master 是否宕机|
|masterha_master_switch |故障转移（自动或手动）|
|masterha_conf_host |添加或删除配置的 server 信息|

Node工具包：这些工具通常由MHA Manager的脚本触发，无需人为操作。主
要包括以下几个工具：

|工具|描述|
|:-|:-|
|save_binary_logs |保存和复制 master 的二进制日志|
|apply_diff_relay_logs |识别差异的中继日志事件并将其差异的事件应用于其他的 slave|
|filter_mysqlbinlog |去除不必要的 ROLLBACK 事件（MHA已不再使用此工具）|
|purge_relay_logs |清除中继日志（不会阻塞 SQL 线程）|

注意：为了尽可能的减少主库硬件损坏宕机造成的数据丢失，因此在配置 MHA 的同时建议配置成 MySQL 5.5的半同步复制 MHA。


自定义扩展：

|工具|描述|
|:-|:-|
|secondary_check_script |通过多条网络路由检测 master 的可用性|
|master_ip_ailover_script |更新 Application 使用的 masterip|
|shutdown_script |强制关闭 master 节点|
|report_script |发送报告|
|init_conf_load_script |加载初始配置参数|
|master_ip_online_change_script |更新 master 节点 ip地址|


配置文件：
- global配置，为各application提供默认配置
- application配置：为每个主从复制集群

## 1.4. 实现MHA

在管理节点上安装两个包：
- mha4mysql-manager
- mha4mysql-node

在被管理节点安装：
- mha4mysql-node

在管理节点建立配置文件
```
vim /etc/mastermha/app1.cnf
[server default]
user=mhauser
password=centos
manager_workdir=/data/mastermha/app1/
manager_log=/data/mastermha/app1/manager.log
remote_workdir=/data/mastermha/app1/
ssh_user=root
repl_user=repluser
repl_password=centos
ping_interval=1


[server1]
hostname=192.168.8.17
candidate_master=1

[server2]
hostname=192.168.8.27
candidate_master=1

[server3]
hostname=192.168.8.37实现MHA
```

实现Master
```
vim /etc/my.cnf
[mysqld]
log-bin
server_id=1
skip_name_resolve=1


mysql> show master logs
mysql> grant replication slave on *.* to repluser@'192.168.8.%' identified by 'centos';
mysql> grant all on *.* to mhauser@'192.168.8.%' identified by 'centos';
```

实现slave
```
vim /etc/my.cnf
[mysqld]
server_id=2     ## 不同节点此值各不相同
log-bin
read_only
relay_log_purge=0
skip_name_resolve=1


mysql> CHANGE MASTER TO 
MASTER_HOST='MASTER_IP',
MASTER_USER='repluser', 
MASTER_PASSWORD='centos',
MASTER_LOG_FILE='mariadb-bin.000001', 
MASTER_LOG_POS=245;
```

在所有节点实现相互之间 ssh key验证

MHA 验证
```
masterha_check_ssh --conf=/etc/mastermha/app1.cnf
masterha_check_repl --conf=/etc/mastermha/app1.cnf
```

MHA 启动
```
masterha_manager --conf=/etc/mastermha/app1.cnf
```

排错日志：
```
/data/mastermha/app1/manager.log
```
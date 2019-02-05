# 1. xtrabackup

Xtrabackup：percona 提供的 mysql 数据库备份工具，惟一开源的能够对 innodb 和 xtradb 数据库进行热备的工具。

手册：https://www.percona.com/doc/percona-xtrabackup/LATEST/index.html

Percona官网：www.percona.com
## 1.1. 特点：
- 备份还原过程快速、可靠
- 备份过程不会打断正在执行的事务
- 能够基于压缩等功能节约磁盘空间和流量
- 自动实现备份检验
- 开源，免费

## 1.2. 程序
Xtrabackup2.2版之前包括4个可执行文件
- innobackupex: Perl 脚本
- xtrabackup: C/C++ 编译的二进制
- xbcrypt: 加解密
- xbstream: 支持并发写的流文件格式

xtrabackup 是用来备份 InnoDB 表的，不能备份非 InnoDB 表。和 MySQL Server 没有交互。

innobackupex 脚本用来备份非 InnoDB 表，同时会调用 xtrabackup 命令来备份 InnoDB 表，还会和 MySQL Server 发送命令进行交互，如加全局读锁（FTWRL）、获取位点（SHOW SLAVE STATUS）等。即 innobackupex 是在 xtrabackup 之上做了一层封装实现的。

虽然目前一般不用 MyISAM 表，只是 MySQL 库下的系统表是 MyISAM 的，因此备份基本都通过 innobackupex 命令进行。


xtrabackup 备份过程  
此处有图


xtrabackup的新版变化
xtrabackup版本升级到2.4后，相比之前的2.1有了比较大的变化：
innobackupex 功能全部集成到 xtrabackup 里面，只有一个 binary程序。另外为了兼容考虑，innobackupex作为 xtrabackup 的软链接，即xtrabackup现在支持非Innodb表备份。并且 Innobackupex 在下一版本中移除，建议通过 xtrabackup 替换 innobackupex。

## 1.3. xtrabackup安装
```
yum install percona-xtrabackup 
```
在EPEL源中。

最新版本下载安装：
https://www.percona.com/downloads/XtraBackup/LATEST/

## 1.4. xtrabackup语法

备份：
```
innobackupex [option] BACKUP-ROOT-DIR
```

选项说明：  
https://www.percona.com/doc/percona-xtrabackup/LATEST/genindex.html

|选项|描述|
|:-|:-|
|--user|该选项表示备份账号。|
|--password|该选项表示备份的密码。|
|--host|该选项表示备份数据库的地址。|
|--databases|该选项接受的参数为数据库名，如果要指定多个数据库，彼此间需要以空格隔开；如："xtra_test dba_test"。<br>同时，在指定某数据库时，也可以只指定其中的某张表，如："mydatabase.mytable"。<br>该选项对innodb引擎表无效，还是会备份所有 innodb 表。|
|--defaults-file|该选项指定从哪个文件读取 MySQL 配置，必须放在命令行第一个选项位置。|
|--incremental|该选项表示创建一个增量备份，需要指定 --incremental-basedir。|
|--incremental-basedir|该选项指定为前一次全备份或增量备份的目录，与 --incremental 同时使用。|
|--incremental-dir|该选项表示还原时增量备份的目录。|
|--include=name|指定表名，格式：databasename.tablename。|

## 1.5. xtrabackup用法
### 1.5.1. Prepare
```
innobackupex --apply-log [option] BACKUP-DIR
```
选项说明：

|选项|描述|
|:-|:-|
|--apply-log|一般情况下，在备份完成后，数据尚且不能用于恢复操作，因为备份的数据中可能会包含尚未提交的事务或已经提交但尚未同步至数据文件中的事务。因此，此时数据文件仍处理不一致状态。<br>此选项作用是通过回滚未提交的事务及同步已经提交的事务至数据文件使数据文件处于一致性状态。|
|--use-memory|和 --apply-log 选项一起使用，当 prepare 备份时，做crash recovery分配的内存大小，单位字节，也可1MB,1M,1G,1GB等，推荐1G。|
|--export|表示开启可导出单独的表之后再导入其他 Mysql 中。|
|--redo-only|此选项在 prepare base full backup，往其中合并增量备份时候使用，但不包括对最后一个增量备份的合并。|

### 1.5.2. 还原
```
innobackupex --copy-back [选项] BACKUP-DIR
innobackupex --move-back [选项] [--defaults-group=GROUP-NAME] BACKUP-DIR
```
选项说明：

|选项|描述|
|:-|:-|
|--copy-back|做数据恢复时将备份数据文件拷贝到 MySQL 服务器的 datadir。|
|--move-back|这个选项与 --copy-back 相似，唯一的区别是它不拷贝文件，而是移动文件到目的地。这个选项移除 backup 文件，用时候必须小心。使用场景：没有足够的磁盘空间同事保留数据文件和 Backup 副本。|

还原注意事项：
1. datadir 目录必须为空。除非指定 innobackupex --force-non-emptydirectorires 选项指定，否则 --copy-backup 选项不会覆盖。
2. 在 restore 之前，必须 shutdown MySQL 实例，不能将一个运行中的实例 restore 到 datadir 目录中。
3. 由于文件属性会被保留，大部分情况下需要在启动实例之前将文件的属主改为 mysql，这些文件将属于创建备份的用户。chown -R mysql:mysql /data/mysql

以上需要在用户调用 innobackupex 之前完成。

--force-non-empty-directories：指定该参数时候，使得 innobackupex --copy-back 或 --move-back 选项转移文件到非空目录，已存在的文件不会被覆盖。如果 --copy-back 和 --move-back 文件需要从备份目录拷贝一个在 datadir 已经存在的文件，会报错失败。

备份生成的相关文件  
使用 innobakupex 备份时，其会调用 xtrabackup 备份所有的 InnoDB 表，复制所有关于表结构定义的相关文件(.frm)、以及 MyISAM、 MERGE、 CSV 和 ARCHIVE 表的相关文件，同时还会备份触发器和数据库配置信息相关的文件。这些文件会被保存至一个以时间命名的目录中，在备份时，innobackupex 还会在备份目录中创建如下文件：
1) xtrabackup_info  
innobackupex 工具执行时的相关信息，包括版本，备份选项，备份时长，备份 LSN(log sequence number日志序列号)，BINLOG 的位置。
2) xtrabackup_checkpoints  
备份类型（如完全或增量）、备份状态（如是否已经为 prepared 状态）和 LSN 范围信息，每个 InnoDB 页(通常为16k大小)都会包含一个日志序列号 LSN。LSN 是整个数据库系统的系统版本号，每个页面相关的 LSN 能够表明此页面最近是如何发生改变的。
3) xtrabackup_binlog_info  
MySQL 服务器当前正在使用的二进制日志文件及至备份这一刻为止二进制日志事件的位置，可利用实现基于 binlog 的恢复。
4) backup-my.cnf  
备份命令用到的配置选项信息。
5) xtrabackup_logfile  
备份生成的日志文件。

## 1.6. 示例：旧版xtrabackup完全备份及还原
1 在原主机
```
innobackupex --user=root /backups
scp -r /backups/2018-02-23_11-55-57/ 目标主机:/data/
```
2 在目标主机
```
innobackupex --apply-log /data/2018-02-23_11-55-57/
systemctl stop mariadb
rm -rf /var/lib/mysql/*
innobackupex --copy-back /data/2018-02-23_11-55-57/
chown -R mysql.mysql /var/lib/mysql/
systemctl start mariadb
```

## 1.7. 示例：新版xtrabackup完全备份及还原
1 在原主机做完全备份到/data/backups
```
xtrabackup --backup --target-dir=/backups/
scp -r /backups/* 目标主机:/backups
```
2 在目标主机上  
1）预准备：确保数据一致，提交完成的事务，回滚未完成的事务
```
xtrabackup --prepare --target-dir=/backups/
```
2）复制到数据库目录  
注意：数据库目录必须为空，MySQL 服务不能启动
```
xtrabackup --copy-back --target-dir=/backups/
```
3）还原属性
```
chown -R mysql:mysql /var/lib/mysql
```
4）启动服务
```
systemctl start mariadb
```

## 1.8. 示例：旧版xtrabackup完全，增量备份及还原
1 在原主机
```
innobackupex /backups
mkdir /backups/inc{1,2}

修改数据库内容

innobackupex --incremental /backups/inc1 --incrementalbasedir=/backups/2018-02-23_14-21-42  ##（完全备份生成的路径）

再次修改数据库内容

innobackupex --incremental /backups/inc2 --incrementalbasedir=/backups/inc1/2018-02-23_14-26-17 ##（上次增量备份生成的路径）
scp -r /backups/* 目标主机:/data/
```
2 在目标主机  
不启动mariadb
```
rm -rf /var/lib/mysql/*
innobackupex --apply-log --redo-only /data/2018-02-23_14-21-42/
innobackupex --apply-log --redo-only /data/2018-02-23_14-21-42/ --
incremental-dir=/data/inc1/2018-02-23_14-26-17
innobackupex --apply-log /data/2018-02-23_14-21-42/ --incrementaldir=/data/inc2/2018-02-23_14-28-29/
ls /var/lib/mysql/
innobackupex --copy-back /data/2018-02-23_14-21-42/
chown -R mysql.mysql /var/lib/mysql/
systemctl start mariadb
```

## 1.9. 示例：新版xtrabackup完全，增量备份及还原
1 备份过程  
1）完全备份：
```
xtrabackup --backup --target-dir=/backups/base
```
2）第一次修改数据  
3）第一次增量备份  
```
xtrabackup --backup --target-dir=/backups/inc1 --incrementalbasedir=/backups/base
```
4）第二次修改数据  
5）第二次增量  
```
xtrabackup --backup --target-dir=/backups/inc2 --incrementalbasedir=/backups/inc1
```
6）复制
```
scp -r /backups/* 目标主机:/backups/
```
备份过程生成三个备份目录  
```
/backups/{base，inc1，inc2}
```

2 还原过程  
1）预准备完成备份，此选项 --apply-log-only 阻止回滚未完成的事务
```
xtrabackup --prepare --apply-log-only --target-dir=/backups/base
```
2）合并第1次增量备份到完全备份，
```
xtrabackup --prepare --apply-log-only --target-dir=/backups/base --incremental-dir=/backups/inc1
```
3）合并第2次增量备份到完全备份：最后一次还原不需要加选项 --apply-log-only
```
xtrabackup --prepare --target-dir=/backups/base --incremental-dir=/backups/inc2
```
4）复制到数据库目录，注意数据库目录必须为空，MySQL 服务不能启动
```
xtrabackup --copy-back --target-dir=/data/backups/base
```
5）还原属性：
```
chown -R mysql:mysql /var/lib/mysql
```
6）启动服务：
```
systemctl start mariadb
```

## 1.10. 示例： xtrabackup单表导出和导入
1 单表备份
```
innobackupex --include='hellodb.students' /backups
```
2 备份表结构
```
mysql -e 'show create table hellodb.students' > student.sql
```
3 删除表
```
mysql -e 'drop table hellodb.students‘
```
4 
```
innobackupex --apply-log --export /backups/2018-02-23_15-03-23/
```
5 创建表
```
mysql>CREATE TABLE `students` (
`StuID` int(10) unsigned NOT NULL AUTO_INCREMENT,
`Name` varchar(50) NOT NULL,
`Age` tinyint(3) unsigned NOT NULL,
`Gender` enum('F','M') NOT NULL,
`ClassID` tinyint(3) unsigned DEFAULT NULL,
`TeacherID` int(10) unsigned DEFAULT NULL,
PRIMARY KEY (`StuID`)
) ENGINE=InnoDB AUTO_INCREMENT=26 DEFAULT CHARSET=utf8
```
6 删除表空间
```
alter table students discard tablespace;
```
7 复制
```
cp /backups/2018-02-23_15-03-23/hellodb/students.{cfg,exp,ibd} /var/lib/mysql/hellodb/
```
8 权限
```
chown -R mysql.mysql /var/lib/mysql/hellodb/
```
9 
```
mysql>alter table students import tablespace;
```
# 1. Myqsql socket 地址
服务器监听的两种socket地址。


|socket地址类型|描述|
|:-|:-|
|ip socket|监听在tcp的3306端口，支持远程通信。|
|unix sock|监听在sock文件上，仅支持本机通信。<br>如：/var/lib/mysql/mysql.sock|

说明：host 为 localhost、127.0.0.1时自动使用 unix sock。
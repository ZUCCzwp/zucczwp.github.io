# Mysql配置


### 修改最大连接数
#### 通过命令
1. 使用数据库提供 SQL语句
```
 set GLOBAL max_connections=100; 
 show variables like "max_connections";
```
及时生效，不需要重启服务，确保使用 root账号执行

#### 修改 my.cnf
```
vim /etc/my.cnf
max_connections=100
/etc/init.d/mysqld restart
```
需要重启服务

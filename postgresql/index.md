# Postgresql常用命令



### 查看 pg版本 ###
```
psql --version
```
***
### 获取当前db中所有的表信息 ###
```
select * from pg_tables;
//获取名为 public的 schema的所有表
select tablename from pg_tables where schemaname='public'
```
### 获取某个表结构信息 ###
```
\d authentik_core_user;
```
### 获取某个表数据 ###
```
SELECT * FROM authentik_core_user;
```
### 外网不能访问数据库 ###
::: 1、修改postgresql.conf文件:::<br>
在安装目录下data/postgresql.confi文件中将listen address改为*
```
listen_addresses = '*'
```
::: 2、修改pg_hba.conf文件:::<br>
在data/pg_hba.conf中设置
```
host all all 127.0.0.1/32 md5
host all all 0.0.0.0/0 md5
```
::: 3、重启 postgres服务:::<br>
```
systemctl restart postgresql
```
### 新建数据库 ###
```
CREATE DATABASE dbname;
```

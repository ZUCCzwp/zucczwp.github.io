# Mysql查询


## 查询顺序
1. 关键字顺序不能颠倒
```
select ...from...where...group by...having
```
2. select语句执行顺序
```
from > where > group by > having > select的字段 > dist
```

## 数据过滤
### 比较运算符
=、<>、<、<=、>、>=、!<、BETWEEN、IS NULL
### 逻辑运算符
AND、 OR、 IN、 NOT
### 通配符
如果要让索引生效，则 LIKE后面不能以%开头。
### 排序方式
FileSort：一般在内存中排序，占用 CPU较多。如果待排结果较大，会产生临时文件 I/O到磁盘进行排序，效率较低

Index：效率更高，索引可以保证数据的有序性

## SQL函数
### 算术函数
| 函数名     | 定义   |
|---------|------|
| ABS()   | 取绝对值 |
| MOD()   | 取余   |
| ROUND() | 四舍五入 |
### 字符串函数
| 函数名           | 定义                    |
|---------------|-----------------------|
| CONTACT()     | 多个字符串拼接               |
| LENGTH()      | 计算字段的长度，一个汉字算三个字符     |
| CHAR_LENGTH() | 计算字段的长度，汉字、数字、字母算一个字符 |
| LOWER()       | 将字符串的字符转化为小写          |
| UPPER()       | 将字符串的字符转化为大写          |
| REPLACE()     | 替换函数                  |
| SUBSTRING()   | 截取字符串                 |
### 日期函数
| 函数名                 | 定义         |
|---------------------|------------|
| CURRENT_DATE()      | 取绝对值       |
| CURRENT_TIME()      | 取余         |
| CURRENT_TIMESTAMP() | 四舍五入       |
| EXTRACT()           | 抽取具体的年、月、日 |
| DATE()              | 日期         |
| YEAR()              | 年份         |
| MONTH()             | 月份         |
| DAY()               | 天数         |
| HOUR()              | 时          |
| MINUTE()            | 分          |
| SECOND()            | 秒          |
### 转换函数
| 函数名        | 定义        |
|------------|-----------|
| CAST()     | 数据类型转化    |
| COALESCE() | 返回第一个非空数值 |
### 命名规范
1. 关键字和函数和名称全部大写
2. 数据库名、表名、字段名全部小写
3. SQL语句必须以分号结尾
### 聚集函数
| 函数名     | 定义  |
|---------|-----|
| COUNT() | 总行数 |
| MAX()   | 最大值 |
| MIN()   | 最小值 |
| SUM()   | 求和  |
| AVG()   | 平均值 |
### WHERE和 HAVING区别
过滤分组使用 HAVING.
数据行使用 WHERE
## 子查询
### 非关联子查询
子查询从数据表中查询了数据结果，如果这个数据结果只执行了一次，然后这个数据结果作为主查询的条件进行执行。
### Exist子查询

### 集合比较子查询
| | |
|----|---|
| IN | 判断是否在集合中 |
| ANY | 需跟比较操作符一起使用 |
| ALL | 需跟比较操作符一起使用 |
| SOME |  ANY的别名 |
### 关联子查询
如果子查询需要执行多次，即采用循环的方式，先从外部查询开始，每次都传入子查询进行查询，然后再将结果反馈给外部
### in和 exist效率
```
SELECT* FROM A WHERE cc IN(SELECT cc FROM B)
SELECT* FROM A WHERE cc EXIST(SELECT cc FROM B WHERE B.cc=A.cc)
```
如果对 cc列都建立了索引的情况下，需判断表 A和表 B的大小。

如果 A>B大，那么 IN子查询的效率要比 EXIST子查询效率高。
## 连接
### 自连接
### 笛卡尔积
### 外连接
### 非等值连接
### 等值连接

## 图
封装了底层和数据表的借口，简化一张表或者多张表的数据结果集
### 创建视图
```
create view view_name As
select column1, column2 from table;
```
### 修改视图
```
Alter view view_name As
select column1, column2 from table;
```
### 删除视图
```
drop view view_name
```

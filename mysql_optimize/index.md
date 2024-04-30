# Mysql查询优化


## 查询分页数据方法
1. 使用数据库提供 SQL语句
```
select * from table_name limit 0,5 

```
语句样式： MySQL中,可用如下方法: SELECT * FROM 表名称 LIMIT M,N

适应场景： 适用于数据量较少的情况(元组百/千级)

原因/缺点： 全表扫描，速度会很慢，且有的数据库结果集返回不稳定(如某次返回1,2,3,另外的一次返回2,1,3)，Limit限制的是从结果集的M位置处取出N条输出，其余抛弃。

2. 建立主键或唯一索引，利用索引

语句样式：可用如下方法：SELECT * FROM 表名称 WHERE pk_id > (pageNum*10) LIMIT M

适应场景：适用于数据量多的情况(元组数上万)

原因：索引扫描，速度会很快。有朋友提出：因为数据查询出来并不是按照pk_id排序的，所以会有漏掉数据的情况，只能方法3

3. 基于索引在排序

语句样式：可用如下方法： 
```
SELECT * FROM 表名称 WHERE pk_id > (pageNum*10) ORDER BY pk_id ASC LIMIT M
```

适应场景： 适用于数据量多的情况(元组数上万)，最好ORDER BY后的列对象是主键或唯一索引，使得ORDER BY操作能利用索引被消除但结果集是稳定的

原因：索引扫描，速度会很快。

4. 基于索引使用prepare（第一个问号表示pageNum，第二个？表示每页元组数）

语句样式：MySQL中,可用如下方法: 
```
PREPARE stmt_name FROM SELECT * FROM 表名称 WHERE pk_id > (？* ？) ORDER BY pk_id ASC LIMIT M
```

适应场景：大数据量

原因：索引扫描，速度会很快。prepare语句又比一般的查询语句快一点。

5. 利用MySQL支持ORDER操作可以利用索引快速定位部分元组，避免全表扫描

比如： 读第1000到1019行元组(pk是主键/唯一键).
```
SELECT * FROM your_table WHERE pk>=1000 ORDER BY pk ASC LIMIT 0,20
```
6. 利用"子查询/连接+索引"快速定位元组的位置，然后再读取元组。

道理同方法5。如(id是主键/唯一键，$page、$pagesize是变量):
利用子查询示例:
```
SELECT * FROM your_table WHERE id <= 
(SELECT id FROM your_table ORDER BY id desc LIMIT ($page-1)*$pagesize ORDER BY id desc LIMIT $pagesize
```
利用连接示例:
```
SELECT * FROM your_table AS t1 
JOIN (SELECT id FROM your_table ORDER BY id desc LIMIT ($page-1)*$pagesize AS t2 
WHERE t1.id <= t2.id ORDER BY t1.id desc LIMIT $pagesize;
```
### 优化方式
1. 直接用limit start, count分页语句：

2. 对limit分页问题的性能优化方法

利用表的覆盖索引来加速分页查询：我们都知道，利用了索引查询的语句中如果只包含了那个索引列（覆盖索引），那么这种情况会查询很快。

因为利用索引查找有优化算法，且数据就在查询索引上面，不用再去找相关的数据地址了，这样节省了很多时间。另外Mysql中也有相关的索引缓存，在并发高的时候利用缓存就效果更好了。

## 短连接风暴
短连接模型存在一个风险，就是一旦数据库处理得慢一些，连接数就会暴涨。max_connections 参数，用来控制一个 MySQL 实例同时存在的连接数的上限，超过这个值，系统就会拒绝接下来的连接请求，并报错提示“Too many connections”。对于被拒绝连接的请求来说，从业务角度看就是数据库不可用。

第一种方法：先处理掉那些占着连接但是不工作的线程。

因此，如果是连接数过多，你可以优先断开事务外空闲太久的连接；如果这样还不够，再考虑断开事务内空闲太久的连接。

从数据库端主动断开连接可能是有损的，尤其是有的应用端收到这个错误后，不重新连接，而是直接用这个已经不能用的句柄重试查询。这会导致从应用端看上去，“MySQL 一直没恢复”。

第二种方法：减少连接过程的消耗。

在 MySQL 8.0 版本里，如果你启用–skip-grant-tables 参数，MySQL 会默认把 --skipnetworking 参数打开，表示这时候数据库只能被本地的客户端连接。可见，MySQL 官方对 skip-grant-tables 这个参数的安全问题也很重视。

## 慢查询性能问题
### 索引没有设计好；

最高效的做法就是直接执行 alter table 语句。
比较理想的是能够在备库先执行。假设你现在的服务是一主一备，主库 A、备库 B，这个方案的大致流程是这样的：
1. 在备库 B 上执行 set sql_log_bin=off，也就是不写 binlog，然后执行 alter table 语句加上索引；
2. 执行主备切换；
3. 这时候主库是 B，备库是 A。在 A 上执行 set sql_log_bin=off，然后执行 alter table语句加上索引。
### SQL 语句没写好；

我们可以通过改写 SQL 语句来处理。MySQL 5.7 提供了 query_rewrite 功能，可以把输入的一种语句改写成另外一种模式。
### MySQL 选错了索引。

应急方案就是给这个语句加上 force index。
1. 上线前，在测试环境，把慢查询日志（slow log）打开，并且把 long_query_time 设置成 0，确保每个语句都会被记录入慢查询日志；
2. 在测试表里插入模拟线上的数据，做一遍回归测试；
3. 观察慢查询日志里每类语句的输出，特别留意 Rows_examined 字段是否与预期一致。

你需要工具帮你检查所有的 SQL语句的返回结果。比如，你可以使用开源工具 pt-querydigest(https://www.percona.com/doc/percona-toolkit/3.0/pt-query-digest.html)。

## QPS突增问题
1. 一种是由全新业务的 bug 导致的。假设你的 DB 运维是比较规范的，也就是说白名单是一个个加的。这种情况下，如果你能够确定业务方会下掉这个功能，只是时间上没那么快，那么就可以从数据库端直接把白名单去掉。
2. 如果这个新功能使用的是单独的数据库用户，可以用管理员账号把这个用户删掉，然后断开现有连接。这样，这个新功能的连接不成功，由它引发的 QPS 就会变成 0。
3. 如果这个新增的功能跟主体功能是部署在一起的，那么我们只能通过处理语句来限制。这时，我们可以使用上面提到的查询重写功能，把压力最大的 SQL 语句直接重写成"select 1"返回。

当然，这个操作的风险很高，需要你特别细致。它可能存在两个副作用：
1. 如果别的功能里面也用到了这个 SQL 语句模板，会有误伤；
2. 很多业务并不是靠这一个语句就能完成逻辑的，所以如果单独把这一个语句以 select 1的结果返回的话，可能会导致后面的业务逻辑一起失败。

所以，方案 3 是用于止血的，跟前面提到的去掉权限验证一样，应该是你所有选项里优先级最低的一个方案。


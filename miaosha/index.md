# 秒杀


## 主要挑战
瞬时高并发

高并发无法避开的热点数据问题

来自黑产的刷子流量

## 页面静态化
cdn加速

## 设计
{{<image src="/images/miaosha_process.png" height= "500" width= "1000"  title="miaosha process">}}
{{<image src="/images/miaosha_yugu.png" height= "500" width= "1000"  title="miaosha yugu">}}
{{<image src="/images/miaosha_loudou.png" height= "500" width= "1000"  title="miaosha loudou">}}

## 秒杀按钮
js文件控制

定时器

## 读多写少
redis

## 缓存问题
### 缓存击穿
缓存预热

分布式锁
### 缓存穿透
* 布隆过滤器：
适用于缓存更新很少的场景

* 将商品id加入缓存，设置超时时间尽量短
## 库存问题
### 预扣库存
* 数据库扣减库存

数据库的乐观锁：Stock>0在更新

问题：

死锁问题

容易造成系统雪崩

* redis扣减库存
非原子操作
* Lua脚本扣减库存
## 分布式锁
### Setnx加锁
### Set加锁
* lockKey
* requestId
* NX
* PX
* expireTime
### 释放锁
通过requestId，只释放自己的锁，不允许释放别人加的锁
### 自旋锁
解决均匀分布的秒杀问题
### Redission
在不同的节点上使用单个实例获取锁的方式去获得锁，且每次获取锁都有超时时间，如果请求超时，则认为该节点不可用。当应用服务成功获取锁的Redis节点超过半数（N/2+1，N为节点数)时，并且获取锁消耗的实际时间不超过锁的过期时间，则获取锁成功

## mq异步处理
### 下单功能
* 消息丢失问题

Job,增加重试

消息发送表
* 重复消费问题

消息处理表
{{<admonition>}}
下单和写消息处理表，需要在同一个事务当中，保证原子操作
{{</admonition>}}
* 垃圾消息问题

Job重试的时候，需要判断一下消息发送表该消息的发送次数是否达到最大限制
* 延迟消费问题

下单消息生产者先生成订单，向延迟队列发送消息

## 如何限流
### 对同一用户限流
### 对同一ip限流
### 对接口限流
### 加验证码
用户体验比较差
### 提高业务门槛

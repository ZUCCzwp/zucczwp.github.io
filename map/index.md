# map用法


### go内建的map类型
map的类型是map[key]

key类型的k必须是可比较的

map[key]函数返回结果可以是一个值，也可以是两个值

map是无序的，想要保证遍历mao时元素有序，可以使用orderedmap

### 使用map的2种常见错误
常见错误一：未初始化

常见错误二：并发读写

### 如何实现线程安全的map类型
#### 加读写锁:扩展map,支持并发读写

#### 分片加锁：更高效的并发map
getShard是一个关键的方法，能够根据key计算出分片索引

### 应对特殊场景的sync.map
#### 应用场景不多

#### 设计与实现
* store方法
* load方法
* delete方法
* loadAndDelete方法
* loadOrStore方法
* range方法

#### 实现优化点
* 空间换时间
* 优先从read字段读取，更新，删除
* 动态调整
* double-checking
* 延迟删除

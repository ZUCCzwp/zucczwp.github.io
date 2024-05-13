# 高性能


## 性能指标
### 响应时间
从发出请求开始到收到最后响应数据所需要的时间。响应时间是系统最
重要的性能指标，最直接地反映了系统的快慢。
### 并发数
系统同时处理的请求数，这个数字反映了系统的负载压力情况。
### 吞吐量
单位时间内系统处理请求的数量，体现的是系统的处理能力。我们一般用每秒
HTTP 请求数 HPS、每秒事务数 TPS、每秒查询数 QPS 这样的一些指标来衡量

吞吐量、响应时间和并发数三者之间是有关联性的。并发数不变，响应时间足够快，那么单
位时间的吞吐量就会相应的提高。

### 性能计数器
服务器或者操作系统性能的一些指标数据，包括系统负载 System
Load、对象和线程数、内存使用、CPU 使用、磁盘和网络 I/O 使用等指标，这些指标是系
统监控的重要参数，反映系统负载和处理能力的一些关键指标，通常这些指标和性能是强相
关的。这些指标很高，成为瓶颈，通常也预示着性能可能会出现问题。

## 性能测试
### 性能测试
性能测试是指以系统设计初期规划的性能指标为预期目标，对系统不断地施加压力，验证系
统在资源可接受的范围内是否达到了性能的预期目标。这个过程中，随着并发数的增加，吞
入量也在增加，但是响应时间变化不大。系统正常情况下的并发访问压力应该都在这个范围
内。

### 负载测试
负载测试则是对系统不断地施加并发请求，增加系统的压力，直到系统的某项或多项指标达
到安全临界值。这个过程中，随着并发数的增加，吞吐量只有小幅的增加，达到最大值后，
吞吐量还会下降，而响应时间则会不断增加。

### 压力测试
压力测试是指在超过安全负载的情况下，增加并发请求数，对系统继续施加压力，直到系统
崩溃，或者不再处理任何请求，此时的并发数就是系统的最大压力承受能力。这个过程中，
吞吐量迅速下降，响应时间迅速增加，到了系统崩溃点，吞吐量为 0，响应时间无穷大。

## 性能优化
### 用户体验优化
性能优化的最终目的是让用户有更好的性能体验，所以性能优化最直接的其实是优化用户体
验。同样 500 毫秒的响应时间，如果收到全部响应数据后才开始显示给用户，相比收到部
分数据就开始显示，对用户的体验就完全不一样。同样，在等待响应结果的时候，只显示一
个空白的页面和显示一个进度条，用户感受到的性能也是完全不同的。
#### 第一层：数据中心优化
在全球各个主要区域都部署自己
的数据中心，就近为区域用户提供服务，加快响应速度。
#### 第二层：硬件优化
事实上，即便使用水平伸缩，在分布式集群服务器内部，依然可以使用垂直伸缩，优化服务
器的硬件能力。有时候，硬件能力的提升，对系统性能的影响是非常巨大的。
#### 第三层：操作系统优化
不同操作系统以及操作系统内的某些特性也会对软件性能有重要影响
#### 第四层：虚拟机优化
特别是垃圾回收，可能会导致应用程序出现巨大的卡顿。
#### 第五层：基础组件优化
如 Web 容器，数据库连接
池，MVC 框架等等。这些基础组件的性能也会对系统性能有较大影响。
#### 第六层：架构优化
主要有缓存、消息队列、集群。
**缓存**：通过从缓存读取数据，加快响应时间，减少后端计算压力，缓存主要是提升读的性
能。

**消息队列**：通过将数据写入消息队列，异步进行计算处理，提升系统的响应时间和处理速
度，消息队列主要是提升写的性能。

**集群**：将单一服务器进行伸缩，构建成一个集群完成同一种计算任务，从而提高系统在高并
发压力时候的性能。各种服务器都可以构建集群，应用集群、缓存集群、数据库集群等等。
#### 第七层：代码优化
通过各种编程技巧和设计模式提升代码的执行效率，也是我们最能控制的一个优化手段。

此外，还可以使用线程池、连接池等对象池化技术，复用资源，减少资源的创建。
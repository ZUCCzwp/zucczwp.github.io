# 高可用架构


## 高可用的度量
{{< raw >}}
\[ 可用性=（1-年度不可用时间÷年度总时间）×100% \]
{{< /raw >}}
一般说来，两个 9 表示系统基本可用，年度不可用时间小于 88 小时；3 个 9 是较高可
用，年度不可用时间小于 9 个小时；4 个 9 是具有自动恢复能力的高可用，年度不可用时
间小于 53 分钟；5 个 9 指极高的可用性，年度不可用时间小于 5 分钟

在互联网企业中，为了更好地管理系统的可用
性，界定好系统故障以后的责任，通常会用故障分进行管理。一般过程是，根据系统可用性
指标换算成一个故障分，这个故障分是整个系统的故障分

## 高可用的架构
### 冗余备份
冗余备份是说，提供同一服务的服务器要存在冗余，即任何服务都不能只有一台服务器，服
务器之间要互相进行备份，任何一台服务器出现故障的时候，请求可以发送到备份的服务器
去处理。这样，即使某台服务器失效，在用户看来，系统依然是可用的。

**负载均衡可以实现系统的高可用。**

负载均衡服务器通过心跳检测发现集群中某台应用服务器失效，然后负载均衡服务器就不将
请求分发给这台服务器，对用户而言，也就感觉不到有服务器失效，系统依然可用。

**数据库主主复制，也是一种冗余备份**。这个时候，
不只是数据库系统 RDBMS 互相进行冗余备份，数据库里的数据也要进行冗余备份，一份
数据存储在多台服务器里，保证当任何一台服务器失效，数据库服务依然可以使用。

### 失败隔离
将失败限制在一个较小的范围之内，使故障影响
范围不扩大。具体实现**失败隔离的主要架构技术是消息队列**。

另一方面，由于分布式消息队列具有削峰填谷的作用，所以在高并发的时候，消息的生产者
可以将消息缓冲在分布式消息队列中，消费者可以慢慢地从消息队列中去处理，而不会将瞬
时的高并发负载压力直接施加到整个系统上，导致系统崩溃。也就是**将压力隔离开来**，使消
息生产者的访问压力不会直接传递到消息的消费者，这样可以提高数据库等对压力比较敏感
的服务的可用性。

同时，**消息队列还使得程序解耦，将程序的调用和依赖隔离开来**，我们知道，低耦合的程序
更加易于维护，也可以减少程序出现 Bug 的几率。
### 限流降级
限流和降级也是保护系统高可用的一种手段。在高并发场景下，如果系统的访问量超过了系
统的承受能力，可以通过限流对系统进行保护。限流是指对进入系统的用户请求进行流量限
制，如果访问量超过了系统的最大处理能力，就会丢弃一部分的用户请求，保证整个系统可
用，保证大部分用户是可以访问系统的。这样虽然有一部分用户的请求被丢弃，产生了部分
不可用，但还是好过整个系统崩溃，所有的用户都不可用要好。

降级是保护系统的另一种手段。有一些系统功能是非核心的，但是它也给系统产生了非常大
的压力。

### 异地多活
为了解决这个问题，同时也为了提高系统的处理能力和改善用户体验，很多大型互联网应用
都采用了异地多活的多机房架构策略，也就是说将数据中心分布在多个不同地点的机房里，
这些机房都可以对外提供服务，用户可以连接任何一个机房进行访问，这样每个机房都可以
提供完整的系统服务，即使某一个机房不可使用，系统也不会宕机，依然保持可用。

异地多活的架构考虑的重点就是，用户请求如何分发到不同的机房去。这个主要可以在域名
解析的时候完成，也就是用户进行域名解析的时候，会根据就近原则或者其他一些策略，完
成用户请求的分发。另一个至关重要的技术点是，因为是多个机房都可以独立对外提供服
务，所以也就意味着每个机房都要有完整的数据记录。用户在任何一个机房完成的数据操
作，都必须同步传输给其他的机房，进行数据实时同步。
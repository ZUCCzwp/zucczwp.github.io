# 分布式架构


## 垂直伸缩与水平伸缩
为了应对高并发用户访问带来的系统资源消耗，一种解决办法是垂直伸缩。所谓的垂直伸缩
就是提升单台服务器的处理能力

垂直伸缩带来的价格成本和服务器的处理能力并不一定呈线性关系，也就是说，增加同样的
费用，并不能得到同样的计算能力。而且计算能力越强大，需要花费的钱就越多。

所谓的水平伸缩，指的是不去提升单机的处理能力，不使用更昂贵更快更厉害的硬件，而是
使用更多的服务器，将这些服务器构成一个分布式集群，通过这个集群，对外统一提供服务，以此来提高系统整体的处理能力。
## 互联网分布式架构演化
![danti.png](/images/danti.png)

第一次分离,只需要把数据库、文件系统进行远程部署，进行
远程访问就可以了.

![first.png](/images/first.png)

使用缓存改善性能

![second.png](/images/second.png)

通过负载均衡服务器，将应用服务器部署为一个集群，添加更多的应用服务器去处
理用户的访问。

![third.png](/images/third.png)

数据库的读写分离，将一个数据库通过数据复制的方式，分裂为两个
数据库，主数据库主要负责数据的写操作，所有的写操作都复制到从数据库上，保证从数据
库的数据和主数据库数据一致，而从数据库主要提供数据的读操作。

![fourth.png](/images/fourth.png)

海量数据的存储，主要通过分布式数据库、分布式文件系统、NoSQL 数据库解决。直接在
数据库上查询已经无法满足这些数据的查询性能要求，还需要部署独立的搜索引擎提供查询
服务。同时减少数据中心的网络带宽压力，提供更好的用户访问延时，使用 CDN 和反向代
理提供前置缓存，尽快返回静态文件资源给用户。

为了使各个子系统更灵活易于扩展，则使用分布式消息队列将相关子系统解耦，通过消息的
发布订阅完成子系统间的协作。使用微服务架构将逻辑上独立的模块在物理上也独立部署，
单独维护，应用系统通过组合多个微服务完成自己的业务逻辑，实现模块更高级别的复用，
从而更快速地开发系统和维护系统。

![fiveth.png](/images/fiveth.png)

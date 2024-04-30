# Go-redis高级用法


### 开启对 Cluster中 Slave Node的访问
在一个负载比较高的Redis Cluster中，如果允许对slave节点进行读操作将极大的提高集群的吞吐能力。
开启对Slave 节点的访问，受以下3个参数的影响
```
type ClusterOptions struct {
    ReadOnly bool
    RouteByLatency bool
    RouteRandomly bool
    ... 
}
```
go-redis 选择节点的逻辑如下
```
func (c *ClusterClient) cmdSlotAndNode(cmd Cmder) (int, *clusterNode, error) {
    state, err := c.state.Get()
    if err != nil {
        return 0, nil, err
    }

    cmdInfo := c.cmdInfo(cmd.Name())
    slot := cmdSlot(cmd, cmdFirstKeyPos(cmd, cmdInfo))

    if c.opt.ReadOnly && cmdInfo != nil && cmdInfo.ReadOnly {
        if c.opt.RouteByLatency {
            node, err := state.slotClosestNode(slot)
            return slot, node, err
        }

        if c.opt.RouteRandomly {
            node := state.slotRandomNode(slot)
            return slot, node, nil
        }

        node, err := state.slotSlaveNode(slot)
        return slot, node, err
    }

    node, err := state.slotMasterNode(slot)
    return slot, node, err
}
```
* 如果ReadOnly = true，只选择Slave Node
* 如果ReadOnly = true 且 RouteByLatency = true 将从slot对应的Master Node 和 Slave Node选择，选择策略为: 选择PING 延迟最低的节点
* 如果ReadOnly = true 且 RouteRandomly = true 将从slot对应的Master Node 和 Slave Node选择，选择策略为:随机选择

### 在集群模式下使用pipeline功能
Redis的pipeline功能的原理是 Client通过一次性将多条redis命令发往Redis Server，减少了每条命令分别传输的IO开销。同时减少了系统调用的次数，因此提升了整体的吞吐能力。

我们在主-从模式的Redis中，pipeline功能应该用的很多，但是Cluster模式下，估计还没有几个人用过。

我们知道 redis cluster 默认分配了 16384 个slot，当我们set一个key 时，会用CRC16算法来取模得到所属的slot，然后将这个key 分到哈希槽区间的节点上，具体算法就是：CRC16(key) % 16384。如果我们使用pipeline功能，一个批次中包含的多条命令，每条命令涉及的key可能属于不同的slot

go-redis 为了解决这个问题, 分为3步

源码可以阅读 defaultProcessPipeline

1. 将计算command 所属的slot, 根据slot选择合适的Cluster Node
2. 将同一个Cluster Node 的所有command，放在一个批次中发送（并发操作）
3. 接收结果
{{<admonition >}}
注意：这里go-redis 为了处理简单，每个command 只能涉及一个key, 否则你可能会收到如下错误`err CROSSSLOT Keys in request don't hash to the same slot`

go-redis不支持类似 MGET 命令的用法
{{</admonition >}}
```
package main

import (
    "github.com/go-redis/redis"
    "fmt"
)

func main() {
    client := redis.NewClusterClient(&redis.ClusterOptions{
        Addrs: []string{"192.168.120.110:6380", "192.168.120.111:6380"},
        ReadOnly: true,
        RouteRandomly: true,
    })

    pipe := client.Pipeline()
    pipe.HGetAll("1371648200")
    pipe.HGetAll("1371648300")
    pipe.HGetAll("1371648400")
    cmders, err := pipe.Exec()
    if err != nil {
        fmt.Println("err", err)
    }
    for _, cmder := range cmders {
        cmd := cmder.(*redis.StringStringMapCmd)
        strMap, err := cmd.Result()
        if err != nil {
            fmt.Println("err", err)
        }
        fmt.Println("strMap", strMap)
    }
}
```

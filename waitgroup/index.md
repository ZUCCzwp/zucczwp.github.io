# waitgroup用法


### 基本用法
once常常用来初始化单例资源，或者并发访问只需初始化一次的共享资源，或者在测试的时候初始化一次测试资源

### 实现
Add方法逻辑：

Add 方法主要操作的是 state 的计数部分。你可以为计数值增加一个 delta 值，内部通过原子操作把这个值加到计数值上。需要注意的是，这个 delta 也可以是个负数，相当于为计数值减去一个值，Done 方法内部其实就是通过Add(-1) 实现的。

Wait方法逻辑：

不断检查 state 的值。如果其中的计数值变为了 0，那么说明所有的任务已完成，调用者不必再等待，直接返回。如果计数值大于 0，说明此时还有任务没完成，那么调用者就变成了等待者，需要加入 waiter 队列，并且阻塞住自己。

### 常见问题
#### 计数器设置为负数

解决方案：
a.调用add的时候传递一个负数

b.调用done方法的次数过多，超过了waitGroup的计数值

使用 WaitGroup 的正确姿势是，预先确定好 WaitGroup 的计数值，然后调用相同次数的 Done 完成相应的任务

#### 不期望的add时机

解决方案：等所有的 Add 方法调用之后再调用 Wait

#### 前一个wait还没结束就重用waitGroup

解决方案：WaitGroup 虽然可以重用，但是是有一个前提的，那就是必须等到上一轮的Wait 完成之后，才能重用 WaitGroup 执行下一轮的 Add/Wait，如果你在 Wait 还没执行完的时候就调用下一轮 Add 方法，就有可能出现 panic

### 基本用法
```
package main

import (
	"fmt"
	"sync"
	"time"
)

type Counter struct {
	mu    sync.RWMutex
	count uint64
}

func (c *Counter) Incr() {
	c.mu.Lock()
	c.count++
	c.mu.Unlock()
}

func (c *Counter) Count() uint64 {
	c.mu.RLock()
	defer c.mu.RUnlock()
	return c.count
}

func worker(c *Counter, wg *sync.WaitGroup) {
	defer wg.Done()
	time.Sleep(time.Second)
	c.Incr()
}

func main() {
	var counter Counter
	var wg sync.WaitGroup

	wg.Add(10)

	for i := 0; i < 10; i++ {
		go worker(&counter, &wg)
	}
	wg.Wait()
	fmt.Println(counter.Count())
}
```
### 实现
```
type WaitGroup struct {
	noCopy noCopy
	// 高 32 位为计数值，低 32 位是 waiter的计数
	state atomic.Uint64
	sema  uint32
}
```

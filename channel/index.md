# channel用法


CSP 允许使用进程组件来描述系统，它们独立运行，并且只通过消息传递的方式通信
### channel的应用场景
执行业务处理的 goroutine 不要通过共享内存的方式通信，而是要通过 Channel 通信的方式分享数据
五种类型：
1. 数据交流
2. 数据传递
3. 信号通知
4. 任务编排
5. 锁
### channel的基本用法
只能接收、只能发送、既可以接收又可以发送三种类型
* 发送数据
```
ch <- 2000
```
* 接收数据
```
x := <-ch //把接收的一条数据赋值给变量x
foo(<-ch) //把接收的一个数据作为参数传给函数
<-ch //丢弃接收的一条数据
```
* 其他操作

send和recv都可以作为select语句的case clause

可以用于for-range语句中

### 实现原理
数据结构

![channel.png](/images/channel.png)
* qcount：代表 chan 中已经接收但还没被取走的元素的个数。内建函数 len 可以返回这个字段的值。
* dataqsiz：队列的大小。chan 使用一个循环队列来存放元素，循环队列很适合这种生产者 - 消费者的场景（我很好奇为什么这个字段省略 size 中的 e）。
* buf：存放元素的循环队列的 buffer。
* elemtype 和 elemsize：chan 中元素的类型和 size。因为 chan 一旦声明，它的元素类型是固定的，即普通类型或者指针类型，所以元素大小也是固定的
* sendx：处理发送数据的指针在 buf 中的位置。一旦接收了新的数据，指针就会加上elemsize，移向下一个位置。buf 的总大小是 elemsize 的整数倍，而且 buf 是一个循环列表。
* recvx：处理接收请求时的指针在 buf 中的位置。一旦取出数据，此指针会移动到下一个位置。
* recvq：chan 是多生产者多消费者的模式，如果消费者因为没有数据可读而被阻塞了，就会被加入到 recvq 队列中。
* sendq：如果生产者因为 buf 满了而阻塞，会被加入到 sendq 队列

初始化

Go 在编译的时候，会根据容量的大小选择调用 makechan64，还是 makechan。

我们只关注 makechan 就好了，因为 makechan64 只是做了 size 检查，底层还是调用makechan 实现的。makechan 的目标就是生成 hchan 对象。

send

Go 在编译发送数据给 chan 的时候，会把 send 语句转换成 chansend1 函数，chansend1 函数会调用 chansend

recv

在处理从 chan 中接收数据时，Go 会把代码转换成 chanrecv1 函数，如果要返回两个返回值，会转换成 chanrecv2，chanrecv1 函数和 chanrecv2 会调用 chanrecv。

close

通过 close 函数，可以把 chan 关闭，编译器会替换成 closechan 方法的调用

### 容易犯的错误
常见panic
1. close 为 nil 的 chan；
2. send 已经 close 的 chan；
3. close 已经 close 的 chan

goroutine泄漏：
```
func process(timeout time.Duration) bool {
    ch := make(chan bool)

    go func() {
        // 模拟处理耗时的业务
        time.Sleep((timeout + time.Second))
        ch <- true // block
        fmt.Println("exit goroutine")
    }()
    select {
    case result := <-ch:
        return result
    case <-time.After(timeout):
        return false
    }
}
```
在这个例子中，process 函数会启动一个 goroutine，去处理需要长时间处理的业务，处理完之后，会发送 true 到 chan 中，目的是通知其它等待的 goroutine，可以继续处理了。

我们来看一下第 10 行到第 15 行，主 goroutine 接收到任务处理完成的通知，或者超时后就返回了。这段代码有问题吗？

如果发生超时，process 函数就返回了，这就会导致 unbuffered 的 chan 从来就没有被读取。我们知道，unbuffered chan 必须等 reader 和 writer 都准备好了才能交流，否则就会阻塞。超时导致未读，结果就是子 goroutine 就阻塞在第 7 行永远结束不了，进而导致 goroutine 泄漏。

解决这个 Bug 的办法很简单，就是将 unbuffered chan 改成容量为 1 的 chan，这样第 7 行就不会被阻塞了。

### 选择方法
* 共享资源的并发访问使用传统并发原语；
* 复杂的任务编排和消息传递使用 Channel；
* 消息通知机制使用 Channel，除非只想 signal 一个 goroutine，才使用 Cond
* 简单等待所有任务的完成用 WaitGroup，也有 Channel 的推崇者用 Channel，都可以；
* 需要和 Select 语句结合，使用 Channel；
* 需要和超时配合时，使用 Channel 和 Context。

|         | nil   | empty         | full          | not full&empty | closed          |
|---------|-------|---------------|---------------|----------------|-----------------|
| receive | block | block         | read value    | read value     | 返回未读的元素，读完后返回零值 |
| send    | block | write value   | block         | write value    | panic           |
| closed  | panic | closed,没有未读元素 | closed,保留未读元素 | closed,保留未读元素  | panic           |

# cond用法


<!--more-->
## go标准库cond
cond通常应用于等待某个条件一组gorotine，等条件变为true的时候，其中一个goroutine或者所有goroutine都会被唤醒执行

## cond的基本用法
### signal
允许调用者caller唤醒一个等待此cond的goroutine
### broadcase
允许调用者caller唤醒所有等待此cond的goroutine
### wait
会把调用者caller放入cond的等待队列中并阻塞，直到被signal或者broadcast的方法从等待队列中移除并唤醒

## 使用cond的2个常见错误
调用wait的时候没有加锁

只调用了一次wait，没有检查等待条件是否满足，结果条件没满足。程序就继续执行

## 知名项目中cond的使用
### K8s
定义了优先队列PriorityQueue这样一个数据结构，用来实现pod的调用

### cond有3点特性，是channel无法替代的
cond和一个locker关联，可以利用这个locker对相关的依赖条件更改提供保护

cond可以同时支持signal和broadcast方法，而channel只能同时支持其中一个

cond的broadcast方法可以被重复调用

## 使用案例
```markdown
var done = false

func read(name string, c *sync.Cond) {
	c.L.Lock()
	for !done {
		c.Wait()
	}
	log.Println(name, "starts reading")
	c.L.Unlock()
}

func write(name string, c *sync.Cond) {
	log.Println(name, "starts writing")
	time.Sleep(time.Second)
	c.L.Lock()
	done = true
	c.L.Unlock()
	log.Println(name, "wakes all")
	c.Broadcast()
}

func main() {
	cond := sync.NewCond(&sync.Mutex{})

	go read("reader1", cond)
	go read("reader2", cond)
	go read("reader3", cond)
	write("writer", cond)

	time.Sleep(time.Second * 3)
}
```

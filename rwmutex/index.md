# rwmutex用法


### 什么是RWMutex？
#### 方法：

Lock/Unlock：写操作时调用的方法

RLock/RUnlock：读操作时调用的方法

RLocker：为读操作返回一个Locker接口的对象
```
package main

import (
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

func (c *Counter) Value() uint64 {
	c.mu.RLock()
	defer c.mu.RUnlock()
	return c.count
}

func main() {
	var counter Counter
	for i := 0; i < 10; i++ {
		go func() {
			for {
				counter.Value()
				time.Sleep(time.Millisecond)
			}
		}()
	}

	for {
		counter.Incr()
		time.Sleep(time.Millisecond)
	}
}
```
#### 应用场景
如果你遇到可以明确区分 reader 和 writer goroutine 的场景，且有大量的并发读、少量的并发写，并且有强烈的性能需求，你就可以考虑使用读写锁 RWMutex 替换 Mutex。
#### 实现原理
readers-writers问题分为三类：
* Read-preferring
* Write-preferring
* 不指定优先级

Go 标准库中的 RWMutex 设计是 Write-preferring 方案。一个正在阻塞的 Lock 调用会排除新的 reader 请求到锁

#### 实现
RLock/RUnlock实现
```
func (rw *RWMutex) RLock() {
	if race.Enabled {
		_ = rw.w.state
		race.Disable()
	}
	if rw.readerCount.Add(1) < 0 {
		// A writer is pending, wait for it.
		runtime_SemacquireRWMutexR(&rw.readerSem, false, 0)
	}
	if race.Enabled {
		race.Enable()
		race.Acquire(unsafe.Pointer(&rw.readerSem))
	}
}
....
func (rw *RWMutex) RUnlock() {
    if race.Enabled {
        _ = rw.w.state
        race.ReleaseMerge(unsafe.Pointer(&rw.writerSem))
        race.Disable()
    }
    if r := rw.readerCount.Add(-1); r < 0 {
        // 有等待的 writer
        rw.rUnlockSlow(r)
    }
    if race.Enabled {
        race.Enable()
    }
}

func (rw *RWMutex) rUnlockSlow(r int32) {
    if r+1 == 0 || r+1 == -rwmutexMaxReaders {
        race.Enable()
        fatal("sync: RUnlock of unlocked RWMutex")
    }
    if rw.readerWait.Add(-1) == 0 {
        //最后一个 reader,writer有机会获取锁了
        runtime_Semrelease(&rw.writerSem, false, 1)
    }
}
```
 Lock
```
func (rw *RWMutex) Lock() {
	if race.Enabled {
		_ = rw.w.state
		race.Disable()
	}
	// 首先解决其他 writer的竞争问题
	rw.w.Lock()
	// 反转 readerCount,告诉 reader有 writer竞争锁
	r := rw.readerCount.Add(-rwmutexMaxReaders) + rwmutexMaxReaders
	// 如果当前 reader持有锁，那么需要等待
	if r != 0 && rw.readerWait.Add(r) != 0 {
		runtime_SemacquireRWMutex(&rw.writerSem, false, 0)
	}
	if race.Enabled {
		race.Enable()
		race.Acquire(unsafe.Pointer(&rw.readerSem))
		race.Acquire(unsafe.Pointer(&rw.writerSem))
	}
}
```
Unlock
```
func (rw *RWMutex) Unlock() {
	if race.Enabled {
		_ = rw.w.state
		race.Release(unsafe.Pointer(&rw.readerSem))
		race.Disable()
	}

	// 告诉 reader没有活跃的 writer了
	r := rw.readerCount.Add(rwmutexMaxReaders)
	if r >= rwmutexMaxReaders {
		race.Enable()
		fatal("sync: Unlock of unlocked RWMutex")
	}
	//  唤醒阻塞的 reader们
	for i := 0; i < int(r); i++ {
		runtime_Semrelease(&rw.readerSem, false, 0)
	}
	// 释放内部的互斥锁
	rw.w.Unlock()
	if race.Enabled {
		race.Enable()
	}
}
```
#### 3 个踩坑点
坑点1：不可复制

坑点2：重入导致死锁
场景 1：
```
func main() {
	l := &sync.RWMutex{}
	foo(l)
}

func foo(l *sync.RWMutex) {
	fmt.Println("in foo")
	l.Lock()
	bar(l)
	l.Unlock()
}

func bar(l *sync.RWMutex) {
	l.Lock()
	fmt.Println("in bar")
	l.Unlock()
}
```
场景二：

有活跃 reader 的时候，writer 会等待，如果我们在 reader 的读操作时调用 writer 的写操作（它会调用 Lock 方法），那么，这个 reader和 writer 就会形成互相依赖的死锁状态

场景三：

writer 依赖活跃的 reader -> 活跃的 reader 依赖新来的 reader -> 新来的 reader依赖 writer

```
func main() {
	var mu sync.RWMutex

	go func() {
		time.Sleep(100 * time.Millisecond)
		mu.Lock()
		fmt.Println("Lock")
		time.Sleep(100 * time.Millisecond)
		mu.Unlock()
		fmt.Println("Unlock")
	}()

	go func() {
		factorial(&mu, 10)
	}()

	select {}
}

func factorial(mu *sync.RWMutex, n int) int {
	if n < 1 {
		return 0
	}
	fmt.Println("Rlock")
	mu.RLock()
	defer func() {
		fmt.Println("RUnlock")
		mu.RUnlock()
	}()
	time.Sleep(100 * time.Millisecond)
	return factorial(mu, n-1) * n
}
```
坑点3：释放未加锁的RWMutex

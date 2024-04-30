# 有用的技巧


## 为整型常量值增加一个 String () 方法
如果你使用 iota 的自定义整型用于枚举，请始终增加一个 String () 方法。假设你写了这样的代码
```go
type State int

const (
    Running State = iota 
    Stopped
    Rebooting
    Terminated
)
```
如果你为 State 类型赋值，并且输出它，那么你将会看到一个数字

在这里 0 是没有任何意义的，除非你去看回你声明变量的代码。如果你为你的 State 类型增加一个 String () 方法，就不用再去看回声明变量的代码了。
```go
func (s State) String() string {
    switch s {
    case Running:
        return "Running"
    case Stopped:
        return "Stopped"
    case Rebooting:
        return "Rebooting"
    case Terminated:
        return "Terminated"
    default:
        return "Unknown"
    }
}
```
其实这些都可以通过 Stringer 工具去自动化实现：
[stringer](https://pkg.go.dev/golang.org/x/tools/cmd/stringer)

## 为访问 map 增加 setter，getters
如果你重度使用 map 读写数据，那么就为其添加 getter 和 setter 吧。通过 getter 和 setter 你可以将逻辑封分别装到函数里。这里最常见的错误就是并发访问。如果你在某个 goroutein 里有这样的代码：
```go
m["foo"] = bar
```
假设你在其他地方也使用这个 map。你必须把互斥量放得到处都是！然而通过 getter 和 setter 函数就可以很容易的避免这个问题：
```go
func Put(key, value string) {
    mu.Lock()
    m[key] = value
    mu.Unlock()
}

func Delete(key string) {
    mu.Lock()
    delete(m, key)
    mu.Unlock()
}
```
使用接口可以对这一过程做进一步的改进。你可以将实现完全隐藏起来。只使用一个简单的、设计良好的接口，然后让包的用户使用它们：
```go
type Storage interface {
    Delete(key string)
    Get(key string) string
    Put(key, value string)
}
```


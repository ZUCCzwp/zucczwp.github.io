# 使用Goroutine如何控制HTTP请求的并发量


### 场景
我们使用 go 并发调用接口发起 HTTP 请求时，只需要在 func() 前面加上 go 关键字就很容易完成了，就是因为让并发变得如此简单，所以有的时候我们就需要控制一下并发请求的数量。

现在有个需求：本地有一千万条手机号，需要调用聚合数据 手机号码归属地 接口，并记录省份、城市、区号、邮编、运营商等查询结果数据。
### 实现
```
package main

import(
    "fmt"
    "io/ioutil"
    "math/rand"
    "net/http"
    "net/url"
    "sync"
    "time"
)

type Limit struct {
    number  int
    channel chan struct{}
}

// Limit struct 初始化
func New(number int) *Limit {
    return &Limit{
        number:  number,
        channel: make(chan struct{}, number),
    }
}

// Run 方法：创建有限的 go f 函数的 goroutine
func (limit *Limit) Run(f func()) {
    limit.channel <- struct{}{}
    go func() {
        f()
        <-limit.channel
    }()
}

// WaitGroup 对象内部有一个计数器，从0开始
// 有三个方法：Add(), Done(), Wait() 用来控制计数器的数量
var wg = sync.WaitGroup{}

const (
    concurrency = 5 // 控制并发量
)

func main() {
    start := time.Now()
    limit := New(concurrency) // New Limit 控制并发量
    // 接口请求URL
    apiUrl := "http://apis.juhe.cn/mobile/get" // 不要使用接口地址测试
    //max := int(math.Pow10(8))                  // 模拟一千万数据
    max := 5                                    // 先测试5次吧

    // 初始化参数
    param := url.Values{}
    param.Set("key", "您申请的KEY") // 接口请求Key

    for i := 0; i < max; i++ {
        wg.Add(1)
        value := i
        goFunc := func() {
            fmt.Printf("start func: %dn", value)
            // 配置请求参数,方法内部已处理urlencode问题,中文参数可以直接传参
            phone := RandMobile()
            param.Set("phone", phone) // 需要查询的手机号码或手机号码前7位
            // 发送请求
            data, err := Get(apiUrl, param)
            if err != nil {
                fmt.Println(err)
                return
            }
            // 其它逻辑代码...
            fmt.Println("phone: ", phone, string(data))
            wg.Done()
        }
        limit.Run(goFunc)
    }

    // 阻塞代码防止退出
    wg.Wait()

    fmt.Printf("耗时: %fs", time.Now().Sub(start).Seconds())
}

// Get 方式发起网络请求
func Get(apiURL string, params url.Values) (rs []byte, err error) {
    var Url *url.URL
    Url, err = url.Parse(apiURL)
    if err != nil {
        fmt.Printf("解析url错误:rn%v", err)
        return nil, err
    }
    //如果参数中有中文参数,这个方法会进行URLEncode
    Url.RawQuery = params.Encode()
    resp, err := http.Get(Url.String())
    if err != nil {
        fmt.Println("err:", err)
        return nil, err
    }
    defer resp.Body.Close()
    return ioutil.ReadAll(resp.Body)
}

var MobilePrefix = [...]string{"130", "131", "132", "133", "134", "135", "136", "137", "138", "139", "145", "147", "150", "151", "152", "153", "155", "156", "157", "158", "159", "170", "176", "177", "178", "180", "181", "182", "183", "184", "185", "186", "187", "188", "189"}

// GeneratorPhone 生成手机号码
func RandMobile() string {
    return MobilePrefix[RandInt(0, len(MobilePrefix))] + fmt.Sprintf("%0*d", 8, RandInt(0, 100000000))
}

// 指定范围随机 int
func RandInt(min, max int) int {
    rand.Seed(time.Now().UnixNano())
    return min + rand.Intn(max-min)
}
```

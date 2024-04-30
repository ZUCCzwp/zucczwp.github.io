# random用法


### golang生成随机数可以使用math/rand包
示例如下：
```
package main

import (
    "fmt"
    "math/rand"
)

func main() {
    for i:=0; i<10; i++ {
        fmt.Println(rand.Intn(100))
    }
}
```

而发现这种情况,每次执行的结果一样.
修改如下：
```
package main

import (
    "fmt"
    "time"
    "math/rand"
)

func main() {
    r := rand.New(rand.NewSource(time.Now().UnixNano()))
    for i:=0; i<10; i++ {
        fmt.Println(r.Intn(100))
    }
}
```
而这种方式就可以使用时间种子来获取不同的结果了。
示例2：
```
package main

import (  
    "fmt"  
    "math/rand"  
    "time"  
)

func main() {  
    rand.Seed(time.Now().UnixNano())  
    for i := 0; i < 10; i++ {  
        x := rand.Intn(100)  
        fmt.Println(x)  
    }  
}
```
例子是打印10个100以内（0-99）的随机数字。

# context用法


## 来历
Go在1.7的版本中才正式把context加入到标准库中

## 适用场景

### 上下文信息传递
### 控制子goroutine的运行
### 超时控制
### 可以取消的方法调用

## 基本使用方法
### 4个实现方法
deadline方法：会返回这个context会被取消的截止日期

done方法：返回一个channel对象

Err:如果done没有close，Err方法返回nil；如果done被close,Err会返回done被close的原因

value返回此ctx中和制定的key相关联的value

### 常用的生成顶层context的方法
Context.Backgroud():返回一个非nil、空的Context，没有任何之，不会被cancel,不会超时，没有截止日期

Context.TODO()：返回一个非nil、空的context，没有任何值，不会被cancel，不会超时，没有截止日期
### 适用规则
一般函数使用context的时候，会把这个参数放在第一个参数的位置

从来不把nil当作context类型的参数值，可以使用context.Backgroud()创建一个空的上下文对象，也不要使用nil

context只用来临时做函数之间的上下文透传，不能持久化context或者context长久保存

key的类型不应该使用字符串类型或者其他内建类型，否则容易在包之间使用context时候冲突

常常使用struct{}作为底层类型定义key的类型

## 创建特殊用途context的方法
WithValue:基于parent context生成一个新的context，保存了一个key-value简直对，常常用来传递上下文

WithCancel:返回parent的副本，只是副本中的done channel是新建的对象，他的类型是cancelCtx

WithTimeout:其实和WithDeadline一样，只不过一个参数是超时时间，一个参数是截止时间

WithDeadline：返回一个parent的副本，并且设置了一个不晚于参数d的截止时间，类型为timeCtx


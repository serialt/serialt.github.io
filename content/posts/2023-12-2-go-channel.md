+++

title = 'Go channel'
date = 2023-12-01T20:22:27+08:00
draft = false

tags = ["go-channel","channel"]
categories = ["Go基础"]

+++

# 通道 channel

## 介绍

### 概括

Go语言设计团队的首任负责人*Rob Pike*对并发编程的一个建议是**不要让计算通过共享内存来通讯，而应该让它们通过通讯来共享内存**。 通道机制就是这种哲学的一个设计结果。（在Go编程中，我们可以认为一个计算就是一个协程。）

通过共享内存来通讯和通过通讯来共享内存是并发编程中的两种编程风格。 当通过共享内存来通讯的时候，我们需要一些传统的并发同步技术（比如互斥锁）来避免数据竞争。

Go提供了一种独特的并发同步技术来实现通过通讯来共享内存。此技术即为通道。 我们可以把一个通道看作是在一个程序内部的一个先进先出（FIFO：first in first out）数据队列。 一些协程可以向此通道发送数据，另外一些协程可以从此通道接收数据。

随着一个数据值的传递（发送和接收），一些数据值的所有权从一个协程转移到了另一个协程。 当一个协程发送一个值到一个通道，我们可以认为此协程释放了（通过此发送值可以访问到的）一些值的所有权。 当一个协程从一个通道接收到一个值，我们可以认为此协程获取了（通过此接受值可以访问到的）一些值的所有权。

当然，在通过通道传递数据的时候，也可能没有任何所有权发生转移。

所有权发生转移的值常常被传递的值所引用着，但有时候也并非如此。 在Go中，数据所有权的转移并非体现在语法上，而是体现在逻辑上。 Go通道可以帮助程序员轻松地避免数据竞争，但不会防止程序员因为犯错而写出错误的并发代码的情况发生。

尽管Go也支持几种传统的数据同步技术，但是只有通道为一等公民。 通道是Go中的一种类型，所以我们可以无需引进任何代码包就可以使用通道。 几种传统的数据同步技术提供在sync和sync/atomic标准库包中。



### 通道类型和值

和数组、切片以及映射类型一样，每个通道类型也有一个元素类型。 一个通道只能传送它的（通道类型的）元素类型的值。

通道可以是双向的，也可以是单向的。

- 字面形式chan T表示一个元素类型为T的双向通道类型。 编译器允许从此类型的值中接收和向此类型的值中发送数据。
- 字面形式chan<- T表示一个元素类型为T的单向发送通道类型。 编译器不允许从此类型的值中接收数据。
- 字面形式<-chan T表示一个元素类型为T的单向接收通道类型。 编译器不允许向此类型的值中发送数据。

双向通道chan T的值可以被隐式转换为单向通道类型chan<- T和<-chan T，但反之不行（即使显式也不行）。 类型chan<- T和<-chan T的值也不能相互转换。

通道类型的零值也使用预声明的nil来表示。 一个非零通道值必须通过内置的make函数来创建。 比如make(chan int, 10)将创建一个元素类型为int的通道值。 第二个参数指定了欲创建的通道的容量。此第二个实参是可选的，它的默认值为0。



### 通道操作

```go

// 定义一个 channel
var ch = make(chan struct{})
	
// 关闭 channel
close(ch)

// 传入数据
ch <- struct{}{}

// 接受数据
<-ch
msg <-ch

// 查询 channel容量
cap(ch)

// 查询 channel 长度
len(ch)
```



### 操作 channel

channel有三类：

* 零值（nil）通道；

* 非零值但已关闭的通道；

* 非零值并且尚未关闭的通道。

下表简单地描述了三种通道操作施加到三类通道的结果。

| 操作     | 一个零值nil通道 | 一个非零值但已关闭的通道 | 一个非零值且尚未关闭的通道 |
| :------- | :-------------- | :----------------------- | :------------------------- |
| 关闭     | 产生恐慌        | 产生恐慌                 | 成功关闭(C)                |
| 发送数据 | 永久阻塞        | 产生恐慌                 | 阻塞或者成功发送(B)        |
| 接收数据 | 永久阻塞        | 永不阻塞(D)              | 阻塞或者成功接收(A)        |

对于上表中的五种未打上标的情形，规则很简单：

- 关闭一个nil通道或者一个已经关闭的通道将产生一个恐慌。
- 向一个已关闭的通道发送数据也将导致一个恐慌。
- 向一个nil通道发送数据或者从一个nil通道接收数据将使当前协程永久阻塞。

示例：

```go
package main
import (
   "fmt"
   "time"
)
func main() {
   c := make(chan int) // 一个非缓冲通道
   go func(ch chan<- int, x int) {
      time.Sleep(time.Second)
      // <-ch    // 此操作编译不通过
      ch <- x*x  // 阻塞在此，直到发送的值被接收
   }(c, 3)
   done := make(chan struct{})
   go func(ch <-chan int) {
      n := <-ch      // 阻塞在此，直到有值发送到c
      fmt.Println(n) // 9
      // ch <- 123   // 此操作编译不通过
      time.Sleep(time.Second)
      done <- struct{}{}
   }(c)
   <-done // 阻塞在此，直到有值发送到done
   fmt.Println("bye")
}
```

一场永不休场的足球比赛：

```go
package main
import (
   "fmt"
   "time"
)
func main() {
   var ball = make(chan string)
   kickBall := func(playerName string) {
      for {
         fmt.Print(<-ball, "传球", "\n")
         time.Sleep(time.Second)
         ball <- playerName
      }
   }
   go kickBall("张三")
   go kickBall("李四")
   go kickBall("王二麻子")
   go kickBall("刘大")
   ball <- "裁判"   // 开球
   var c chan bool // 一个零值nil通道
   <-c             // 永久阻塞在此
}
```



## 通道遍历

### for-range

for-range循环控制流程也适用于通道。 此循环将不断地尝试从一个通道接收数据，直到此通道关闭并且它的缓冲队列为空为止。 和应用于数组/切片/映射的for-range语法不同，应用于通道的for-range语法中最多只能出现一个循环变量，此循环变量用来存储接收到的值。

```go
for v := range aChannel {
   // 使用v
}
// 等价于
for {
   v, ok = <-aChannel
   if !ok {
      break
   }
   // 使用v
}


   for x := range c {
      time.Sleep(time.Second)
      fmt.Println(x)
   }
```

### select-case

Go中有一个专门为通道设计的select-case分支流程控制语法。 此语法和switch-case分支流程控制语法很相似。 比如，select-case流程控制代码块中也可以有若干case分支和最多一个default分支。 但是，这两种流程控制也有很多不同点。在一个select-case流程控制中，

- select关键字和{之间不允许存在任何表达式和语句。
- fallthrough语句不能被使用.
- 每个case关键字后必须跟随一个通道接收数据操作或者一个通道发送数据操作。 通道接收数据操作可以做为源值出现在一条简单赋值语句中。 以后，一个case关键字后跟随的通道操作将被称为一个case操作。
- 所有的非阻塞case操作中将有一个被随机选择执行（而不是按照从上到下的顺序），然后执行此操作对应的case分支代码块。
- 在所有的case操作均为阻塞的情况下，如果default分支存在，则default分支代码块将得到执行； 否则，当前协程将被推入所有阻塞操作中相关的通道的发送数据协程队列或者接收数据协程队列中，并进入阻塞状态。

按照上述规则，一个不含任何分支的select-case代码块select{}将使当前协程处于永久阻塞状态。

```go
// default分支将铁定得到执行，因为两个case分支后的操作均为阻塞的。
package main
import "fmt"
func main() {
   var c chan struct{} // nil
   select {
   case <-c:             // 阻塞操作
   case c <- struct{}{}: // 阻塞操作
   default:
      fmt.Println("Go here.")
   }
}
```

下面这个例子中实现了尝试发送（try-send）和尝试接收（try-receive）。 它们都是用含有一个case分支和一个default分支的select-case代码块来实现的。

```go
package main
import "fmt"
func main() {
   c := make(chan string, 2)
   trySend := func(v string) {
      select {
      case c <- v:
      default: // 如果c的缓冲已满，则执行默认分支。
      }
   }
   tryReceive := func() string {
      select {
      case v := <-c: return v
      default: return "-" // 如果c的缓冲为空，则执行默认分支。
      }
   }
   trySend("Hello!") // 发送成功
   trySend("Hi!")    // 发送成功
   trySend("Bye!")   // 发送失败，但不会阻塞。
   // 下面这两行将接收成功。
   fmt.Println(tryReceive()) // Hello!
   fmt.Println(tryReceive()) // Hi!
   // 下面这行将接收失败。
   fmt.Println(tryReceive()) // -
}
```





## 使用示例：

### 使用通道实现通知

#### 向一个通道发送一个值来实现单对单通知

```go
package main
import (
   "crypto/rand"
   "fmt"
   "os"
   "sort"
)
func main() {
   values := make([]byte, 32 * 1024 * 1024)
   if _, err := rand.Read(values); err != nil {
      fmt.Println(err)
      os.Exit(1)
   }
   done := make(chan struct{}) // 也可以是缓冲的
   // 排序协程
   go func() {
      sort.Slice(values, func(i, j int) bool {
         return values[i] < values[j]
      })
      done <- struct{}{} // 通知排序已完成
   }()
   // 并发地做一些其它事情...
   <- done // 等待通知
   fmt.Println(values[0], values[len(values)-1])
}
```

从一个通道接收一个值来实现单对单通

```go
package main
import (
   "fmt"
   "time"
)
func main() {
   done := make(chan struct{})
      // 此信号通道也可以缓冲为1。如果这样，则在下面
      // 这个协程创建之前，我们必须向其中写入一个值。
   go func() {
      fmt.Print("Hello")
      // 模拟一个工作负载。
      time.Sleep(time.Second * 2)
      // 使用一个接收操作来通知主协程。
      <- done
   }()
   done <- struct{}{} // 阻塞在此，等待通知
   fmt.Println(" world!")
}
```

### 使当前协程永久阻塞

可以用一个无分支的select流程控制代码块使当前协程永久处于阻塞状态。 这是select流程控制的最简单的应用。 事实上，上面很多例子中的for {time.Sleep(time.Second)}都可以换为select{}。

```go
package main
import "runtime"
func DoSomething() {
   for {
      // 做点什么...
      runtime.Gosched() // 防止本协程霸占CPU不放
   }
}
func main() {
   go DoSomething()
   go DoSomething()
   select{}
}
```


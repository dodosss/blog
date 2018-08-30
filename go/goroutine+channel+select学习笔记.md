 
# goroutine+channel+select学习笔记



## 通过sync包中的WaitGroup实现并发控制

注：A WaitGroup must not be copied after first use.

https://www.jianshu.com/p/6032f2db6be5

go 中五种引用类型有 slice， channel， function， map， interface


出现死锁。这是因为 wg 给拷贝传递到了 goroutine 中，导致只有 Add 操作，其实 Done操作是在 wg 的副本执行的。因此 Wait 就死锁了。

改正方法一：

将匿名函数中 wg 的传入类型改为 ```*sync.WaitGrou```，这样就能引用到正确的WaitGroup了。

改正方法二：

将匿名函数中的 wg 的传入参数去掉，因为Go支持闭包类型，在匿名函数中可以直接使用外面的 wg 变量

## channel

Channel是Go中的一个核心类型，可以把它看成一个管道，通过它可以发送或者接收数据进行通讯(communication)。配合goroutine，就形成了一种既简单又强大的请求处理模型，即N个工作goroutine将处理的中间结果或者最终结果放入一个channel，另外有M个工作goroutine从这个channel拿数据，再进行进一步加工，通过组合这种过程，可以胜任各种复杂的业务模型。

### channel的基本操作：

```
var c chan int   //声明一个int类型的channel，注意，该语句仅声明，不初始化channel

c := make(chan type_name)   //创建一个无缓冲的type_name型的channel，无缓冲的channel当放入1个元素后，后续的输入便会阻塞

c := make(chan type_name, 100)   //创建一个缓冲区大小为100的type_name型的channel

c <- x   //将x发送到channel c中，如果channel缓冲区满，则阻塞当前goroutine

<- c   //从channel c中接收一个值，如果缓冲区为空，则阻塞

x = <- c   //从channel c中接收一个值并存到x中，如果缓冲区为空，则阻塞

x, ok = <- c   //从channel c中接收一个值，如果channel关闭了，那么ok为false（在没有defaultselect语句的前提下），在channel未关闭且为空的情况下，仍然阻塞

close(c)   //关闭channel c

for term := range c {}   //等待并取出channelc中的值，直到channel关闭，会阻塞
```

单向channel：

```
var ch1 chan<- float64    //只能向里面写入float64的数据，不能读取

var ch2 <-chan int        //只能读取int型数据

```

在channel的用法中，最常见的包括写入和读出：

```
// 将一个数据value写入至channel，这会导致阻塞，直到有其他goroutine从这个channel中读取数据
ch <- value

// 从channel中读取数据，如果channel之前没有写入数据，也会导致阻塞，直到channel中被写入数据为止
value := <-ch
```

### select：

select用于在多个channel上同时进行侦听并收发消息，当任何一个case满足条件时即执行，如果没有可执行的case则会执行default的case，如果没有指定defaultcase，则会阻塞程序，select的语法：

select语句是会跳过nil的channels的. 因为在Go里往已经close掉的channel里发送数据是会panic的, 可以利用select语句.

附: channel操作导致panic的情况有: 关闭一个nil的channel, 关闭一个已经关闭的channel( j,ok:= <- ch, ok为false时代表ch已经关闭了), 往一个已经关闭的channel里发送数据(从已经关闭的channel里读数据是OK的, 如果这个channel是带缓冲的, 那么可以读到所有数据)

https://imhanjm.com/2017/06/24/go%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5/



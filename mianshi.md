### 1. go的熟练程度
### 2. 分布式CAP理论

CAP理论概述

一个分布式系统最多只能同时满足一致性（Consistency）、可用性（Availability）和分区容错性（Partition tolerance）这三项中的两项。

Consistency 一致性

一致性指“all nodes see the same data at the same time”，即所有节点在同一时间的数据完全一致。

Availability 可用性
可用性指“Reads and writes always succeed”，即服务在正常响应时间内一直可用。

好的可用性主要是指系统能够很好的为用户服务，不出现用户操作失败或者访问超时等用户体验不好的情况。可用性通常情况下可用性和分布式数据冗余，负载均衡等有着很大的关联。

Partition Tolerance分区容错性

分区容错性指“the system continues to operate despite arbitrary message loss or failure of part of the system”，即分布式系统在遇到某节点或网络分区故障的时候，仍然能够对外提供满足一致性或可用性的服务。

CAP权衡

- CA without P：如果不要求P（不允许分区），则C（强一致性）和A（可用性）是可以保证的。但其实分区不是你想不想的问题，而是始终会存在，因此CA的系统更多的是允许分区后各子系统依然保持CA。
- CP without A：如果不要求A（可用），相当于每个请求都需要在Server之间强一致，而P（分区）会导致同步时间无限延长，如此CP也是可以保证的。很多传统的数据库分布式事务都属于这种模式。
- AP wihtout C：要高可用并允许分区，则需放弃一致性。一旦分区发生，节点之间可能会失去联系，为了高可用，每个节点只能用本地数据提供服务，而这样会导致全局数据的不一致性。现在众多的NoSQL都属于此类。


### 3. redis
### 4. 数据库
### 5. 操作系统
### 6. 连接达到上限了怎么办？

连接池

### 7. 缓存穿透，击穿，雪崩 是什么 怎么解决？

### 8. 对自己的定位，初中高资深专家？

### 9. http 网络请求整个流程？及相关安全问题？

1.URL解析/DNS解析查找域名IP地址

2.网络连接发起HTTP请求

3.HTTP报文传输过程

4.服务器接收数据

5.服务器响应请求/MVC

6.服务器返回数据

7.客户端接收数据

8.浏览器加载/渲染页面

### 10. go并发？单例？

多协程 单进程

### 11. go 切片

https://zhuanlan.zhihu.com/p/88741532

是对原数组的引用
改变切片的值 也会改变数组的对应值

```
// arr := [8]int{1, 2, 3, 4, 5, 6, 7, 8}
s := arr[3:6]
s[0] = 10
fmt.Println(s) // [10 5 6]
fmt.Println("capicity", cap(s)) // 5
```

1) 切片赋值
```
ar s1 = []int{1, 2, 3}  // 初始化一个有3个元素的切片
var s2 = s1              // 将切片s1赋值给一个新的切片s2
s2[0] = 99               // 将s2的第一个元素设为99
fmt.Println(s1[0])       // 此时s1[0]是多少
```

2）切片的函数传递
```
//定义一个函数，给切片添加一个元素
func addOne(s []int) {
    s = append(s, 1)
}
var s1 = []int{2}   // 初始化一个切片
addOne(s1)          // 调用函数添加一个切片
fmt.Println(s1)
```
此时打印s1是应该显示{2, 1}吗？答案错误。显示的还是{2}。
首先要明白Golang函数所有的参数传递都是值传递，也就是说将切片s1传递给函数addOne后，函数addOne其实是将切片s1复制了一份，然后在函数内部对复制出来的切片s进行append，append后切片s的len变成了2，打印切片s的话会显示{2, 1}。虽然切片s1的array和函数内切片s的array是同一份，都有两个元素，但是切片s1的len还是1，所以打印出来的s1也就只有一个元素{2}.

二维切片的初始化
```
var s = make([][]int, n)
for i := 0; i < n; i++ { s[i] = make([]int, n) }
```

### 12. 对go 语言对理解？

Go语言是非常parsimonious的，可以说是一种吝啬，除非极其必要，不提供任何语法糖，do while是没有的，i++ i--是没有返回值的，等等等等！

### 13. RabbitMQ五种模式


### 14. redis 面试题？

https://blog.csdn.net/ThinkWon/article/details/103522351

### 15. mq的消费性能：

1.4s 200

### 15. mq常用三大模式？

- Direct 模式

所有发送到 Direct Exchange 的消息被转发到 RouteKey 中指定的 Queue。
Direct 模式可以使用 RabbitMQ 自带的 Exchange: default Exchange，所以不需要将 Exchange 进行任何绑定(binding)操作。
消息传递时，RouteKey 必须完全匹配才会被队列接收，否则该消息会被抛弃，

- Topic 模式：

可以使用通配符进行模糊匹配

符号'#" 匹配一个或多个词
符号"*”匹配不多不少一个词
例如:

'log.#"能够匹配到'log.info.oa"
"log.*"只会匹配到"log.erro“

- Fanout 模式

不处理路由键，只需要简单的将队列绑定到交换机上发送到交换机的消息都会被转发到与该交换机绑定的所有队列上。
Fanout交换机转发消息是最快的。

### 16. redis 持久化策略？

rdb, aof

- rdb 性能最好，默认的持久化策略 只有一个dump.rdb 文件 方便持久化
性能最大化，fork 子进程来完成写操作，让主进程继续处理命令

缺点：RDB 是间隔一段时间进行持久化，数据安全性低


- aof 是将Redis执行的每次写命令记录到单独的日志文件中
当两种方式同时开启时，数据恢复Redis会优先选择AOF恢复。

数据安全性高

优点：每进行一次 命令操作就记录到 aof 文件中一次。AOF 文件比 RDB 文件大，且恢复速度慢。

### 16. redis 常用类型

Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。

### 17. Redis 主从同步

- 全量同步

- 增量同步


### 18. mysql，事务隔离级别，锁，http请求过程。

- MySQL事务隔离级别

事务隔离级别｜ 脏读 | 不可重复读 | 幻读
---|---|---|---
读未提交（read-uncommitted）是|	是|	是
不可重复读（read-committed） | 否 |	是 | 是
可重复读（repeatable-read）	| 否 |	否 | 是
串行化（serializable）|	否 |	否 | 否 

- 读未提交是指，一个事务还没提交时，它做的变更就能被别的事务看到。
- 读提交是指，一个事务提交之后，它做的变更才会被其他事务看到。
- 可重复读是指，一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。当然在可重复读隔离级别下，未提交变更对其他事务也是不可见的。
- 串行化，顾名思义是对于同一行记录，“写”会加“写锁”，“读”会加“读锁”。当出现读写锁冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行。

- mysql默认的事务隔离级别为repeatable-read

### 19 数据库的mvcc 数据库的多版本并发控制？

- 不同时刻启动的事务会有不同的 read-view。如图中看到的，在视图 A、B、C 里面，这一个记录的值分别是 1、2、4，同一条记录在系统中可以存在多个版本，就是数据库的多版本并发控制（MVCC）

- 长事务的缺点？

长事务意味着系统里面会存在很老的事务视图。由于这些事务随时可能访问数据库里面的任何数据，所以这个事务提交之前，数据库里面它可能用到的回滚记录都必须保留，这就会导致大量占用存储空间。

### 20. linux 5 种 I/O 模型

https://www.jianshu.com/p/486b0965c296

- 同步模型（synchronous IO）
- 阻塞IO（bloking IO）
- 非阻塞IO（non-blocking IO）
- 多路复用IO（multiplexing IO）
- 信号驱动式IO（signal-driven IO）
- 异步IO（asynchronous IO）

### 21. go runtime 包？

- runtime.GOMAXPROCS 设置运行时系统中p的最大数量

### 22. laravel 面试题

https://zhuanlan.zhihu.com/p/196780449
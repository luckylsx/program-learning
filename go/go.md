### Go 相关题目部分1

#### 1. go并发？单例？

多协程 单进程

通过互斥锁实现单例
```
// get singleton with mutex lock
func getSingletonWithLock() *Singleton {
	if singleton == nil {
		mutex.Lock()
		singleton = &Singleton{}
		fmt.Println("get singleton...")
		mutex.Unlock()
	}
	return singleton
}
```

通过sync.Once 方是实现单例模式
```
// get the singleton with sync.Once
func getSingleton() *Singleton {
	once.Do(func() {
		singleton = &Singleton{}
		fmt.Println("get singleton...")
	})
	return singleton
}
```

#### 2. slice 的扩展？
```
arr := [...]int{0,1,2,3,4,5,6,7}
s1 := arr[2:6]
s2 := s1[3:5]
```

- s1 的值为 [2,3,4,5], s2的值为 [5,6]
- slice 可以向后扩展，不可以向前扩展
- s[i] 不可以超越len(s), 向后扩展不可以超越底层数组cap(s)

append 操作
```
arr := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8}
fmt.Printf("%#v\n", arr)
s := arr[3:7]
s1 := s[2:4]
fmt.Println("s = ", s)
fmt.Println("s1 = ", s1)

fmt.Println("append")
s1 = append(s1, 100)
fmt.Println("s1 = ", s1)
fmt.Println("arr = ", arr)

// output:
[9]int{0, 1, 2, 3, 4, 5, 6, 7, 8}
s =  [3 4 5 6]
s1 =  [5 6]
append
s1 =  [5 6 100]
arr =  [0 1 2 3 4 5 6 100 8]
```

- 添加元素时如果超越了cap ,系统会重新分配更大的底层数组
- 由于值传递的关系，必须接收append 的返回值

```
// copy slice
copy(变量, srcSlice)

// delete s[2]
s = append(s[:1], s[2:]...)

// popping from front
front := s[0]
s = s[1:]

// popping from back
tail := s[len(s)-1]
s = s[:len(s)-1]
```

### 3. 切片和数组的区别?

1. 声明数组时，方括号内写明了数组的长度或者...,声明slice时候，方括号内为空
2. 作为函数参数时，数组传递的是数组的副本，而slice传递的是指针。

**golang array 特点：**

- golang中的数组是值类型,也就是说，如果你将一个数组赋值给另外一个数组，那么，实际上就是整个数组拷贝了一份

- 如果golang中的数组作为函数的参数，那么实际传递的参数是一份数组的拷贝，而不是数组的指针

- array的长度也是Type的一部分，这样就说明[10]int和[20]int是不一样的。

**slice类型**

- slice是一个引用类型，是一个动态的指向数组切片的指针。
- slice是一个不定长的，总是指向底层的数组array的数据结构。

### 4. defer是怎么执行?

defer 类似 栈 后进先出

### 5. return和defer哪个最后执行?

defer 最后执行

### 6. map的底层实现？

golang map 底层是一个散列表，实现map 的过程就是实现散列表的过程。在这个散列表中，主要的结构体有两个，一个叫hmap(a header of a go map),一个叫bucket.map 的key 经过hash 运算之后，得到一个16位的字符。然后分为低8位和高8位。低八位找在哪个bucket，高八位找具体在bucket的哪个位置。

### 7. go 的GC 过程？

三色标记：

通过mspan查看是否被引用

- 灰色：对象已被标记，但这个对象包含的子对象未标记
- 黑色：对象已被标记，且这个对象包含的子对象也已标记，gcmarkBits对应的位为1（该对象不会在本次GC中被清理）
- 白色：对象未被标记，gcmarkBits对应的位为0（该对象将会在本次GC中被清理）

初始所有内存都是白色的，进入标记队列就是灰色

将root 跟对象放入标记队列（放入标记队列里的就是灰色）

从标记队列里面取出对象，标记为黑色（不能GC）

到这一阶段，所有内存要么是黑色的要么是白色的，清楚所有白色的即可

**GC触发条件**

- 内存大小阈值， 内存达到上次gc后的2倍
- 达到定时时间 ，2m interval

### 8. 关闭后的通道有以下特点：
- 对一个关闭的通道再发送值就会导致panic。
- 对一个关闭的通道进行接收会一直获取值直到通道为空。
- 对一个关闭的并且没有值的通道执行接收操作会得到对应类型的零值。
- 关闭一个已经关闭的通道会导致panic。

### 9. go的调度 ？

- 说到调度，我们首先想到的就是操作系统对进程、线程的调度。操作系统调度器会将系统中的多个线程按照一定算法调度到物理CPU上去运行。

- Goroutine占用的资源非常小(Go 1.4将每个goroutine stack的size默认设置为2k)，goroutine调度的切换也不用陷入(trap)操作系统内核层完成，代价很低。因此，一个Go程序中可以创建成千上万个并发的goroutine。所有的Go代码都在goroutine中执行，哪怕是go的runtime也不例外。
- 一个Go程序对于操作系统来说只是一个用户层程序，对于操作系统而言，它的眼中只有thread。
- go调度器中抽象出了G、P、M三种角色。除此之外，为了防止某个协程一直运行导致其他协程饥饿，使用 sysmon 对执行权的抢占式调度。
- Go程序通过调度器来调度Goroutine在内核级线程上执行，但是并不直接绑定os线程M-Machine运行，而是由Goroutine Scheduler中的 P-processor作获取内核线程资源的

### 10. .go struct能不能比较？

因为是强类型语言，所以不同类型的结构不能作比较，但是同一类型的实例值是可以比较的，实例不可以比较，因为是指针类型

### 11. select可以用于什么场景？

监听基于 channel 的 IO 操作

### 12. context包的用途？

协程的上下文传递，通过 context，上层的 goroutine 可以控制下层的 goroutine

### 13. client 如何实现长连接？

通过 net 包中的方法 Dial 建立长连接

### 14. 主协程如何等其余协程完再操作？
- 通过 channel 通信的方式
- 通过 context 控制
> 在go服务器中，对于每个请求的request都是在单独的goroutine中进行的，处理一个request也可能设计多个goroutine之间的交互， 使用context可以使开发者方便的在这些goroutine里传递request相关的数据、取消goroutine的signal或截止日期。


### Go 相关题目部分2

#### 1. 数组和切片的区别是什么，切片的结构大概是什么样的？

- 数组是一个值类型，切片是一个对数组对引用

切片是对数组的一个视图

#### 2. channel 是否是安全的？

channel内部维护了一个互斥锁，来保证线程安全

#### 3. 如何防止数据竞争？

加锁

#### 4. Go 逃逸分析？

- 逃逸分析的好处是为了减少gc的压力，不逃逸的对象分配在栈上，当函数返回时就回收了资源，不需要gc标记清除。
- 逃逸分析完后可以确定哪些变量可以分配在栈上，栈的分配比堆快，性能好(逃逸的局部变量会在堆上分配 ,而没有发生逃逸的则有编译器在栈上分配)。
- 同步消除，如果你定义的对象的方法上有同步锁，但在运行时，却只有一个线程在访问，此时逃逸分析后的机器码，会去掉同步锁运行。


逃逸总结：
栈上分配内存比在堆中分配内存有更高的效率
栈上分配的内存不需要GC处理
堆上分配的内存使用完毕会交给GC处理
逃逸分析目的是决定内分配地址是栈还是堆
逃逸分析在编译阶段完成

#### 5. Go的反射包怎么找到对应的方法 ?
先获取到反射类型对象
getValue := reflect.ValueOf(user)

再通过methodByName 获取到方法

methodValue := getValue.MethodByName("ReflectCallFuncHasArgs")

#### 6. 进程和线程、协程的区别?

- 进程
> 进程是具有一定独立功能的程序关于某个数据集合上的一次运行活动,进程是系统进行资源分配和调度的一个独立单位。每个进程都有自己的独立内存空间，不同进程通过进程间通信来通信。由于进程比较重量，占据独立的内存，所以上下文进程间的切换开销（栈、寄存器、虚拟内存、文件句柄等）比较大，但相对比较稳定安全。

- 线程
> 线程是进程的一个实体,是CPU调度和分派的基本单位,它是比进程更小的能独立运行的基本单位.线程自己基本上不拥有系统资源,只拥有一点在运行中必不可少的资源(如程序计数器,一组寄存器和栈),但是它可与同属一个进程的其他的线程共享进程所拥有的全部资源。线程间通信主要通过共享内存，上下文切换很快，资源开销较少，但相比进程不够稳定容易丢失数据。

- 协程
> 协程是一种用户态的轻量级线程，协程的调度完全由用户控制。协程拥有自己的寄存器上下文和栈。协程调度切换时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈，直接操作栈则基本没有内核切换的开销，可以不加锁的访问全局变量，所以上下文的切换非常快。



#### 7. golang的channel是如何实现的？?

channel有个很重要的sutrct，hchan！ hchan里面有个 mutex lock，数组模拟的fifo队列，生产者游标，消费者游标，生产者等待队列，消费者等待队列等。。。看完这些数据结构有点想法了吧？  当我生产者可以push数据之后，我会顺带把消费者等待队列的某个g给激活，如果我push被阻塞了，那么我自然就存在于等待唤醒的生产者队列。 谁来唤醒？ 当然是某个消费者...简单说，是有mutex锁的，但不会引起协程绑定的线程的上下文切换，只是在go runtime里调度了Goroutine而已。。。


#### 8. mutex和channel作并发控制你喜欢用哪个，哪个快，为什么。?

选择最适合当前业务、问题的那个

- 关注数据的流动，就可以使用channel解决并发问题。
- 不流动的数据，如果存在并发访问，尝试使用sync.Mutex保护数据。
- channel不一定某个并发问题的最优解。
- 不要害怕、拒绝使用mutex，如果mutex是问题的最优解，那就大胆使用。
- 对于大问题，channel plus mutex也许才是更好的方案。

[https://segmentfault.com/a/1190000017890174](https://segmentfault.com/a/1190000017890174)

#### 9. map如何顺序读取？

map不能顺序读取，是因为他是无序的，想要有序读取，首先的解决的问题就是，把ｋｅｙ变为有序，所以可以把key放入切片，对切片进行排序，遍历切片，通过key取值。
```
func getSortMap() {
	lists := map[string]string{
		"Dad": "D",
		"Aad": "A",
		"Cad": "C",
		"Bad": "B",
		"Ead": "E",
	}
	var keys []string
	for _, v := range lists {
		keys = append(keys, v)
	}
	sort.Strings(keys)
	for _, k := range keys {
		if v, ok := lists[k]; ok {
			fmt.Println(v)
		}
	}
}
```

#### 10. 实现set?
```
type inter interface{}

type Set struct {
	m map[inter]bool
	sync.Mutex
}

func New() *Set {
	return &Set{
		m: map[inter]bool{},
	}
}

func (s *Set) Add(item inter) {
	s.Lock()
	defer s.Unlock()
	s.m[item] = true
}
```

#### 11. 下面程序 分别注释掉1 和 2 输出什么？

```
a := [2]int{2, 3}
b := [2]int{2, 3}

// 1
if a == b {
	fmt.Println("equal")
} else {
	fmt.Println("not equal")
}

// 2
if a[:] == b[:] {
	fmt.Println("equal")
} else {
	fmt.Println("not equal")
}
```

#### 12. nsq 相关？

nsqd 
- HTTP 默认监听 4151
- tcp 默认监听 4150

nsqlookupd

```
// nsqd 服务
nsqd --lookupd-tcp-address=127.0.0.1:4160
```

```
// web 后台nsqadmin
nsqadmin --lookupd-http-address=127.0.0.1:4161
```

#### 13. go byte 与 rune 的区别？

rune是用来区分字符值和整数值的


- byte 等同于int8，即一个字节长度，常用来处理ascii字符
- rune 等同于int32，即4个字节长度,常用来处理unicode或utf-8字符

```
str := "你好 world"
fmt.Printf("len(str):%d\n", len(str)) //返回len(str):12
fmt.Printf("len(rune(str)):%d\n", len([]rune(str))) //len(rune(str)):8
```

range 默认使用的是 rune 处理
```
str := "你好 world"
for _,v := range str {
	fmt.Printf("%c\n", v)
}
// output 
你
好
 
w
o
r
l
d
```

#### 14. go fmt.Printf %v %+v %#v的区别?

- %v    只输出所有的值
- %+v 先输出字段类型，再输出该字段的值
- %#v 先输出结构体名字值，再输出结构体（字段类型+字段的值）

```
a := Student{Name: "test"}
fmt.Printf("%v\n", a)
fmt.Printf("%+v\n", a)
fmt.Printf("%#v\n", a)

// output:
{test}
{Name:test}
main.Student{Name:"test"}
```

#### 15. go语言 格式化输出fmt.Printf()使用大全?

【打印】

占位符：

[一般]

%v 相应值的默认格式。在打印结构体时，“加号”标记（%+v）会添加字段名

%#v 相应值的 Go 语法表示

%T 相应值的类型的 Go 语法表示

%% 字面上的百分号，并非值的占位符

[布尔]

%t 单词 true 或 false。

[整数]

%b 二进制表示

%c 相应 Unicode 码点所表示的字符

%d 十进制表示

%o 八进制表示

%q 单引号围绕的字符字面值，由 Go 语法安全地转义

%x 十六进制表示，字母形式为小写 a-f

%X 十六进制表示，字母形式为大写 A-F

%U Unicode 格式：U+1234，等同于 “U+%04X”

[浮点数及其复合构成]

%b 无小数部分的，指数为二的幂的科学计数法，与 strconv.FormatFloat 的 ‘b’ 转换格式一致。例如 -123456p-78

%e 科学计数法，例如 -1234.456e+78

%E 科学计数法，例如 -1234.456E+78

%f 有小数点而无指数，例如 123.456

%g 根据情况选择 %e 或 %f 以产生更紧凑的（无末尾的 0）输出

%G 根据情况选择 %E 或 %f 以产生更紧凑的（无末尾的 0）输出

[字符串与字节切片]

%s 字符串或切片的无解译字节

%q 双引号围绕的字符串，由 Go 语法安全地转义

%x 十六进制，小写字母，每字节两个字符

%X 十六进制，大写字母，每字节两个字符

[指针]

%p 十六进制表示，前缀 0x

### 1. 分布式CAP理论

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

### 2. redis 支持的数据类型？

Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。

### 3. redis 最全的面试题 ?

https://blog.csdn.net/ThinkWon/article/details/103522351

### 4. redis 相关

#### (1). Redis的过期键的删除策略 ?

过期策略通常有以下三种：

- 定时过期：每个设置过期时间的key都需要创建一个定时器，到过期时间就会立即清除。该策略可以立即清除过期的数据，对内存很友好；但是会占用大量的CPU资源去处理过期的数据，从而影响缓存的响应时间和吞吐量。
- 惰性过期：只有当访问一个key时，才会判断该key是否已过期，过期则清除。该策略可以最大化地节省CPU资源，却对内存非常不友好。极端情况可能出现大量的过期key没有再次被访问，从而不会被清除，占用大量内存。
- 定期过期：每隔一定的时间，会扫描一定数量的数据库的expires字典中一定数量的key，并清除其中已过期的key。该策略是前两者的一个折中方案。通过调整定时扫描的时间间隔和每次扫描的限定耗时，可以在不同情况下使得CPU和内存资源达到最优的平衡效果。
(expires字典会保存所有设置了过期时间的key的过期时间数据，其中，key是指向键空间中的某个键的指针，value是该键的毫秒精度的UNIX时间戳表示的过期时间。键空间是指该Redis集群中保存的所有键。)

Redis中同时使用了惰性过期和定期过期两种过期策略。

#### (2). Redis key的过期时间和永久有效分别怎么设置？

EXPIRE和PERSIST命令。

### (3). Redis有三种集群模式

- 主从模式

在主从复制中，数据库分为两类：主数据库(master)和从数据库(slave)。

工作机制：

当slave启动后，主动向master发送SYNC命令。master接收到SYNC命令后在后台保存快照（RDB持久化）和缓存保存快照这段时间的命令，然后将保存的快照文件和缓存的命令发送给slave。slave接收到快照文件和命令后加载快照文件和缓存的执行命令。

复制初始化后，master每次接收到的写命令都会同步发送给slave，保证主从数据一致性。

- Sentinel模式

主从模式的弊端就是不具备高可用性，当master挂掉以后，Redis将不能再对外提供写入操作，因此sentinel应运而生。

sentinel中文含义为哨兵，顾名思义，它的作用就是监控redis集群的运行状况

由sentinel来提供具体的可提供服务的Redis实现，这样当master节点挂掉以后，sentinel就会感知并将新的master节点提供给使用者。

- Cluster模式

sentinel模式基本可以满足一般生产的需求，具备高可用性。但是当数据量过大到一台服务器存放不下的情况时，主从模式或sentinel模式就不能满足需求了，这个时候需要对存储的数据进行分片，将数据存储到多个Redis实例中。cluster模式的出现就是为了解决单机Redis容量有限的问题，将Redis的数据根据一定的规则分配到多台机器。

cluster可以说是sentinel和主从模式的结合体，通过cluster可以实现主从和master重选功能，所以如果配置两个副本三个分片的话，就需要六个Redis实例。因为Redis的数据是根据一定规则分配到cluster的不同机器的，当数据量过大时，可以新增机器进行扩容。

使用集群，只需要将redis配置文件中的cluster-enable配置打开即可。每个集群中至少需要三个主数据库才能正常运行，

cluster集群特点：

* 多个redis节点网络互联，数据共享

* 所有的节点都是一主一从（也可以是一主多从），其中从不提供服务，仅作为备用

* 不支持同时处理多个key（如MSET/MGET），因为redis需要把key均匀分布在各个节点上，
  并发量很高的情况下同时创建key-value会降低性能并导致不可预测的行为
  
* 支持在线增加、删除节点

* 客户端可以连接任何一个主节点进行读写

#### Redis集群最大节点个数是多少？

Redis集群有16384个哈希槽

16384个

#### 缓存异常

##### 缓存雪崩

缓存雪崩是指缓存同一时间大面积的失效，所以，后面的请求都会落到数据库上，造成数据库短时间内承受大量请求而崩掉。

**解决方案**

- 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。
- 使用定时任务定期刷新key 
- 暴力方法 设置key 永不过期
- redis 集群部署时 将热点key 平均分配到不通的redis 服务器上
- 一般并发量不是特别多的时候，使用最多的解决方案是加锁排队。
- 给每一个缓存数据增加相应的缓存标记，记录缓存的是否失效，如果缓存标记失效，则更新数据缓存。

##### 缓存穿透

缓存穿透是指缓存和数据库中都没有的数据，导致所有的请求都落到数据库上，造成数据库短时间内承受大量请求而崩掉。

**解决方案**

- 接口层增加校验，如用户鉴权校验，id做基础校验，id<=0的直接拦截；
- 从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为key-null，缓存有效时间可以设置短点，如30秒（设置太长会导致正常情况也没法使用）。这样可以防止攻击用户反复用同一个id暴力攻击
- 采用布隆过滤器，将所有可能存在的数据哈希到一个足够大的 bitmap 中，一个一定不存在的数据会被这个 bitmap 拦截掉，从而避免了对底层存储系统的查询压力

**缓存击穿**

缓存击穿是指某个非常热点的key,缓存中没有但数据库中有的数据（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力。和缓存雪崩不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。

**解决方案**

- 设置热点数据永远不过期。
- 加互斥锁，单进程访问，其他进程从缓存中获取

##### 缓存预热

缓存预热就是系统上线后，将相关的缓存数据直接加载到缓存系统。这样就可以避免在用户请求的时候，先查询数据库，然后再将数据缓存的问题！用户直接查询事先被预热的缓存数据！

**解决方案**

- 直接写个缓存刷新页面，上线时手工操作一下；

- 数据量不大，可以在项目启动的时候自动进行加载；

- 定时刷新缓存；

##### 缓存降级

当访问量剧增、服务出现问题（如响应时间慢或不响应）或非核心服务影响到核心流程的性能时，仍然需要保证服务还是可用的，即使是有损服务。系统可以根据一些关键数据进行自动降级，也可以配置开关实现人工降级。

缓存降级的最终目的是保证核心服务可用，即使是有损的。而且有些服务是无法降级的（如加入购物车、结算）。

在进行降级之前要对系统进行梳理，看看系统是不是可以丢卒保帅；从而梳理出哪些必须誓死保护，哪些可降级；比如可以参考日志级别设置预案：

- 一般：比如有些服务偶尔因为网络抖动或者服务正在上线而超时，可以自动降级；

- 警告：有些服务在一段时间内成功率有波动（如在95~100%之间），可以自动降级或人工降级，并发送告警；

- 错误：比如可用率低于90%，或者数据库连接池被打爆了，或者访问量突然猛增到系统能承受的最大阀值，此时可以根据情况自动降级或者人工降级；

- 严重错误：比如因为特殊原因数据错误了，此时需要紧急人工降级。

#### 5. Redis与Memcached的区别

两者都是非关系型内存键值数据库，现在公司一般都是用 Redis 来实现缓存，而且 Redis 自身也越来越强大了！Redis 与 Memcached 主要有以下不同：

Redis与Memcached的区别：

1、类型

Redis是一个开源的内存数据结构存储系统，用作数据库，缓存和消息代理。

Memcached是一个免费的开源高性能分布式内存对象缓存系统，它通过减少数据库负载来加速动态Web应用程序。

2、数据结构

Redis支持字符串，散列，列表，集合，有序集，位图，超级日志和空间索引；而Memcached支持字符串和整数。

3、执行速度

Memcached的读写速度高于Redis。

4、复制

Memcached不支持复制。而，Redis支持主从复制，允许从属Redis服务器成为主服务器的精确副本；来自任何Redis服务器的数据都可以复制到任意数量的从属服务器。

5、密钥长度

Redis的密钥长度最大为2GB，而Memcached的密钥长度最大为250字节。

6、线程

Redis是单线程的；而，Memcached是多线程的。


#### 6. MongoDB Memcache Redis MySQL?

**MongoDB**

使用磁盘的非关系型数据库

**Memcache**

- 使用内存的非关系型数据库
- Memcached还可用于缓存其他东西，例如图片、视频等等；

**redis**
- 使用内存的非关系型数据库，有更多的数据结构，如 String Set hashtable 等
- 使用场景：热数据、计数器、队列、排行榜（有序集合zset）、分布式锁等

**MySQL**

- 使用磁盘的关系型数据库

#### 7。 merge和rebase的区别？
- 采用merge和rebase后，git log的区别，merge命令不会保留merge的分支的commit：
- 处理冲突的方式：
    - （一股脑）使用merge命令合并分支，解决完冲突，执行git add .和git commit -m'fix conflict'。这个时候会产生一个commit。
    - （交互式）使用rebase命令合并分支，解决完冲突，执行git add .和git rebase --continue，不会产生额外的commit。这样的好处是，‘干净’，分支上不会有无意义的解决分支的commit；坏处，如果合并的分支中存在多个commit，需要重复处理多次冲突。

[参考](https://www.cnblogs.com/xueweihan/p/5743327.html)

#### 8. git merge 和 git merge --no-ff的区别？

- 冲突的时候，git merge 会自动产生一个commit。
- git merge --no-ff 在没有冲突的情况下也自动生成一个commit

[参考](https://www.cnblogs.com/xueweihan/p/5743327.html)

#### 9. http get跟head?
HEAD和GET本质是一样的，区别在于HEAD不含有呈现数据，而仅仅是HTTP头信息。有的人可能觉得这个方法没什么用，其实不是这样的。想象一个业务情景：欲判断某个资源是否存在，我们通常使用GET，但这里用HEAD则意义更加明确。

#### 10. 基本排序，哪些是稳定的?
堆排序、快速排序、希尔排序、直接选择排序是不稳定的排序算法，而基数排序、冒泡排序、直接插入排序、折半插入排序、归并排序是稳定的排序算法。

#### 11. TCP第四次挥手为什么要等待2MSL?
- 为了保证客户端发送的最后一个ACK报文段能够到达服务器。因为这个ACK有可能丢失，从而导致处在LAST-ACK状态的服务器收不到对FIN-ACK的确认报文。服务器会超时重传这个FIN-ACK，接着客户端再重传一次确认，重新启动时间等待计时器。最后客户端和服务器都能正常的关闭。假设客户端不等待2MSL，而是在发送完ACK之后直接释放关闭，一但这个ACK丢失的话，服务器就无法正常的进入关闭连接状态。
- 防止已失效的报文段.客户端在发送最后一个ACK之后，再经过经过2MSL，就可以使本链接持续时间内所产生的所有报文段都从网络中消失。从保证在关闭连接后不会有还在网络中滞留的报文段去骚扰服务器。

#### 12. HTTP2.0和HTTP1.X相比的新特性?

- 新的二进制格式（Binary Format），HTTP1.x的解析是基于文本。基于文本协议的格式解析存在天然缺陷，文本的表现形式有多样性，要做到健壮性考虑的场景必然很多，二进制则不同，只认0和1的组合。基于这种考虑HTTP2.0的协议解析决定采用二进制格式，实现方便且健壮。

- 多路复用（MultiPlexing），即连接共享，即每一个request都是是用作连接共享机制的。一个request对应一个id，这样一个连接上可以有多个request，每个连接的request可以随机的混杂在一起，接收方可以根据request的 id将request再归属到各自不同的服务端请求里面。

- header压缩，如上文中所言，对前面提到过HTTP1.x的header带有大量信息，而且每次都要重复发送，HTTP2.0使用encoder来减少需要传输的header大小，通讯双方各自cache一份header fields表，既避免了重复header的传输，又减小了需要传输的大小。

- 服务端推送（server push），同SPDY一样，HTTP2.0也具有server push功能。

#### 13. TCP协议的滑动窗口具体是怎样控制流量的？

TCP滑动窗口分为接受窗口，发送窗口

滑动窗口协议是传输层进行流控的一种措施，接收方通过通告发送方自己的窗口大小，从而控制发送方的发送速度，从而达到防止发送方发送速度过快而导致自己被淹没的目的。

#### 14. TCP协议的滑动窗口具体是怎样控制流量的？

#### 15. tcp四次挥手?

由于TCP连接是全双工的，因此每个方向都必须单独进行关闭。这个原则是当一方完成它的数据发送任务后就能发送一个FIN来终止这个方向的连接。收到一个 FIN只意味着这一方向上没有数据流动，一个TCP连接在收到一个FIN后仍能发送数据。首先进行关闭的一方将执行主动关闭，而另一方执行被动关闭。

（1）客户端A发送一个FIN，用来关闭客户A到服务器B的数据传送（报文段4）。

(2）服务器B收到这个FIN，它发回一个ACK，确认序号为收到的序号加1（报文段5）。和SYN一样，一个FIN将占用一个序号。

(3) 服务器B关闭与客户端A的连接，发送一个FIN给客户端A（报文段6）。

（4）客户端A发回ACK报文确认，并将确认序号设置为收到序号加1（报文段7）

#### 16. 为什么连接的时候是三次握手，关闭的时候却是四次握手？

因为当Server端收到Client端的SYN连接请求报文后，可以直接发送SYN+ACK报文。其中ACK报文是用来应答的，SYN报文是用来同步的。但是关闭连接时，当Server端收到FIN报文时，很可能并不会立即关闭SOCKET，所以只能先回复一个ACK报文，告诉Client端，"你发的FIN报文我收到了"。只有等到我Server端所有的报文都发送完了，我才能发送FIN报文，因此不能一起发送。故需要四步握手。

#### 17. rabbitMQ 四种消息类型？

- 1. fanout（订阅）
- 2. direct（路由）
- 3. topic（主题）
- 4. header

#### 18. Elasticsearch中text与keyword的区别?
text类型
- 支持分词，全文检索,支持模糊、精确查询,不支持聚合,排序操作;
- test类型的最大支持的字符长度无限制,适合大字段存储；

使用场景：

    存储全文搜索数据, 例如: 邮箱内容、地址、代码块、博客文章内容等。
    默认结合standard analyzer(标准解析器)对文本进行分词、倒排索引。
    默认结合标准分析器进行词命中、词频相关度打分。

keyword
- 不进行分词，直接索引,支持模糊、支持精确匹配，支持聚合、排序操作。
- keyword类型的最大支持的长度为——32766个UTF-8类型的字符,可以通过设置ignore_above指定自持字符长度，超过给定长度后的数据将不被索引，无法通过term精确匹配检索返回结果。

1:不进行分词，直接索引,支持模糊、支持精确匹配，支持聚合、排序操作。
2:keyword类型的最大支持的长度为——32766个UTF-8类型的字符,可以通过设置ignore_above指定自持字符长度，超过给定长度后的数据将不被索引，无法通过term精确匹配检索返回结果。

使用场景：

    存储邮箱号码、url、name、title，手机号码、主机名、状态码、邮政编码、标签、年龄、性别等数据。
    用于筛选数据(例如: select * from x where status='open')、排序、聚合(统计)。
    直接将完整的文本保存到倒排索引中。

#### 19. Opcache 是什么?

Opcache是字节码缓存，PHP在被编译的时候，首先会把php代码转换为字节码，字节码然后被执行。

#### 20. 常见的web攻击方式?

- SQL注入------常见的安全性问题。

解决方案：前端页面需要校验用户的输入数据（限制用户输入的类型、范围、格式、长度），不能只靠后端去校验用户数据。一来可以提高后端处理的效率，二来可以提高后端数据的安全。

后端不要动态sql语句，使用存储过程查询语句。限制用户访问数据库权限。后端接受前端的数据时要过滤一些特殊字符（如：“--”等字符）

后端如果出现异常的话，要使用自定义错误页，防止用户通过服务器默认的错误页面找到服务器漏洞。

- XSS攻击------相对复杂的安全性问题

攻击方式：基于DOM的XSS即通过浏览器来直接运行js脚本，无须提交服务器，从客户端的代码引起的。

解决方案：后端输出页面的时候需要进行转换html实体。

- CSRF攻击------比xss攻击更危险的安全性问题

解决方案：验证 HTTP Referer 字段，给用户分配token。

- cc 攻击 ---- 模拟多个用户不停的访问，CC攻击可以归为DDoS攻击的一种。他们之间的原理都是一样的，即发送大量的请求数据来导致服务器拒绝，是一种连接攻击。

解决方法：

1. 使用Session 来执行访问计数器
2. 将网站生成静态页面
3. 限制频繁 发起大量请求的IP
4. 购买专业的抵御DDos攻击工具

#### 21. rabbitmq学习总结 RabbitMQ五种交换机类型，六种队列模式

- 五种交换机？
1. Direct exchange.
> a、会根据routingKey 完全匹配成功后才会消费。比如：如果生产一条消息 “我是中国人”，发送到交换机的时候绑定了路由键是：“中国”，则如果要消费的话只有匹配了路由键是“中国”的才能消费。（可以比喻为交换机是 “地球”，路由键是“国家—中国”，消息是“人”，这个消息的身份证是哪个国家的“路由键”那就是只能在这个国家享有权益。）<br>
b、如果都消费同一个routingKey的话，多个消费者谁先消费到就是谁的

2. Topic exchange.
> a、该模式不仅仅需要exchange和queue绑定还需要和路由键routingKey关联.<br>
b、模糊匹配模式，比如：两个路由键 animal.dog ， animal.dog.eat。如果该队列不仅仅对“dog”的消息感兴趣，同时还对与“dog”相关的消息感兴趣就可以使用topic模式，animal.dog.# 。（支持# 0或多词模糊匹配，*一个词匹配）<br>
应用：订阅任务，信息分类更新业务

3. Fanout exchange.
> a、该模式不需要路由键routingKey<br>
b、该模式只需要将queue和exchange绑定就好。一个exchange可以绑定N多个queue，每一个queue都会得到同样的消息<br>
c、一个queue可以和多个exchange绑定，消费来自不同的exchange的消息<br>
d、转发消息最快<br>
应用：群聊功能、全网消息推送功能

4. Headers exchange.
>  a、无路由键routingKey的概念<br>
b、是以 header和message中的消息匹配上才能消费

5. System exchange
> 其实就是系统默认和direct模式没区别，只不过不需要定义exchange名字而已。

- 二、六种队列模式

1. hello word 模式（单发送，单接收模式）
2. work模式（工厂模式）
> 一个发送端，多个接收端，支持持久化durable，公平消费原则basicQos，消息的可靠性ack=true，false<br>
a、消息队列durable——true持久化
<br>b、在消费的时候，由channel.basicAck()在消息处理完成后发送消息false确认单条或true批量确认。<br>
c、使用了channel.basicQos(1)保证在接收端一个消息没有处理完时不会接收另一个消息，即接收端发送了ack后才会接收下一个消息。在这种情况下发送端会尝试把消息发送给下一个空闲的的接收端。

3. Publish/Subscribe
> 一个生产者发送消息到多个消费者

4. routing模式
> 发送消息到交换机并且要指定路由key ，消费者需要匹配对路由key才能消费

5. topic模式
> 发送消息到交换机并和路由key进行绑定，但该路由key支持模糊匹配，是指成为“一类”消息

6. RPC模式

#### 22. 布隆过滤器介绍

是一个很长的二进制向量，其实就是一个二进制数组，由0和1表示的，布隆过滤器的主要作用就是判断一个数据存不存在这个数组里面，存在是1，不存在就是0；用二进制0 1 来表示数据存不存在；对于一个数据，会经过（多次）一般三次哈希函数将数据存到布隆过滤器里面。主要步骤 有两个 一是经过多次哈希函数处理，二 是二进制的数据都是1 才能证明这个数据存在。删除数据比较麻烦，会将哈希过后相同位置的数据一块删除掉

误差率越小 占用的空间越大，经过的hash 函数越多

**优点：**
1. 由二进制数组组成的一个数据，占用的内存是非常小的
2. 插入和查询的速度是非常快的，因为是计算数据的哈希值，再由哈希值映射到这个数组的下标，时间复杂度为O(k) ,k 为哈希函数个数
3. 安全性好，数据只有0 1本身不存储原始数据

**缺点：**
1. 很难做删除操作
2. 存在误判 本身不存在集合中，但是经过一系列哈希运算之后，得到数据存在这个集合中

http://imhuchao.com/1271.html

#### 23. 单点登录的实现方案：

1. 利用共享cookie 实现
> (1). 共享cookie 的方法， 客户端对cookie 进行解析，将token 解析出来,之后的每次请求都将token 带上<br>
(2). 多个域名共享cookie, 在些客户端的时候设置Cookie 的domain<br>
(3). 将token 保存在SessionStorage 中，（不依赖cookie就没有跨域问题）
2. 增加业务sso 登录系统实现
3. 通过认证中心 oAuth 实现

#### 24. http和https有什么区别?

http协议和https协议的区别：传输信息安全性不同、连接方式不同、端口不同、证书申请方式不同

一、传输信息安全性不同
1. http协议：是超文本传输协议，信息是明文传输。如果攻击者截取了Web浏览器和网站服务器之间的传输报文，就可以直接读懂其中的信息。
2. https协议：是具有安全性的ssl加密传输协议，为浏览器和服务器之间的通信加密，确保数据传输的安全。

http:

http->TCP->IP

https:

HTTP->SSL or TLS->TCP->IP

二、连接方式不同
1. http协议：http的连接很简单，是无状态的。
2. https协议：是由SSL＋HTTP协议构建的可进行加密传输、身份认证的网络协议。

三、端口不同
1. http协议：使用的端口是80。
2. https协议：使用的端口是443．

四、证书申请方式不同
1. http协议：免费申请。
2. https协议：需要到ca申请证书，一般免费证书很少，需要交费。

### 25. http 状态码 302 和 304？
- 301 表示永久重定向（301 moved permanently），表示请求的资源分配了新url，以后应使用新url。
- 302 重定向 当响应码302 时，表示服务器要求浏览器重新发一个请求，服务器会发送一个响应头Location，他指向了新请求的url地址
- 304 是对客户端已经有缓存对的情况下服务端的一种响应。常见与静态文件，自从上次请求后，内容未修改过，可以直接使用客户端缓存的数据。客户端有缓存的文档并发送附带条件的请求时（if-matched,if-modified-since,if-none-match,if-range,if-unmodified-since任一个）服务器端允许请求访问资源，但因发生请求未满足条件的情况后，直接返回304Modified（服务器端资源未改变，可直接使用客户端未过期的缓存）。

### 26. 雪花算法：1 个bit 位 不用 + 41 位时间戳 + 10位机器ID + 12位序列号（自增） 转换成长度为18位的长整型

#### 相关名词 ：限流降级


**[其他面试题](https://studygolang.com/articles/17796?spm=a2c6h.12873639.0.0.42156786jfSm5s)** 

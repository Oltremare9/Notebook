## Redis

### <u>Redis为什么快</u>

> 纯内存操作
>
> 单线程操作，减少了频繁的上下文切换
>
> 采用了非阻塞IO多路复用机制

### <u>什么是多路复用机制</u>

> 小名在 A 城开了一家快餐店店，负责同城快餐服务。小明因为资金限制，雇佣了一批配送员，然后小曲发现资金不够了，只够买一辆车送快递。
>
> 小明只雇佣一个配送员。当客户下单，小明按送达地点标注好，依次放在一个地方。最后，让配送员依次开着车去送，送好了就回来拿下一个。
>
> - 每个配送员→每个线程
> - 每个订单→每个 Socket(I/O 流)
> - 订单的送达地点→Socket 的不同状态
> - 客户送餐请求→来自客户端的请求
> - 明曲的经营方式→服务端运行的代码
> - 一辆车→CPU 的核数
>
> 只有单个线程(一个配送员)，通过跟踪每个 I/O 流的状态(每个配送员的送达地点)，来管理多个 I/O 流。
>
> Redis-client 在操作的时候，会产生具有不同事件类型的 Socket。在服务端，有一段 I/O 多路复用程序，将其置入**队列**之中。然后，**文件事件分派器**，**依次去队列中取**，转发到不同的事件处理器中。

### <u>多路复用poll水平触发和边缘触发的区别</u>

### <u>Redis和Memcached的区别</u>

两者相同点 

> 两者都是基于内存的数据存储系统
>
> 本质上都是一个内存的key-value存储系统

两者的不同点

##### 1数据操作不同

> Memcached仅支持key-value的数据结构 不支持枚举，不支持持久化和复制
>
> Redis支持的数据结构更加丰富 支持list set zset hash string 并且通过了持久化和复制的功能

##### 2内存管理机制的不同

> Redis中，并不是所有的数据都一直存储在内存中，可以通过lru的机制，将一些很久没用的value交换到磁盘，并且在内存中进行删除
>
> Memcached默认使用Slab Allocation机制管理内存 思想是按照预先规定的大小，将分配的内存分割成特定长度的块以存储相应长度的key-value数据记录，以完全解决内存碎片问题。

##### 3性能不同

> Redis只能使用单核 平均每一个核上Redis在存储小数据时性能更高
>
> Memcached可以使用多核 存储100k以上的数据 性能更好

##### 4集群管理不同

> Memcached是全内存的数据缓冲系统，Redis虽然支持数据的持久化，但是全内存毕竟才是其高性能的本质。作为基于内存的存储系统来说，机器物理内存的大小就是系统能够容纳的最大数据量。如果需要处理的数据量超过了单台机器的物理内存大小，就需要构建分布式集群来扩展存储能力。
>
> Memcached本身不支持分布式，只能通过在客户端通过一致性hash的方式实现分布式存储。
>
> Redis则偏向于在服务端构建分布式存储。



### <u>为什么要使用Redis</u>

#### 性能问题

> 对于一些执行耗时很久，结果不频繁变动的SQL，适合将运行结果放入缓存，以减少对于MySQL的访问
>
> **例如 一些静态不会进行变更的一些数据 在秒杀时可以将其放在redis**

#### 并发问题

> 高并发的情况下，所有的请求直接访问数据库会极容易出现问题，所以需要用redis做一个缓冲操作
>
> **例如 redis实现的消息队列的形式**



### <u>Redis的问题以及解决方案</u>

#### *缓存和数据库双写一致性问题*

#### 	*缓存雪崩问题*

> 缓存雪崩，是指在某一个时间段，缓存集中过期失效。
>
> ​	解决方案：将设置时间时加入随机量

#### 	*缓存击穿问题*

> 是指一个key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，就像在一个屏障上凿开了一个洞。
>
> ​	让缓存永不过期

#### 	*缓存穿透*

> 是指查询一个数据库一定不存在的数据。正常的使用缓存流程大致是，数据查询先进行缓存查询，如果key不存在或者key已经过期，再对数据库进行查询，并把查询到的对象，放进缓存。如果数据库查询对象为空，则不放进缓存。
>
> ​	**互斥锁**
>
> ​		缓存失效的时候，先去获得锁，得到锁了，再去请求数据库。没得到锁，则休眠一段时间重试。
>
> ​	**布隆过滤器**
>
> ​	**缓存空对象**
>
> ​		即使结果为空也将其缓存下来，设置一个较短的过期时间

#### *缓存的并发竞争问题*

#### 

### <u>分布式锁一定要用Redis吗 分布式锁的实现</u>

#### *数据库乐观锁*

#### *基于Redis的分布式锁*

#### *基于ZooKeeper的分布式锁*



### <u>Redis的淘汰策略</u>

> redis使用的是定期删除+惰性删除的形式
>
> 为什么不适用定时删除
>
> ​	因为占用cpu资源
>
> 如何工作
>
> ​	定期删除，Redis 默认每个 100ms 检查，有过期 Key 则删除。需要说明的是，Redis 不是每个 100ms 将所有的 Key 检查一次，而是随机抽取进行检查。如果只采用定期删除策略，会导致很多 Key 到时间没有删除。于是，惰性删除派上用场。

- noeviction：当**内存不足**以容纳新写入数据时，新写入操作**会报错**。
- allkeys-lru：当**内存不足**以容纳新写入数据时，在键空间中，**移除最近最少使用的 Key**。（推荐使用，目前项目在用这种）(最近最久使用算法)
- allkeys-random：当**内存不足**以容纳新写入数据时，在键空间中，**随机移除某个 Key**。（应该也没人用吧，你不删最少使用 Key，去随机删）
- volatile-lru：当**内存不足**以容纳新写入数据时，在**设置了过期时间的键空间中，移除最近最少使用的 Key**。这种情况一般是把 Redis 既当缓存，又做持久化存储的时候才用。（不推荐）
- volatile-random：当**内存不足**以容纳新写入数据时，在**设置了过期时间的键空间中，随机移除某个 Key**。（依然不推荐）
- volatile-ttl：当**内存不足**以容纳新写入数据时，在设置了过期时间的键空间中，**有更早过期时间的 Key 优先移除**。（不推荐）

### <u>Redis内部的数据结构(使用场景)以及底层实现</u>

#### string

> ​	动态字符串 记录长度  空闲数量 以及用数组来保存字符串	

#### map

> ​	数组+链表 进行rehash的优化 存储两个数组结构 进行部分的扩容，当完全结束后再释放空间

#### 链表

> 双向链表的基础上 拓展了头尾节点 记录元素个数

#### 集合Set

> 

#### 有序集合ZSet

> ​	跳表（其本质和平衡树类似 但是相比而言 对于插入的调整较少 并且范围查询更方便）

[知乎解答]: https://zhuanlan.zhihu.com/p/50392209

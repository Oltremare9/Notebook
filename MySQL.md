# MySQL

### 事务

> 事务具有四个特征：原子性（ Atomicity ）、一致性（ Consistency ）、隔离性（ Isolation ）和持续性（ Durability ）。这四个特性简称为 ACID 特性。
>
> 1 、原子性。事务是数据库的逻辑工作单位，事务中包含的各操作要么都做，要么都不做
>
> 2 、一致性。事 务执行的结果必须是使数据库从一个一致性状态变到另一个一致性状态。因此当数据库只包含成功事务提交的结果时，就说数据库处于一致性状态。如果数据库系统 运行中发生故障，有些事务尚未完成就被迫中断，这些未完成事务对数据库所做的修改有一部分已写入物理数据库，这时数据库就处于一种不正确的状态，或者说是 不一致的状态。
>
> 3 、隔离性。一个事务的执行不能其它事务干扰。即一个事务内部的操作及使用的数据对其它并发事务是隔离的，并发执行的各个事务之间不能互相干扰。
>
> 4 、持续性。也称永久性，指一个事务一旦提交，它对数据库中的数据的改变就应该是永久性的。接下来的其它操作或故障不应该对其执行结果有任何影响。

### 隔离级别

> #### 读未提交
>
> 在该隔离级别，所有事务都可以看到其他未提交事务的执行结果。本隔离级别很少用于实际应用，因为它的性能也不比其他级别好多少。读取未提交的数据，也被称之为脏读（Dirty Read）。
>
> #### 读已提交
>
> 这是大多数数据库系统的默认隔离级别（但不是MySQL默认的）。它满足了隔离的简单定义：一个事务只能看见已经提交事务所做的改变。这种隔离级别 也支持所谓的不可重复读（Nonrepeatable Read），因为同一事务的其他实例在该实例处理其间可能会有新的commit，所以同一select可能返回不同结果。
>
> #### 可重复读
>
> 这是MySQL的**默认事务隔离级别**，它确保同一事务的多个实例在并发读取数据时，会看到同样的数据行。不过理论上，这会导致另一个棘手的问题：幻读 （Phantom Read）。简单的说，幻读指当用户读取某一范围的数据行时，另一个事务又在该范围内插入了新行，当用户再读取该范围的数据行时，会发现有新的“幻影” 行。InnoDB和Falcon存储引擎通过多版本并发控制（MVCC，Multiversion Concurrency Control）机制解决了该问题。
>
> #### 串行化
>
> 这是最高的隔离级别，它通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。简言之，它是在每个读的数据行上加上共享锁。在这个级别，可能导致大量的超时现象和锁竞争。

### MVCC简介

> MVCC多版本并发控制(Multi-Version Concurrency Control)是MySQL中**基于乐观锁理论实现隔离级别**的方式，用于实现读已提交和可重复读取隔离级别。
>
> 这项技术使得InnoDB的事务隔离级别下执行一致性读操作有了保证，换言之，就是**为了查询一些正在被另一个事务更新的行**，并且可以看到它们被更新之前的值。这是一个可以用来增强并发性的强大的技术，因为这样的一来的话查询就**不用等待另一个事务释放锁**。
>
> 核心是在每一行隐藏了两个字段，用于表示版本号。
>
> - **SELECT**
>   - **读取创建版本小于或等于当前事务版本号，并且删除版本为空或大于当前事务版本号的记录。这样可以保证在读取之前记录是存在的。**
> - **INSERT**
>   - **将当前事务的版本号保存至行的创建版本号**
> - **UPDATE**
>   - **新插入一行，并以当前事务的版本号作为新行的创建版本号，同时将原记录行的删除版本号设置为当前事务版本号**
>   - **修改的时候一定不是快照读，而是当前读。**
> - **DELETE**
>   - **将当前事务的版本号保存至行的删除版本号**
> - 只有普通的SELECT才是快照读，其它诸如UPDATE、删除都是当前读。
> - 修改的时候一定不是快照读，而是当前读。

### 各种隔离级别的实现 使用的协议（一致性读+锁）

> **一致性读**：InnoDB用多版本来提供查询数据库在**某个时间点的快照**。一致性读不会给它所访问的表加任何形式的锁，因此其它事务可以同时并发的修改它们。
>
> 可重复读：
>
> ​	如果隔离级别是REPEATABLE READ，那么在同一个事务中的所有一致性读都读的是事务中第一个这样的读读到的快照；
>
> 读已提交：
>
> ​	如果是READ COMMITTED，那么一个事务中的每一个一致性读都会读到它自己刷新的快照版本。
>
> 幻读：
>
> ​	利用Gap Locks间隙锁和Next-Key可以阻止其它事务在锁定区间内插入数据，因此解决了幻读问题

##### 通过MVCC实现一致性非锁定读，这就有保证在同一个事务中多次读取相同的数据返回的结果是一样的，解决了不可重复读的问题

##### 利用Gap Locks和Next-Key可以阻止其它事务在锁定区间内插入数据，因此解决了幻读问题



### 索引的下推和覆盖

> > 索引的下推
> >
> > ​	可以在索引遍历过程中，**对索引中包含的字段先做判断，过滤掉不符合条件的记录，减少回表字数**。
>
> ```mysql
> select * from tuser 
> where name like '张 %' and age=10 and ismale=1;
> ```
>
> 该语句在搜索索引树的时候，只能匹配到名字第一个字是‘张’的记录（即记录ID3）接下来是怎么处理的呢？当然就是从ID3开始，逐个回表，到主键索引上找出相应的记录，再比对age和ismale这两个字段的值是否符合。
>
> 索引下推优化后，可以在索引遍历过程中，**对索引中包含的字段先做判断，过滤掉不符合条件的记录，减少回表字数**。
>
> ![img](https://upload-images.jianshu.io/upload_images/6271376-e4e98a8af8fc9ca8.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1142/format/webp)
>
> ![](https://upload-images.jianshu.io/upload_images/6271376-53f9161adfddeb10.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1142/format/webp)
>
> > 索引的覆盖
> >
> > ​	只需要在一棵索引树上就能获取SQL所需的所有列数据，无需回表，速度更快。
> >
> > explain的输出结果Extra字段为Using index时，能够触发索引覆盖。
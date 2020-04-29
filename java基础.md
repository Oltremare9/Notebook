# Java基础

### <u>JDK1.8新特性</u>

#### 			Lambda表达式

> ​	简化匿名内部类的代码量

#### 			函数式接口

> 简单来说就是只定义了一个抽象方法的接口（Object类的public方法除外），就是函数式接口，并且还提供了注解：@FunctionalInterface
>
> 通过提供的四大函数式接口 可以简化lamda，不用手动创建一个新的函数式接口

#### 		方法引用和构造器调用

#### 		Stream API

#### 		接口中的默认方法和静态方法

> 在接口中可以使用default和static关键字来修饰接口中定义的普通方法

```java
public interface Interface {
    default  String getName(){
        return "zhangsan";
    }

    static String getName2(){
        return "zhangsan";
    }
}

```

> 可以避免在1.8中新增的接口方法 需要在1.7中全部添加实现，可以通过默认实现的方式
>
> 当一个类继承父类又实现接口时，若后两者方法名相同，则优先继承父类中的同名方法，即“类优先”，如果实现两个同名方法的接口，则要求实现类必须手动声明默认实现哪个接口中的方法。

#### 		新时间日期API

> 新的日期API LocalDate | LocalTime | LocalDateTime
>
> 新的日期API都是不可变的，更使用于多线程的使用环境中
>
> >  * 之前使用的java.util.Date月份从0开始，我们一般会+1使用，很不方便，java.time.LocalDate月份和星期都改成了enum
> >  * java.util.Date和SimpleDateFormat都不是线程安全的，而LocalDate和LocalTime和最基本的String一样，是不变类型，不但线程安全，而且不能修改。
> >  * java.util.Date是一个“万能接口”，它包含日期、时间，还有毫秒数，更加明确需求取舍
> >  * 新接口更好用的原因是考虑到了日期时间的操作，经常发生往前推或往后推几天的情况。用java.util.Date配合Calendar要写好多代码，而且一般的开发人员还不一定能写对。

[新特性]: https://blog.csdn.net/qq_29411737/article/details/80835658





### <u>java中的流</u>

#### 	字节流

> ​	以字节为导向的 stream------InputStream/OutputStream

#### 	字符流

> ​	以字符为导向的 stream Reader/Writer

#### 	流没有释放会导致的问题

> ​	长时间不进行释放会一直占用内存 长时间可能导致OOM



### <u>Tomcat类加载</u>

> https://www.cnblogs.com/aspirant/p/8991830.html



### <u>类加载如何打破双亲委派</u>

> 重写classload方法
>
> 重写findclass方法 实现自定义的类加载方式

### <u>OOM的场景 查询工具</u>

#### 场景

> 1堆空间设置不合理 将其设置调大
>
> 2永久代空间太小
>
> 3gc时对象过多，导致内存溢出，可以调整gc的策略，不要占满后再gc
>
> 4本地线程空间溢出 线程栈过大？
>
> 5分配了一个大于堆大小的数组（连续空间）  是否数组过大 或调大堆
>
> 6由于从native堆中分配内存失败，并且堆内存可能接近耗尽
>
> 线程池中等待队列无限制扩张

#### 工具

> **jstat**
>
> 可以使用jstat查看程序运行的实时情况，包括堆内存信息和垃圾回收信息，常用来查看程序的垃圾回收情况
>
> > jstat -gc pid
>
> Jprofiler
>
> > 插件 用于看内存是一个内存工具。可以直观的看到各个对象在堆内存中所占空间大小 类实例数量 对象引用关系



### <u>JDK提供的工具</u>



### <u>异常分为两种 有什么不同</u>

#### error

> 是指 java 运行时系统的内部错误和资源耗尽错误。应用程序不会抛出该类对象。如果出现了这样的错误，除了告知用户，剩下的就是尽力使程序安全的终止。

#### exception

> 运行时异常RuntimeException
>
> > 是那些可能在 Java 虚拟机正常运行期间抛出的异常的超类。 如果出现 RuntimeException，那么一定是程序员代码书写导致的错误.
> >
> > 例如：空指针异常 下标越界
>
> 检查异常 CheckedException
>
> > 一般是外部错误，这种异常都发生在编译阶段，Java 编译器会强
> > 制程序去捕获此类异常，即会出现要求你把这段可能出现异常的程序进行 try catch
> >
> > 例如：IOException SQLException

### java集合

#### list

> ArrayList
>
> > **优点:** 底层数据结构是数组，查询快，增删慢。
> > **缺点:** 线程不安全，效率高
>
> Vector
>
> > **优点:** 底层数据结构是数组，查询快，增删慢。
> > **缺点:** 线程安全，效率低
>
> LinkedList
>
> > **优点:** 底层数据结构是链表，查询慢，增删快。
> > **缺点:** 线程不安全，效率高

#### set

> HashSet
>
> > 底层数据结构是哈希表。(无序,唯一)
> > 如何来保证元素唯一性?
> > 1.依赖两个方法：hashCode()和equals()
>
> LinkedHashSet
>
> > 底层数据结构是链表和哈希表。(FIFO插入有序,唯一)
> > 1.由链表保证元素有序
> > 2.由哈希表保证元素唯一
>
> TreeSet
>
> > 底层数据结构是红黑树。(唯一，有序)
> > \1. 如何保证元素排序的呢?
> > 自然排序
> > 比较器排序
> > 2.如何保证元素唯一性的呢?
> > 根据比较的返回值是否是0来决定

#### map

> Hashtable
>
> > 线程安全 使用了synchronized
>
> LinkedHashMap
>
> > 在hashmap的基础上增加了有序
>
> ConcurrentHashMap
>
> > 1.7中使用分段锁 1.8放弃了分段锁的概念，而是直接用Node数组+链表+红黑树的数据结构来实现，如下图所示，并发控制使用Synchronized和CAS来操作，每一个Node节点都是用volatile修饰的，整个看起来就像是优化过且线程安全的HashMap。
>
> HashMap
>
> > Hashmap是线程不安全的 1.7数组+链表 1.8数组+链表/红黑树


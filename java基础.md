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


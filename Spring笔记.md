## Spring笔记

### 自动装配

#### 自动装配的两种方式

##### byName 

> 通过bean的id和set方法中的后缀匹配进行自动装配
>
> 例如：setAddress方法 能匹配bean配置文件中id为address的bean

##### byType

> 根据bean的类型
>
> 但是如果IOC容器中有一个以上的类型匹配则会报异常

#### 自动装配缺点

> 所有属性都要自动装配 不够灵活、
>
> byType和byName不能混用



### bean的关系

#### bean的继承

> 使用parent属性指定继承哪个bean
>
> 父类bean可以设置为abstract类似于抽象类使得其作为模板不可以被实例化
>
> 若某一个bean的class属性没有指定，则该bean一定是一个抽象bean

#### bean的依赖关系

> depends-on 表示bean之间的依赖



### bean的作用域 scope

#### 单例的singleton

> 在整个声明周期内只创建这一个bean

#### 原型的prototype

> 容器创建时，不创建bean的实例，每次请求时创建一个新的Bean

#### Session

#### Request



### SpEL动态进行赋值

> 通过#{}的形式可以进行对属性赋值 包括别的bean 字面值 别的bean的属性值 方法返回值



### IOC容器中的Bean的生命周期

> 通过构造器或工厂方法创建Bean实例				**构造方法**
>
> 为Bean的属性设置 值和其他Bean的引用			**set方法**
>
> 调用bean后置处理器的前置方法 				**postBeforeInitialization方法**     
>
> ​																		参数一 Object bean 参数二 String beanName
>
> 调用Bean的初始化方法（指定init-method）	**指定的init方法**
>
> 调用bean后置处理器的后置方法 				**postAfterInitialization方法**     实现BeanPostProcessor接口
>
> Bean投入使用 
>
> 容器关闭时 调用Bean的销毁方法（指定destroy-method）		**指定的destroy方法**



### Bean的配置

#### 反射 通过全类名

#### 工厂方法

##### 	静态工厂

> 直接调用某个类的静态方法就可以返回Bean的实例
>
> 不用创建工厂实例 通过静态方法获取bean实例
>
> ```java
> public class StaticCarFactory{
> 	private static Map<String,Car> cars=new HashMap<String,Car>();
>     static{
>         cars.put("audi", new Car("audi",300));
>     }
>     public static Car getCar(String name){
>         return cars.get(name);
>     }
> }
> ```
>
> 在配置bean时只需要配置bean 尤其是其中的class为静态工厂方法 

##### 	实例工厂

> 通过创建工厂实例来进行生成Bean实例 
>
> 在配置时需要先配置工厂的bean 其中factory-bean为实例工厂

##### 通过FactoryBean

> 自定一的FactoryBean需要实现FactoryBean<>接口
>
> 通过类代码实现FactotyBean
>
> xml配置文件中bean的class指定为对应的FactoryBean的全类名



### 基于注解配置bean

#### 配置简单bean

> ##### 组件扫描
>
> Spring能够从classpath下自动扫描，侦测和实例化具有特定注解的组件 包括：
>
> @Component		 基本注解 标识了一个受Spring管理的组件
>
> @Respository		 标识持久层组件
>
> @Service				 标识服务层 业务层组件
>
> @Controller			标识表现层组件
>
> 使用组件后，还要在配置中声明 context:component-scan 指定IOC容器扫描的包
>
> resource-pattern  仅希望扫描特定的类而非基包下面的所有类
>
> include-filter   	   子节点标识要包含的目标类
>
> exclude-filter		  子节点表示要排除在外的目标类
>
> ​		其中存在五种方式进行指定 通过注解类型 通过bean名字

#### bean的关联关系的导入

> \<context:component-scan>元素自动注册AutowiredAnnotationBeanPostProcessor实例，可以指定装配具有@Autowired 和 @Resource @Inject注解的属性
>
> autowired可以自动装配类型兼容的bean
>
> 如果多个类型兼容 则需要通过名字进行装配（通过@Qualifier进行名字的设定）



### 泛型依赖注入

> 建立了依赖的两个父类，其子类也会建立依赖关系
>
> <img src="C:\Users\11346\AppData\Roaming\Typora\typora-user-images\image-20200421233205346.png" alt="image-20200421233205346" style="zoom:67%;" />



### AOP面向切面编程

#### 为什么需要AOP

> 例如 有一个接口定义四则运算的操作，而一个类对他进行实现
>
> 现在有两个需求：1在每个运算前 输出日志 2进行数值的整型判断
>
> 如果不是用AOP技术，则需要每个方法前后都进行print。其中日志的输出方法是重复的，维护麻烦
>
> **问题1**代码混乱 原有的业务方法急剧膨胀 每个方法在处理核心逻辑时需要兼顾多个其他关注点
>
> **问题2**代码分散，多个模块重复实现重复代码，如果需求变更，需要改变多个模块的实现

#### 动态代理实现AOP

> #### 代理模式
>
> 代理类和被代理类实现共同的接口（或继承），代理类中存有指向被代理类的索引，实际执行时通过调用代理类的方法、实际执行的是被代理类的方法。
>
> #### 静态代理
>
> 由程序员创建或特定工具自动生成源代码，再对其编译。在程序运行前，代理类的.class文件就已经存在了。
>
> ```java
> public class BirdLogProxy implements Flyable {
>     private Flyable flyable;
> 
>     public BirdLogProxy(Flyable flyable) {
>         this.flyable = flyable;
>     }
> 
>     @Override
>     public void fly() {
>         System.out.println("Bird fly start...");
> 
>         flyable.fly();
> 
>         System.out.println("Bird fly end...");
>     }
> }
> ```
>
> ```java
>     public static void main(String[] args) {
>         Bird bird = new Bird();
>         BirdLogProxy p1 = new BirdLogProxy(bird);
>         BirdTimeProxy p2 = new BirdTimeProxy(p1);
> 
>         p2.fly();
>     }
> 
> ```
>
> 这就时一个代理类，通过与被代理的类实现相同接口的形式，加上聚合的形式，将需要代理的对象作为参数传入，将原有方法进行增强。但是作为静态方法，面对大量需要被代理的方法，需要同样多的静态代理类。所以需要一个代理类来代理任意对象。
>
> #### 动态代理
>
> 程序运行时，运用反射机制动态创建而成，动态代理类的字节码在程序运行时由Java反射机制动态生成，无需程序员手工编写它的源代码。
>
> ##### 动态代理的两种实现方式
>
> ##### **JDK代理**
>
> JDK动态代理主要涉及java.lang.reflect包下的Proxy类和InvocationHandler接口。 JDK代理实现的三个要点：
>
> **通过java.lang.reflect.Proxy类来动态生成代理类；**
> **代理类要实现InvocationHandler接口；**
> **JDK代理只能基于接口进行动态代理；**
>
> 每一个动态代理类都必须要实现InvocationHandler这个接口，并且每个代理类实例都关联到了一个handler，当我们通过代理对象调用一个方法的时候，这个方法的调用就会被转发为由InvocationHandler这个接口的 invoke 方法来进行调用。
>
> ```java
> Object invoke(Object proxy, Method method, Object[] args) throws Throwable
> proxy:　　 指代我们所代理的那个真实对象
> method:　　指代的是我们所要调用真实对象的某个方法的Method对象
> args:　　  指代的是调用真实对象某个方法时使用的参数
> ```
>
>  Proxy这个类的作用就是用来动态创建一个代理对象的类，它提供了许多的方法，但是我们用的最多的就是 newProxyInstance 这个方法，这个方法的作用就是得到一个动态的代理对象，其接收三个参数，我们来看看这三个参数所代表的含义。
>
> ```java
> public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) throws IllegalArgumentException
> loader:　　    一个ClassLoader对象，定义了由哪个ClassLoader对象来对生成的代理对象进行加载
> interfaces:　　一个Interface对象的数组，表示的是我将要给我需要代理的对象提供一组什么接口，如果我提供了一组接口给它，这样我就能调用这组接口中的方法了
> h:　　         一个InvocationHandler对象，表示的是当我这个动态代理对象在调用方法的时候，会关联到哪一个InvocationHandler对象上
> ```
>
> 

#### AOP术语

> 切面：横切关注点（跨越应用程序多个模块的功能）被模块化的特殊对象
>
> 通知：且面必须完成的工作（参数验证 日志每一个方法就是一个通知）
>
> 目标：被通知的对象（原本的加减乘除方法）
>
> 代理：向目标对象应用统治之后创建的对象
>
> 连接点：程序执行的某个特定位置 是一个真实存在的物理位置（例如加法执行之前 减法执行之后的位置 **某某某方法执行前/后/抛出异常时**）
>
> 切点：AOP通过切点定位到特殊的连接点

<img src="C:\Users\11346\AppData\Roaming\Typora\typora-user-images\image-20200422004426090.png" alt="image-20200422004426090" style="zoom:67%;" />
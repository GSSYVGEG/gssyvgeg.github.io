---

layout:     post
title:      spring framework 官方文档学习记录
subtitle:   0530持续更新中... 
date:       2020-05-29
author:     有林
header-img: img/home-bg-o.jpg
catalog: true
tags:

    - 学习

---
依据官方文档：<https://docs.spring.io/spring/docs/5.3.0-SNAPSHOT/spring-framework-reference/>

仅是个人学习记录，存在大量错误！

# part II：core

### 1.3 bean overview

+ Spring IoC container》》》存放bean的容器。

+ configuration metadata》》》配置元数据。

  可以理解为，他是bean的元数据。比如：你可以通过观察xml文件形式的configuration metadata就能知道bean的依赖关系等信息。

  它以xml文件、注解、java配置类 的形式存在。

  该信息由用户提供

+ bean的创建，需要configuration metadata
+ 在IoC container中，bean以BeanDefinition对象的形式存在。

#### 1.3.1 Naming Beans  命名

基于xml文件的configuration metadata，bean有id属性作为唯一名称，有name属性作为别名。

若不值得id和name，IoC container将为bean生成驼峰命名方式的名称。

#### 1.3.2 Instantiation Beans 初始化bean

> A bean definition is essentially a recipe for creating one or more objects. The container looks at the recipe for a named bean when asked and uses the configuration metadata encapsulated by that bean definition to create (or acquire) an actual object.

bean的本质是IoC container创建对象的配方。

当提出创建对象的请求时，IoC container先查看这个配方，并使用bean的元数据（即configuration metadata）来创建实际对象。

可以指定IoC container以何种方式通过bean创建出对象。

这里注意与依赖注入的方式进行区别。有以下几种：

+ 用构造函数实例化

  注意，此次由于有了一个有参构造函数，则编译器不再为您自动创建无参构造函数。问题是，有时你可能需要一个空参数的构造函数。

+ 用静态工厂方式进行实例化

  ```java
  public class ClientService {
      private static ClientService clientService = new ClientService();
      private ClientService() {}
  
      public static ClientService createInstance() {
          return clientService;
      }
  }
  ```

  ```xml
  <bean id="clientService" class="examples.ClientService"
      factory-method="createInstance"/>
  ```

  此处的createInstance()必须是返回ClientService对象的一个静态方法

+ 用实例工厂方式进行实例化

  ```java
  public class DefaultServiceLocator {
      private static ClientService clientService = new ClientServiceImpl();
      public ClientService createClientServiceInstance() {
          return clientService;
      }
  }
  ```

  ```xml
  <!-- the factory bean, which contains a method called createInstance() -->
  <bean id="serviceLocator" class="examples.DefaultServiceLocator">
      <!-- inject any dependencies required by this locator bean -->
  </bean>
  
  <!-- the bean to be created via the factory bean -->
  <bean id="clientService"
      factory-bean="serviceLocator"
      factory-method="createClientServiceInstance"/>
  ```

### 1.4 Dependencies  依赖

bean之间相互依赖的那点事。

#### 1.4.1 Dependency Injection 

DI 》》》依然注入

+ 基于构造函数的DI

  ```java
  package x.y;
  
  public class ThingOne {
      private ThingTwo thingTwo;
      private ThingThree thingThree;
      
      public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
          this.thingTwo = thingTwo;
          this.thingThree = thingThree;
      }
  }
  ```

  ```xml
  <beans>
      <bean id="beanOne" class="x.y.ThingOne">
          <constructor-arg ref="beanTwo"/>
          <constructor-arg ref="beanThree"/>
      </bean>
      <bean id="beanTwo" class="x.y.ThingTwo"/>
      <bean id="beanThree" class="x.y.ThingThree"/>
  </beans>
  ```

这里回顾一下1.3.2中的“用静态工厂方式进行实例化”。

那么xml文件应该这样：

```xml
<beans>
    <bean id="beanOne" class="x.y.ThingOne" factory-method="createInstance">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>
    //与上面相同
    ...
</beans>
```

ThingOne类应该这样：

```java
public class ThingOne {
    private ThingTwo thingTwo;
    private ThingThree thingThree;
    
    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        this.thingTwo = thingTwo;
        this.thingThree = thingThree;
    }
    
    public static ThingOne createInstance(ThingTwo thingTwo, ThingThree thingThree){
        ThingOne to = new ThingOne(thingTwo,ThingThree);
        return to;
    }
}
```

静态工厂createInstance方法的参数由 xml中的constructor-arg / elements 提供

+ 基于setter的DI

  下面是一个基于xml文件的setter方式依赖注入的例子：

  ```xml
  <bean id="exampleBean" class="examples.ExampleBean">
      <!-- setter injection using the nested ref element -->
      <property name="beanOne">
          <ref bean="anotherExampleBean"/>
      </property>
  
      <!-- setter injection using the neater ref attribute -->
      <property name="beanTwo" ref="yetAnotherBean"/>
      <property name="integerProperty" value="1"/>
  </bean>
  
  <bean id="anotherExampleBean" class="examples.AnotherBean"/>
  <bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
  ```

  观察可知，exampleBean依赖anotherExampleBean和yetAnotherBean

  ```java
  public class ExampleBean {
      private AnotherBean beanOne;
      private YetAnotherBean beanTwo;
      private int i;
  
      public void setBeanOne(AnotherBean beanOne) {
          this.beanOne = beanOne;
      }
  
      public void setBeanTwo(YetAnotherBean beanTwo) {
          this.beanTwo = beanTwo;
      }
  
      public void setIntegerProperty(int i) {
          this.i = i;
      }
  }
  ```

  在ExampleBean类中写上setBeanOne和setBeanTwo方法的实现，如此便可依赖注入成功

  AnotherBean类中如果有其他依赖，同上理。

+ 选择哪个方式注入？

> Since you can mix constructor-based and setter-based DI, it is a good rule of thumb to use constructors for mandatory dependencies and setter methods or configuration methods for optional dependencies. Note that use of the [@Required](https://docs.spring.io/spring/docs/5.3.0-SNAPSHOT/spring-framework-reference/core.html#beans-required-annotation) annotation on a setter method can be used to make the property be a required dependency; however, constructor injection with programmatic validation of arguments is preferable.

文档建议mandatory dependencies使用构造函数依赖注入，optional dependencies使用setter方式

文档还建议通常使用构造函数注入。原因如下：

> The Spring team generally advocates constructor injection, as it lets you implement application components as immutable objects and ensures that required dependencies are not `null`. Furthermore, constructor-injected components are always returned to the client (calling) code in a fully initialized state. As a side note, a large number of constructor arguments is a bad code smell, implying that the class likely has too many responsibilities and should be refactored to better address proper separation of concerns.

它可以使你意识到当前构造函数的参数过多，而过多的构造函数参数意味着这个类的责任太多，代码不好，应该重构分离代码。

接下来文档讲解了依赖注入的实现过程。

#### 1.4.2 Dependencies and Configuration in Detail 

详细介绍了xml中，bean标签的各种属性和使用细节。

#### 1.4.3 depends-on属性

使用条件：

> However, sometimes dependencies between beans are less direct. An example is when a static initializer in a class needs to be triggered, such as for database driver registration.

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

通常使用ref来表示bean依赖于某个其他bean。

depends-on也可以达到相同效果，用法如上。

#### 1.4.4 Lazy-initialized Beans懒加载

默认下，spring会预先实例化所有bean，这使得在程序一开始运行时，就可以马上发现配置或环境中的错误。通常应该采用这种方式。

但懒加载告诉IoC 容器，在第一次请求bean的实例的时候才对bean进行实例化。

#### 1.4.5 Autowiring Collaborators 自动连线

spring可以通过检查ApplicationContext 的内容，自动解析bean之间的依赖。

这样不仅更方便，而且好处多多。

相关注解@AutoWired

### 1.5 Bean Scopes  范围

强调bean是一个ioc容器用来创建实例对象的**配方**

通过bean这个配方创建的实例对象具有作用范围，超出范围就被回收？变为Null？空指针？

这个范围可以手动指定。

#### 1.5.1 The Singleton Scope  单例范围

类似于单例模式。

ioc容器创建的实例对象是单例，即返回的实例是内存中存储的同一个对象。

#### 1.5.2 The Prototype Scope 原型范围

每次向ico容器请求实例对象是，都会返回新的对象。

例子：数据访问对象，即DAO对象通常不会设置为原型范围。通常设为单例。

### ！1.9 Annotation-based Container Configuration 注解

注解方式和xml方式，哪个更好？

文档回答：视情况而定，各有优劣，可以混合使用。

注解的方式可以配合java配置类，使得可以非侵入性得使用注解方式。

#### 1.9.1 ~~~@Required~~~

从 Spring Framework 5.1开始,@required 注释就被正式废弃了

```java
public class SimpleMovieLister {
    private MovieFinder movieFinder;
    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

通过上面的知识，我们知道movieFinder是SimpleMovieLister的一项依赖。

@Required表示：在实例化SimpleMovieLister对象时，movieFinder属性必须要有一个值。

如果由于movieFinder对象实例化失败等原因，导致movieFinder是空指针，则SimpleMovieLister的实例化会抛出异常。

#### 1.9.2 @Autowired

```java
public class SimpleMovieLister {
    //方式三：
    @Autowired
    private MovieFinder  movieFinder;

   	//方式一：
    @Autowired
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
    //方式二：
    @Autowired
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

通常我们会选用方式三，因为这样代码量最少最简洁。

方式三其实等效于方式二，使用setter方式依赖注入？？？

#### 1.9.3 @Primary 微调

在@Bean出增加@Primary，就会被自动连线优先匹配到。

```java
@Configuration
public class MovieConfiguration {
    @Bean
    @Primary
    public MovieCatalog firstMovieCatalog() { ... }

    @Bean
    public MovieCatalog secondMovieCatalog() { ... }
}

public class MovieRecommender {
    @Autowired
    private MovieCatalog movieCatalog;//得到的是firstMovieCatalog返回的对象
}
```

#### 1.9.4 @Qualifier 更细致微调

```java
public class MovieRecommender {
    @Autowired
    @Qualifier("main")
    private MovieCatalog movieCatalog;//得到的是firstMovieCatalog返回的对象
}
```

```xml
<bean class="example.SimpleMovieCatalog">
    <qualifier value="main"/>
    <!-- inject any dependencies required by this bean -->
</bean>
```

不知道为什么，给bean指定qualifier限定值，官方文档写的是xml方式而不是注解方式。这个问题在文档1.10.8中解答。

#### 1.9.7 @Resource

可以注解在字段或者setter方法上。

```java
public class SimpleMovieLister {
    //方式二：
    @Resource(name="myMovieFinder")
    private MovieFinder movieFinder;

    //方式一：
    @Resource(name="myMovieFinder") 
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

上述代码表示为movieFInder注入名为“myMovieFinder”的bean对象。

如果未指定name属性。注解字段时，它采用字段名做匹配；注解setter方法时，他采用the bean property name

> In case of a setter method, it takes the bean property name.

#### 1.9.8 @Value

通常用来注入外部化属性。

```java
@Component
public class MovieRecommender {
    private final String catalog;

    public MovieRecommender(@Value("${catalog.name}") 
                            String catalog) {
        this.catalog = catalog;
    }
}
```

```java
@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig { }
```

```java
//在application.properties文件中。
catalog.name=MovieCatalog
```

### 1.10 Classpath Scanning and Managed Components 组件管理

@Component

@Repository 》》》用于注解DAO类。能自动转换（处理？）与数据库交互时发生的异常。

@Service

@Controller

#### 1.10.3 Automatically Detecting Classes and Registering Bean Definitions

@ComponentScan》》》自动扫描和注册那些添加了@Component注解的组件

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    // ...
}
```

#### 1.10.4 Using Filters to Customize Scanning 扫描过滤

@ComponentScan可以使用一些方式增加或缩小扫描范围。

```java
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
```

#### 1.10.6 Naming Autodetected Components 组件命名

如果@Component注解后不带有value值，则spring的BeanNameGenerator 会自动将其以驼峰命名法命名。

```java
@Service(value = "myMovieLister")
public class SimpleMovieLister {
    // ...
}
```

#### 1.10.7 Providing a Scope for Autodetected Components 组件作用域

默认作用域是单例。

可以用@Scope注解指定作用域。

```java
@Scope("prototype")
@Repository
public class MovieFinderImpl implements MovieFinder {}
```

#### 1.10.8 Providing Qualifier Metadata with Annotations 限定符

这里解答了我在1.9.4中的一个疑问。

```java
@Component
@Qualifier("Action")
public class ActionMovieCatalog implements MovieCatalog {}
```

限定符起到的效果和1.10.6中组件命名的效果，是基本一样的？

#### 1.10.9 Generating an Index of Candidate Components 扫描索引

组件自动扫描之前，建立索引，可以加速扫描。

### 1.11 Using JSR 330 Standard Annotations

使用@Inject替代@Autowired。

功能不如后者，不讨论了。

### 1.12 Java-based Container Configuration @Configuration 

详细介绍@Configuration 注解，及与其搭配使用的其他注解，例如：@Bean。

这里的大部分内容与上文重复。

#### 1.12.1 Basic Concepts 基本概念

+ @Bean表明了IoC容器如何实例化、初始化和配置一个对象。

他可以与@Component配合使用，但通常都是与@Configuration配合。

+ @Configuration表明了这个类是bean definitions的源。换句话说，就是这个类是用来存放Bean的。相当于xml方式中的<beans>标签的效果。

~~~java
@Configuration
public class AppConfig {
    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
~~~

+ 当@Bean写在@Component内或其他地方时，表示Bean被声明在“lite”模式下。

  一般是配合工厂模式使用的。

#### 1.12.2 Instantiating the Spring Container by Using AnnotationConfigApplicationContext

介绍AnnotationConfigApplicationContext类的使用。

+ 简单使用

  ~~~java
  public static void main(String[] args) {
      ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
      MyService myService = ctx.getBean(MyService.class);
      myService.doStuff();
  }
  ~~~

#### 1.12.3 Using the @Bean Annotation

+ 方法参数依赖。

+ 生命周期的回调。

  ```java
  //方式一：
  @Bean(initMethod = "init")  //init是返回对象的一个方法名。下同理
  
  @Bean(destroyMethod = "cleanup")
  
  //方式二：
  直接在方法实现里调用。
  ```

+ 作用范围  @Scope

+ 命名 

  @Bean(name = "myThing")

#### 1.12.4 Using the @Configuration annotation

讲解了@Configuration类中的@Bean之间的相互依赖那点事。

***题外话：为什么spring boot中标注了@Service 等注解的类，不用手写一个对应的@Bean就可以直接@Autowired使用？

答：spring boot帮我们自动生成了对应的ServiceConfig类？类似下面：

~~~java
@Configuration
public class ServiceConfig {
    @Bean
    public TransferService transferService() {
        return new TransferService();
    }
}
~~~



#### 1.12.5 Composing Java-based Configurations

+ @Import

  加载另一个@Configuration类的Bean到当前类中。

  ~~~java
  @Configuration
  public class ConfigA {
      @Bean
      public A a() {
          return new A();
      }
  }
  
  @Configuration
  @Import(ConfigA.class)
  public class ConfigB {
      @Bean
      public B b() {
          return new B();
      }
  }
  ~~~

+ xml方式和java配置类方式结合使用

  + 以xml方式为主。

  将@Configuration类引入到xml中

  + 以java配置类方式为主。

  使用@InportResource("classpath:xxx.xml")，将xml引入到配置类中。

  ~~~java
  @Configuration
  @ImportResource("classpath:/com/acme/properties-config.xml")
  public class AppConfig {
      @Value("${jdbc.url}")
      private String url;
  
      @Value("${jdbc.username}")
      private String username;
  
      @Value("${jdbc.password}")
      private String password;
  
      @Bean
      public DataSource dataSource() {
          return new DriverManagerDataSource(url, username, password);
      }
  }
  ~~~

### 1.13 Environment Abstraction 环境

#### 1.13.1Bean Definition Profiles

它描述了Bean的 一些行为。对它下手，我们可以达到一些目的，比如:让bean只在开发环境下才会被注册和实例化。

+ @Profile

  ~~~java
  @Configuration
  @Profile("development")
  public class StandaloneDataConfig {
      @Bean
      public DataSource dataSource() {
          return new EmbeddedDatabaseBuilder()
              .setType(EmbeddedDatabaseType.HSQL)
              .addScript("classpath:com/bank/config/sql/schema.sql")
              .addScript("classpath:com/bank/config/sql/test-data.sql")
              .build();
      }
  }
  ~~~

  

## 2.Resources  资源

### 2.1 Introduction 

java自带的Java.net.URL类功能太弱。

比如无法坚持http://内容是否可失效。

### 2.2 The Resource Interface

介绍Resource接口类中的几个核心方法。

+ getInputStream()获取数据流

+ exists()
+ isOpen() ：getInputStream()获取的数据流是否是打开状态。
+ getDescription()

### 2.3 Built-in Resource Implementations 接口实现

### 2.4 The ResourceLoader  资源加载器

资源获取器，用它来从目标地获取资源，资源以Resource对象的形式返回。

spring中所有ApplicationContext类都实现了ResourceLoader，因此ApplicationContext就是一个资源加载器。

下文中的ctx就是一个ApplicationContext对象。

~~~java
Resource template = ctx.getResource("https://myhost.com/resource/path/myTemplate.txt");

Resource template = ctx.getResource("classpath:some/resource/path/myTemplate.txt");
~~~

## 4 Spring Expression Language（SpEL）

为什么要用SpEL？

## 5 Aspect Oriented Programming with Spring（AOP）

用spring实现面向切面编程。

### 5.1 AOP Concepts 概念

Aspect：切面。跨多个类的一些功能集合。

JoinPoint：连接点、织入点。织入的目标点，如：核心业务代码。

Advice: 一般翻译为“增强”。准备织入的功能。

Pointcut：一般翻译为“切入点”。匹配连接点与advice的语句表达式。

Weaving：织入。

Introduction：可以为代理目标动态地增加新接口。



织入时机，即advice执行时间点：

befor：连接点执行之前、

after return：连接点执行之后、

After throwing：连接点抛出异常后、

Finaly：无论连接点正常执行还是抛出异常，都会执行advice，类似关键字finally、

Around：包围。

### 5.2 Spring AOP Capabilities and Goals  能力

spring 的aop只支持对方法的拦截。不支持对字段拦截。

### 5.3 AOP Proxies

对于接口，spring aop采用标准JDK动态代理来实现aop。

对于普通类，采用CGLIB来实现。

### 5.4 @AspectJ support

#### 5.4.1 Enabling @AspectJ Support

@EnableAspectJAutoProxy  开启aop支持

~~~java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {}
~~~

#### 5.4.2 Declaring an Aspect 声明切面

@Aspect

~~~java
package org.xyz;
import org.aspectj.lang.annotation.Aspect;

@Aspect
public class NotVeryUsefulAspect {
	//...
    //这里会有advice、piontcut、introduction等内容
}
~~~

#### 5.4.3 Declaring a Pointcut 声明切入点

`

@Pointcut  使用此注解时，Advice在何时执行，是取决于括号内的表达式？

~~~java
@Pointcut("execution(* transfer(..))") // the pointcut expression 切入点表达式
private void anyOldTransfer() {} // the pointcut signature  切入点签名
~~~

"execution(* transfer(..))"，括号内是一个正则的Aspectj 5 的切入点表达式。

表达式写法参照AspectJ的官方指南。

+ Supported Pointcut Designators 切入点指示器

  包括：execution、within、this、target、args等

+ 标准JDK方式代理时，只有public修饰的方法能被拦截；

  CGLIB方式时，public和protected修饰的方法能被拦截。

+ Combining Pointcut Expressions 切入点表达式

  可以使用&&、||、！组合表达式。

  ~~~java
  @Pointcut("execution(public * *(..))")
  private void anyPublicOperation() {} 
  
  @Pointcut("within(com.xyz.someapp.trading..*)")
  private void inTrading() {} 
  
  @Pointcut("anyPublicOperation() && inTrading()")
  private void tradingOperation() {} 
  ~~~

#### 5.4.4 Declaring Advice 

+ @Before

  ~~~java
  @Aspect
  public class BeforeExample {
      @Before("execution(* com.xyz.myapp.dao.*.*(..))")
      public void doAccessCheck() {
          // ...
      }
  }
  ~~~

+ @AfterReturning  代理对象的方法正常返回后执行

  ~~~java
  @Aspect
  public class AfterReturningExample {
      @AfterReturning(
          pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
          returning="retVal")
      public void doAccessCheck(Object retVal) {
          // ...
      }
  }
  ~~~

  retVal是代理对象的方法的返回值

+ @AfterThrowing  代理对象的方法抛出异常时执行

  ~~~java
  @Aspect
  public class AfterThrowingExample {
      @AfterThrowing(
          pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
          throwing="ex")
      public void doRecoveryActions(DataAccessException ex) {
          // ...
      }
  }
  ~~~

  ex是代理对象的方法抛出的异常对象，例子中异常对象类型是DataAccessException，虽然可以是Exception，但明确些会更好。

+ @After  代理对象的方法抛出异常或是正常执行，都会执行

+ @Around  

  + 该注解修饰的方法的定义内容时线程安全的。

  + 该注解修饰的方法的第一个参数必须是ProceedingJoinPoint类型。

  + 在方法定义中调用ProceedingJoinPoint的proceed ()，会使代理目标的方法开始执行。
  + proceed ()调用一次或多次或不调用，都是可以的。

  + 官方文档建议除非必要，否则尽量不用这个注解。总之，不建议杀鸡用牛刀。

  ~~~java
  @Aspect
  public class AroundExample {
      @Around("com.xyz.myapp.SystemArchitecture.businessService()")
      public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
          // start stopwatch
          Object retVal = pjp.proceed();
          // stop stopwatch
          return retVal;
      }
  }
  ~~~

+ Advice的方法第一个参数可以是org.aspectj.lang.JoinPoint类型。

  around advice的ProceedingJoinPoint是JoinPoint的子类。

  JoinPoint有以下方法：

  + getArgs()  返回接入点方法的参数
  + getThis()  返回代理目标对象
  + getTarget()  同getThis() 
  + getSignature()  
  + toString()

#### 5.4.5 Introductions 

### 5.8 Proxying Mechanisms  代理机制

+ 当一个类实现了某个接口，spring aop使用标准JDK动态代理的方式为这个类动态创建代理对象，实现aop。

  注意此时只能拦截接口中已声明的方法。未声明的的方法无法拦截。

+ 当一个类似普通类，也就是没有实现某个接口。spring aop使用CGLIB来动态创建代理对象。CGLIB被打包在spring-core中。

  此时代理对象是继承自原对象。原对象中的所有方法都能被拦截。

  注意此方式下，切面中被final修饰的方法不能作为advice，因为他不能重写。

#### 5.8.1 Understanding AOP Proxies 理解代理

~~~java
public class SimplePojo implements Pojo {
    public void foo() {
        // this next method invocation is a direct call on the 'this' reference
        this.bar();
    }
    public void bar() {
        // some logic...
    }
}
~~~

上面代码中，如果接入点是foo（）方法，foo中调用了bar。

我们假设spring aop动态创建的代理类是SimplePojoProxy。代理类中的foo方法中调用的bar（）方法不是代理类中的bar，而会使SimplePojo中的bar方法。

若要改变上述现象，使其调用的是代理类中的bar方法。则需要写侵入性的代码。

~~~java
public class SimplePojo implements Pojo {
    public void foo() {
        // this works, but... gah!
        ((Pojo) AopContext.currentProxy()).bar();
    }

    public void bar() {
        // some logic...
    }
}
~~~

另外，调用的时候还需要这样。

~~~java
Pojo pojo = (Pojo) proxyFactory.getProxy();
        // this is a method call on the proxy!
pojo.foo();
~~~

总之，尽量别这样做。

## 7 Null-safety 空指针

@Nullable：指示字段、方法入参、返回值可以为空指针。

@NonNull：与@Nullable相反，不可为空。

​	对于注解了@NonNullApi或@NonNullFields的字段、参数、返回值，可以不用再加@NonNull。

@NonNullApi：注解在包上。指示参数和返回值不能为空指针。

@NonNullFields：注解在包上。指示字段值不能为空指针。

这些注解可以被IDE工具识别，用于警告空指针。

## 8 Data Buffers and Codecs 数据缓冲

spring提供了一些用于数据缓冲相关的类。

这部分内容官方文档笔墨不多，并且基本没有提供使用示例代码，应该是因为使用方法过于简单。

### 8.1 DataBufferFactory

一个抽象类。

用来创建DataBuffer。

可以指定容量大小，这样更有效率。但其实它可以自动增缩。

### 8.2 DataBuffer

### 8.3 PooledDataBuffer

没啥用，官方文档建议直接使用DataBufferUtils

### 8.4 DataBufferUtils

### 8.6 Using DataBuffer

注意记得释放缓冲区数据。

Decoder：编码器。帮我们管理了缓冲区数据，包括释放缓冲区数据。

# part III. Testing

## 2 Unit Testing

### 2.1 Mock Objects 仿冒对象

## 3 Integration Testing

### 3.6 Spring MVC Test Framework

#### 3.6.1 Server-Side tests

+ Performing Requests

  详细介绍mockMvc.perform()的用法。//TODO:待详细列出







# 快速上手，初步理解

#### 从xml中获取对象

```java
package cf.vgeg;
public class Car{
    private String tyre;
    public  void run(){
        ...
    }
}
```

```xml
spring.xml文件中

<bean id="car" class="cf.vgeg.Car"></bean>
```

```java
public static void main(){
    ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
    Car c = (Car)context.getBean("car");//此处的car为xml中bean的id
} 
```

+ context.getBean()仅为示例。官方文档不建议使用此类方法获取spring生成的对象，文档建议使用@Autowired注解等方式。

  > You can then use `getBean` to retrieve instances of your beans. The `ApplicationContext` interface has a few other methods for retrieving beans, but, ideally, your application code should never use them. Indeed, your application code should have no calls to the `getBean()` method at all and thus have no dependency on Spring APIs at all. For example, Spring’s integration with web frameworks provides dependency injection for various web framework components such as controllers and JSF-managed beans, letting you declare a dependency on a specific bean through metadata (such as an autowiring annotation).

##### 使用@Component等效实现:

value="car"    value缺省值为类名的驼峰写法。

这个值相当于xml文件中bean的id

```java
@Component(value="car")
public class car(){...}
```

在xml配置扫描位置:

```xml
<context:component-scan base-package="cf.vgeg"></context:component-scan>
<!--<bean id="car" class="cf.vgeg.Car"></bean>
```

#### 依赖注入

```java
Car c = (Car)context.getBean("car");
System.out.println(c.getTyre());//输出:null
```

##### 方法一：set注入

+ xml方式

```xml
<bean id="car" class="cf.vgeg.Car">
	<property name="Tyre" value="丰田"></property>
</bean>
```

这句其实调用了Car类中的setTyre方法，将tyre属性值设为“丰田”

##### 方法二：构造函数注入

```java
public class Car{
    private String tyre;
    public Car(String tyre){
        this.tyre = tyre;
    }
}
```



```xml
<bean id="car" class="cf.vgeg.Car">
	<constructor value="丰田"></constructor>
</bean>
```

#### 使用java配置类，抛弃xml

创建AppConfig类，添加注解@Configuration

添加@ComponentScan(basePackages="cf.vgeg")，相当于xml配置扫描位置

使用方式：

```java
ApplicationContext context2 = new AnnotationConfigApplicationContext(AppConfig.class);
Car c2 = context2.getBean(Car.class);
```

以上动作在spring boot中以自动完成。


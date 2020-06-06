---

layout:     post
title:      spring boot 官方文档学习记录
subtitle:   0530持续更新中...
date:       2020-05-29
author:     有林
header-img: img/home-bg-o.jpg
catalog: true
tags:

    - 学习

---
根据官方文档编写：<https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/reference/html/index.html>

# part IV. Using Spring Boot

## 2. Structuring Your Code 构建你的代码

### 2.1 Using the “default” Package 

总之就是，强烈推荐遵循java推荐的包命名约定，例如：com.example.project

否则可能引发不明问题。例如在使用@ComponentScan、@EntityScan等时，扫描不到响应的类。

### 2.2 Locating the Main Application Class 主启动类

建议放在根包中。并用SpringBootApplication注解。

该注解做的事，包括但不限于：扫描@Entity类、扫描@Component。

主启动类还需含有一个main方法，并执行run()方法

~~~java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
~~~

## 3. Configuration Classes 配置类

spring boot 支持java配置类的方式进行配置。

官方文档建议尽量始终使用java配置类的方式，而不是xml方式。

## 3.1 Importing Additional Configuration Classes 配置类嵌套

一个@Configuration类可以使用@Import注解来引入其他的@Configuration类。

或者使用@ComponentScan注解来扫描所有@Component组件和@Configuration类。

## 4. Auto-configuration 自动配置

spring boot根据添加的jar包依赖，自动配置spring 程序。

用@EnableAutoConfinguration注解或@SpringBootApplication注解，来开启自动配置功能。

### 4.1 Gradually Replacing Auto-configuration 自定义配置取代自动配置

自动配置功能具有非侵入的特点。

添加自定义配置后，自动配置对应的配置内容将会自动失效，替换为自定义配置内容。

在ide中，使用debug模式启动spring boot程序，更详细的的日志将会打印在控制台上，方便了解自动配置功能到底配置了哪些内容。

### 4.2 Disabling Specific Auto-configuration Classes 取消某个自动配置项

@SpringBootApplication的exclude属性可以取消某个自动配置项。

~~~java
@SpringBootApplication(exclude={DataSourceAutoConfiguration.class})
public class MyApplication {
}
~~~

## 5. Spring Beans and Dependency Injection 依赖注入和Bean

可以使用spring framework中的bean注册方式。

但spring boot提供了更简单的方式。

使用@ComponentScan即可自动搜索bean，并完善bean所需的configuration metadata信息。

`@Component`, `@Service`, `@Repository`, `@Controller` etc.  这些内容会自动注册为Bean。

~~~java
@Service
public class DatabaseAccountService implements AccountService {
    private final RiskAssessor riskAssessor;

    @Autowired  //满足条件时，这注解甚至可以不写
    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }
    // ...
}
~~~

上面代码中，如果DatabaseAccountService中只有一个构造函数，你甚至可以省略@AutoWired注解，也能注入riskAssessor对象。

## 6. Using the @SpringBootApplication Annotation

这个注解包含了下面三个注解：

+ @EnableAutoConfiguration：开启自动配置。
+ @ComponentScan：与被注解类 处于同级别和以下级别的包中的组件、配置类，将会被扫描并注册。
+ @Configuration：将被注解类 导入到applicationContent中。

### 7.5 Hot Swapping 热部署

spring boot只是普通的java程序，所以jvm的hot-swapping可以实现热部署。但会受限与jvm本身的热部署能力。（简单说就是有些修改无法实现热部署效果。）

更进一步的解决方案是使用JRebel。

另外spring-boot-devtools模块包括了对springboot程序进行快速重启的支持。

## 8. Developer Tools  开发工具

引入：

~~~xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
~~~

spring-boot-devtools做了些什么：

+ 禁用缓存

+ 在项目文件发生修改时，它会让springboot程序自动重启。

  可以在IDEA中设置失去焦点后自动Build，即可触发重启。

  它触发的重启，与常规手动重启程序还是有所不同的。可能会出现重启失败、更新失败等各种问题，最稳妥的还是常规手动重启。

+ `/META-INF/maven`, `/META-INF/resources`, `/resources`, `/static`, `/public`, or `/templates`，此类与静态资源相关的路径下的文件变动不会触发重启，但会触发live reload重新加载。

  

# part V. Spring Boot Features

## 1. SpringApplication

深入介绍spring boot 的主启动类。

...

## 2. Externalized Configuration 自定义配置

深入介绍如何自定义配置，以替代默认配置。

简单的单个配置值，可以使用@Value直接注入。

@ConfigurationProperties可以加载配置文件或配置对象。

官方文档接着列出了17项配置的优先级顺序。

...

### 2.4 Profile-specific Properties

主配置文件application.yml，可以通过不同的命名，区分运行环境。

命名规则：application-{ profile }.yml，profile值缺省值为default

例子：application-production.yml

## 4. logging 日志

spring boot 对所有内部日志记录都使用Commons Logging日志接口，并且开发了日志实现。

可以自由选用喜欢的日志实现，例如：Log4J2、Logback等。并且springboot还未他们提供了默认配置。

//TODO:

## 6. JSON 对json的支持

推荐首选Jackson。

### 6.1 Jackson

spring boot启动时，会自动配置一个 ObjectMapper 的bean。

如需修改ObjectMapper配置，参考 part 10.“How-to” Guides的4.3 Customize the Jackson ObjectMapper

## 15. Calling REST Services with RestTemplate

官网文档restTemplate对象的注入采用的是RestTemplateBuilder的方式。

~~~java
@Service
public class MyService {
    private final RestTemplate restTemplate;
	
    @AutoWired //官网文档代码中，这里注解省略了。还是手动加上吧。
    public MyService(RestTemplateBuilder restTemplateBuilder) {
        this.restTemplate = restTemplateBuilder.build();
    }

    public Details someRestCall(String name) {
        return this.restTemplate.getForObject("/{name}/details", Details.class, name);
    }
}
~~~

## 17. Validation 数据校验

使用条件：项目依赖中有JSR-303的实现，如：Hibernate validator

就可以使用javax.validation中的注解。一个例子：

~~~java
@Service
@Validated
public class MyService {

    public Archive findByCodeAndAuthor(@Size(min = 8, max = 10) String code,
            Author author) {
        ...
    }
}
~~~









## 26. Testing

在这里总结一下spring boot程序测试。

单元测试通常会用到mock对象。组成一个程序的代码单元可能成千上万，所以一般只对核心逻辑等部分作单元测试。其他部分用集成测试包括在内。

下面列举一个集成测试的例子，可能会持续优化：

~~~ java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
//@ContextConfiguration(classes = xxx.class)  //加载配置类
//@ActiveProfiles("dev")  //当bean具有@Profile("dev")注解时，才会在ApplicationContext加载时be active
//@TestPropertySource(locations = "classpath:application-dev.yml")  //加载测试用的配置
public class IntegrationTest {
    @Autowired
    TestRestTemplate restTemplate;

    @Test
    void testExample() throws Exception {
        ResponseEntity<String> response = restTemplate.getForEntity("/music/api/lrc/5", String.class);
        assertEquals(response.getStatusCode(), HttpStatus.OK);

        //ObjectMapper objectMapper = new ObjectMapper();
        boolean b = response.getBody().contains("翩翩一叶扁舟");
        System.out.println(response.getBody());
        assertTrue(b);
    }
}
~~~

+ Q：使用MockMvc还是TestRestTemplate?

  A：官网描述是：

  > If you have web endpoints that you want to test against this mock environment, you can additionally configure [`MockMvc`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/testing.html#spring-mvc-test-framework) 

  MockMvc使用在mock环境中，也就是@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)时。

  所以它无法测试到一些底层的servlet行为。（在26.3.5中有具体描述）

  也就是说MockMvc能替代TestRestTemplate的大部分功能，但后者更为全面，更稳妥。所以在集成测试中最好用TestRestTemplate。

最后列举一些参考文章：

+ [springboot test 人类使用指南](https://zhuanlan.zhihu.com/p/111418479)
+ [Spring Boot中的测试](https://zhuanlan.zhihu.com/p/139398204)
+ [一文教会你如何在Spring中进行集成测试，太赞了](<https://zhuanlan.zhihu.com/p/128510132>) 解释了一些常用注解
+ [基于spring-boot的单元和集成测试方案](https://zhuanlan.zhihu.com/p/67801427) 非常详细的介绍！

### 26.3 Testing Spring Boot Applications

spring boot 提供@SpringBootTest注解来标注测试类。

就像 @Component有更具体话的@Service之类的注解；@SpringBootTest也有具体化注解，例如：@WebMvcTest、@DataJpaTest等。

+ 默认情况下，测试时不会启动原服务程序。

  可以通过@SpringBootTest的webEnvironment属性来选择测试的运行方式

  + MOCK模式（默认）：加载一个web `ApplicationContext`并提供一个仿冒的web环境。
  
    个人感觉这个模式应该常用于单元测试时。
  
  + RANDOM_PORT：加载一个WebServerApplicationContext，并提供真实web环境，但访问端口号是随机的。
  
  + DEFINED_PORT：与上面类似，区别是端口号来源于application.yml中的配置，未配置时使用默认的8080端口。
  
    个人感觉，该模式和随机端口模式，应该常用于集成测试。
  
  + NONE ：只加载ApplicationContext，不提供web服务。

#### 26.3.5 Testing with a mock environment 测试执行在虚拟环境中

~~~java
@SpringBootTest
@AutoConfigureMockMvc
class MockMvcExampleTests {
    @Test
    void exampleTest(@Autowired MockMvc mvc) throws Exception {
        mvc.perform(get("/")).andExpect(status().isOk()).andExpect(content().string("Hello World"));
    }
}
~~~

如果测试代码需要一个web客户端的话，可以如上面代码所示，注入一个MockMvc。

或者可以注入一个WebTestClient

~~~java
@Test
    void exampleTest(@Autowired WebTestClient webClient) {
        webClient.get().uri("/").exchange().expectStatus().isOk().expectBody(String.class).isEqualTo("Hello World");
    }
~~~

上面两种方式中，MockMvc的方式通常执行起来更快。

但MockMvc也有个缺点，不能测试低级别的行为，例如：来自servlet的错误页面，mockmvc无法取得错误页面。（具体见官方文档）

#### 26.3.10 Auto-configured Tests 自动配置

这个标题有点不知所云。其实，这里主要讲的是：

对于单元测试，我们不需要加载并配置完整的程序，只加载待测部分的程序即可。

例如，测试spring mvc的controller层时，只关心是否对url请求正确处理，而不关心它具体返回的数据，因为这些数据在数据库中，数据的正确与否是Repository层该测试的。

spring boot会利用spring-boot-test-autoconfigure模块，自动完成以上效果。

与之相关的注解有@AutoConfigure…，包括在了@...Test内部。

#### 26.3.12 Auto-configured Spring MVC Tests @WebMvcTest

这个注解是@SpringBootTest针对controller层测试的具体化注解之一。

它主要针对@controller、Filter、HandlerInterceptor等，测试他们的功能是否正常。例如spring security的功能是否正常。

@WebMvcTest通常只针对一个@Controller类，并与@MockBean配合使用。

MockMvc的作用相当于HTTP客户端，它不需要启动完整的HTTP服务器，就能对Controller发出请求。

~~~java
@WebMvcTest(UserVehicleController.class)
class MyControllerTests {
    @Autowired
    private MockMvc mvc;

    @MockBean
    private UserVehicleService userVehicleService;

    @Test
    void testExample() throws Exception {
        given(this.userVehicleService.getVehicleDetails("sboot"))
                .willReturn(new VehicleDetails("Honda", "Civic"));
        this.mvc.perform(get("/sboot/vehicle").accept(MediaType.TEXT_PLAIN))
                .andExpect(status().isOk()).andExpect(content().string("Honda Civic"));
    }
}
~~~

上面@MockBean虚拟出的userVehicleService对象，只是用来方便提供原Service层方法的返回对象。

+ 额外的：
  + given()可翻译为“假定”.
  + mvc.perform()的详细用法，在spring framework官方文档的Part iii.Testing的3.6.1的Performing Requests

#### 26.3.14 Auto-configured Data JPA Tests 测试jpa

@DataJpaTest

+ 默认下，它会扫描被@Entity注解的类，并自动在内存中创建出一个数据库。

  注意上面自动配置的数据库中不会包含具体数据条目。所以在测试中，一些读取数据库的操作可能会返回空内容。

  可以通过@AutoConfigureTestDatabase使其不使用内存中的数据库，转而使用实际的数据库。这样能获得实际数据库中的具体数据，防止测试返回空值。

  ~~~java
  @DataJpaTest
  @AutoConfigureTestDatabase(replace=Replace.NONE)
  class MusicRepositoryTest {
      //@Autowired
      //private TestEntityManager entityManager;
  
      @Autowired
      private MusicRepository musicRepository;
  
      @Test
      void testExample(){
          boolean existsByWyId = this.musicRepository.existsByWyId("144187");
          boolean existsById = this.musicRepository.existsById(2);
          this.musicRepository.findAll().forEach(music -> {
              System.out.println("***music list:"+music.getTitle());
          });
          System.out.println("***existsByWyId:"+ existsByWyId);
          System.out.println("***existsById:"+ existsById);
      }
  }
  ~~~

  

+ 默认下，jpa测试是具有事务性，会在测试结束的时候回滚数据。可以通过@Transactional禁用。

~~~java
@DataJpaTest
@AutoConfigureTestDatabase(replace= AutoConfigureTestDatabase.Replace.NONE)
class ExampleNonTransactionalTests {
~~~

+ Q：报错：Unable to find a @SpringBootConfiguration, you need to use @ContextConfiguration or @SpringBootTest(classes=...) with your test java.lang.IllegalStateException

  A：

  + 无法找到@SpringBootConfiguration。

  + Stack Overflow上有相关问题：<https://stackoverflow.com/questions/39084491/unable-to-find-a-springbootconfiguration-when-doing-a-jpatest>

  + 这个问题在spring boot官方文档的spring boot features的26.3.2中也有详细解释。

  > Spring Boot’s `@*Test` annotations search for your primary configuration automatically whenever you do not explicitly define one.

  > The search algorithm works up from the package that contains the test until it finds a class annotated with `@SpringBootApplication` or `@SpringBootConfiguration`. As long as you [structured your code](https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/reference/html/using-spring-boot.html#using-boot-structuring-your-code) in a sensible way, your main configuration is usually found.

  ​	在spring framework中使用测试时，需要手动使用@ContextConfiguration(classes=…)指定加载某个@Configuration标注的类。

  ​	但在spring boot里，spring boot的@SpringBootTest、@DataJpaTest等注解会自动搜索匹配主配置类。

  ​	搜索算法是，从包含测试类的包开始搜索，知道找到一个带`@SpringBootApplication` 或`@SpringBootConfiguration`的类。

  ​	所以在maven项目中，测试类的包结构要与待测代码的包结构相符。
  
  ​	为了让spring boot能自动找到它，需要注意测试类的全限定名要与业务代码对应。即注意测试类的包路径。如下：
  
  ```java
  my-test-project
    +--pom.xml
    +--src
      +--main
        +--com
          +--example
            +--Application.java
      +--test
        +--com
          +--example
            +--test
          +--JpaTest.java
  ```
  
  ​	如果不想变动包结构，可以使用手动指定的方式。（未实践）
  
  ```java
  @SpringBootTest
  @Import(MyTestsConfiguration.class)
  class MyTests {
      @Test
      void exampleTest() {
          ...
      }
  }
  ```
  
  + 官方文档进一步描述了一个特殊情况，需注意。
  
    位置在：在spring boot官方文档的spring boot features的26.3.26 User Configuration and Slicing
  
    > It then becomes important not to litter the application’s main class with configuration settings that are specific to a particular area of its functionality.
  
    解决办法是将配置写在另一个配置类中，不要注解在spring启动类上。

#### 26.3.24  Auto-configured Spring Web Services Tests @WebServiceClientTest

不常用。

默认下，它会配置一个mock的 WebServiceServer 的Bean。并自动定制WebServiceTemplateBuilder

### 26.4 Test Utillities 一些工具

#### 26.4.4 TestRestTemplate

替代RestTemplate，在集成测试中很有用。

它不会抛出服务器端的错误。

可以直接实例化它，然后即可使用，如下：

~~~java
public class MyTest {
    private TestRestTemplate template = new TestRestTemplate();

    @Test
    public void testRequest() throws Exception {
        HttpHeaders headers = this.template.getForEntity(
                "https://myhost.example.com/example", String.class).getHeaders();
        assertThat(headers.getLocation()).hasHost("other.example.com");
    }

}
~~~

当测试环境选择的是`WebEnvironment.RANDOM_PORT` 或`WebEnvironment.DEFINED_PORT`时，可以直接注入。

通过RestTemplateBuilder bean，可以对TestRestTemplate进行详细自定义设置。

如下：

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class SampleWebClientTests {
    @Autowired
    private TestRestTemplate template;

    @Test
    void testRequest() {
        HttpHeaders headers = this.template.getForEntity("/example", String.class).getHeaders();
        assertThat(headers.getLocation()).hasHost("other.example.com");
    }

    @TestConfiguration(proxyBeanMethods = false)
    static class Config {
        @Bean
        RestTemplateBuilder restTemplateBuilder() {
            return new RestTemplateBuilder().setConnectTimeout(Duration.ofSeconds(1))
                    .setReadTimeout(Duration.ofSeconds(1));
        }

    }
}
```

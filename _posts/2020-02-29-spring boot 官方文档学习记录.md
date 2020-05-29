---

layout:     post
title:      spring boot 官方文档学习记录
subtitle:   桃花满溪水似镜，尘心如垢洗不去。
date:       2020-05-29
author:     有林
header-img: img/home-bg-o.jpg
catalog: true
tags:

    - 学习

---
根据官方文档编写：<https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/reference/html/index.html>

# part V. Spring Boot Features

## 26 Testing

### 26.3 Testing Spring Boot Applications

spring boot 提供@SpringBootTest注解来标注测试类。

就像 @Component有更具体话的@Service之类的注解；@SpringBootTest也有具体化注解，例如：@WebMvcTest、@DataJpaTest等。

+ 默认情况下，测试时不会启动原服务程序。

  可以通过@SpringBootTest的webEnvironment属性来选择测试的运行方式

  + MOCK模式（默认）：加载一个web `ApplicationContext`并提供一个仿冒的web环境。
  + RANDOM_PORT：加载一个WebServerApplicationContext，并提供真实web环境。

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

  A：无法找到@SpringBootConfiguration。

  这个问题在spring boot官方文档的spring boot features的26.3.2中详细解释了。

  > Spring Boot’s `@*Test` annotations search for your primary configuration automatically whenever you do not explicitly define one.

  > The search algorithm works up from the package that contains the test until it finds a class annotated with `@SpringBootApplication` or `@SpringBootConfiguration`. As long as you [structured your code](https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/reference/html/using-spring-boot.html#using-boot-structuring-your-code) in a sensible way, your main configuration is usually found.

  在spring framework中使用测试时，需要手动使用@ContextConfiguration(classes=…)指定加载某个@Configuration标注的类。

  但在spring boot里，spring boot的@SpringBootTest、@DataJpaTest等注解会自动搜索匹配主配置类。

  搜索算法是，从包含测试类的包开始搜索，知道找到一个带`@SpringBootApplication` 或`@SpringBootConfiguration`的类。

  所以在maven项目中，测试类的包结构要与待测代码的包结构相符。

  为了让spring boot能自动找到它，需要注意测试类的全限定名要与业务代码对应。即注意测试类的包路径。如下：

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

  如果不想变动包结构，可以使用手动指定的方式。（未实践）

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

## 核心特性
### SpringApplication
#### 懒初始化
懒初始化：开启懒初始化后，bean会在使用的时候被创建。开启懒初始化后会减少应用启动时间。懒初始化的缺点在于，会延迟暴露应用程序的问题（比如循环依赖），因此懒初始化不是默认开启的。

#### events
Spring Boot 内部使用events来处理各种各样的任务，具体有哪些事件可以参考[7.1.7. Application Events and Listeners](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.spring-application.application-events-and-listeners)。了解即可。
> You often need not use application events, but it can be handy to know that they exist. Internally, Spring Boot uses events to handle a variety of tasks.

#### 获取Application Arguments
通过以下方式可以获取参数
```java
@Component
public class MyBean {

    public MyBean(ApplicationArguments args) {
        boolean debug = args.containsOption("debug");
        List<String> files = args.getNonOptionArgs();
        if (debug) {
            System.out.println(files);
        }
        // if run with "--debug logfile.txt" prints ["logfile.txt"]
    }

}
```

### Profiles
Spring Profiles使得应用可以有多个不同的配置，根据不同的环境激活不同的配置。@Component, @Congifuration, or @ConfigurationProperties都可以被标记成@Profile来指定在不同的环境下被激活。
```java
@Configuration(proxyBeanMethods = false)
@Profile("production")
public class ProductionConfiguration {

    // ...

}
```

环境相关的还有：`spring.profiles.active`、`spring.profiles.group`等配置。此外还可以通过编程来设置环境。

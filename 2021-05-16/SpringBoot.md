
## Spring Boot学习
## Spring Boot 的核心注解是哪个？它主要由哪几个注解组成的？
启动类上面的注解是@SpringBootApplication，它也是 Spring Boot 的核心注解，主要组合包含了以下 3 个注解：

- @SpringBootConfiguration：组合了 @Configuration 注解，实现配置文件的功能。
- @EnableAutoConfiguration：打开自动配置的功能，也可以关闭某个自动配置的选项，
- 例如：java 如关闭数据源自动配置功能： @SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })。
- @ComponentScan：Spring组件扫描。

###  Spring Security 和 Shiro 各自的优缺点

Spring Boot 官方提供了大量的非常方便的开箱即用的 Starter ，包括 Spring Security 的 Starter ，使得在 Spring Boot 中使用 Spring Security 变得更加容易，甚至只需要添加一个依赖就可以保护所有的接口，所以，如果是 Spring Boot 项目，一般选择 Spring Security 。当然这只是一个建议的组合，单纯从技术上来说，无论怎么组合，都是没有问题的。Shiro 和 Spring Security 相比，主要有如下一些特点：

- Spring Security 是一个重量级的安全管理框架；Shiro 则是一个轻量级的安全管理框架
- Spring Security 概念复杂，配置繁琐；Shiro 概念简单、配置简单
- Spring Security 功能强大；Shiro 功能简单
### Spring Boot 中的 starter 到底是什么 ?

- Starter 并非什么新的技术点，基本上还是基于 Spring 已有功能来实现的。
- 首先它提供了一个自动化配置类，一般命名为 XXXAutoConfiguration ，在这个配置类中通过条件注解来决定一个配置是否生效（条件注解就是 Spring 中原本就有的）
- 然后它还会提供一系列的默认配置，也允许开发者根据实际情况自定义相关配置
- 然后通过类型安全的属性(spring.factories)注入将这些配置属性注入进来，新注入的属性会代替掉默认属性。
- 因为如此，很多第三方框架，我们只需要引入依赖就可以直接使用了。当然，开发者也可以自定义 Starter

### Spring Boot 配置加载顺序？
在 Spring Boot 里面，可以使用以下几种方式来加载配置。

1. properties文件；
2. YAML文件；
3. 系统环境变量；
4. 命令行参数；
5. 。。。

### SpringBoot的自动配置原理是什么
主要是Spring Boot的启动类上的核心注解SpringBootApplication注解主配置类，有了这个主配置类启动时就会为SpringBoot开启一个@EnableAutoConfiguration注解自动配置功能。
这个EnableAutoConfiguration类：

- 从配置文件META_INF/Spring.factories加载可能用到的自动配置类
- 去重，并将exclude和excludeName属性携带的类排除
- 过滤，将满足条件（@Conditional）的自动配置类返回

### SpringBoot 实现热部署有哪几种方式？

热部署就是可以不用重新运行SpringBoot项目可以实现操作后台代码自动更新到以运行的项目中
主要有两种方式：

1. Spring Loaded
1. Spring-boot-devtools



### SpringBoot事务的使用

SpringBoot的事务很简单:
1. 首先使用注解EnableTransactionManagement开启事物之后
1. 然后在Service方法上添加注解Transactional便可。

### Async异步调用方法

在SpringBoot中使用异步调用是很简单的:
- 只需要在方法上使用@Async注解即可实现方法的异步调用。
- 注意：需要在启动类加入@EnableAsync使异步调用@Async注解生效。

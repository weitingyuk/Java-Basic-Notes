## SpringBoot 是如何进行自动配置的？

1. 在使用main()启动SpringBoot的时候，只有一个注解@SpringBootApplication;
2. @SpringBootApplication等同于下面三个注解：

- @SpringBootConfiguration: 底层是Configuration注解,就是支持JavaConfig的方式来进行配置(使用Configuration配置类等同于XML文件)。
- @EnableAutoConfiguration: 开启自动配置功能
- @ComponentScan: 扫描注解，默认是扫描当前类下的package。将@Controller/@Service/@Component/@Repository等注解加载到IOC容器中。

其中@EnableAutoConfiguration是关键(启用自动配置)，内部实际上就去加载META-INF/spring.factories文件的信息，然后筛选出以EnableAutoConfiguration为key的数据，加载到IOC容器中，实现自动配置功能。


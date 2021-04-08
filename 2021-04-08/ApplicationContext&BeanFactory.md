Spring ApplicationContext VS BeanFactory

-  国际化：BeanFactory没有扩展MessageResource接口
-  强大的事件机制（Event）:ApplicationContext主要通过ApplicationEvent和ApplicationListener这两个接口来实现的
-  底层资源的访问：ApplicationContext实现了ResourceLoader接口，可以加载多个Resource
-  对Web应用的支持：ApplicationContext可以通过声明的方式创建ClassLoader
-  延迟加载：ApplicationContext一次性创建所有bean，BeanFactory是调用Bean才加载的

总结：
- BeanFactory是比较原始的Factory, 不支持AOP、WEB等插件,ApplicationContext是面向框架的，会上下文进行分层，提供更多面向应用的功能

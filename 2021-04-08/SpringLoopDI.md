## Spring 是如何处理循环依赖？



### 说明：
- 这里的循环依赖指的是单例模式下的 Bean 字段注入时出现的循环依赖。Spring 只解决单例模式下的循环依赖。
- 构造器注入对于 Spring 无法自动解决（应该考虑代码设计是否有问题），可通过延迟初始化来处理。


### 实现方法：
在 Spring 底层 IoC 容器 BeanFactory 中处理循环依赖的方法主要借助于以下 3 个 Map 集合：

##### singletonObjects（一级 Map）：
- 里面保存了所有已经初始化好的单例 Bean，也就是会保存 Spring IoC 容器中所有单例的 Spring Bean；
##### earlySingletonObjects（二级 Map）：
- 里面会保存从 三级 Map 获取到的正在初始化的Bean 
##### singletonFactories（三级 Map）
- 里面保存了正在初始化的 Bean 对应的 ObjectFactory 实现类，调用其 getObject() 方法返回正在**初始化的 Bean 对象**（仅实例化还没完全初始化好）
- 如果存在则将获取到的 Bean 对象并保存至 二级 Map，同时从当前 三级 Map 移除该 ObjectFactory 实现类。


当通过 getBean 依赖查找时会首先依次从上面三个 Map 获取，存在则返回，不存在则进行初始化，这三个 Map 是处理循环依赖的关键。

##### 例如：
两个 Bean 出现循环依赖，A 依赖 B，B 依赖 A；当我们去依赖查找 A，在实例化后初始化前会先生成一个 ObjectFactory  对象（可获取当前正在初始化 A）保存在上面的 singletonFactories 中，初始化的过程需注入 B；接下来去查找 B，初始 B 的时候又要去注入 A，又去查找 A ，由于可以通过 singletonFactories 直接拿到正在初始化的 A，那么就可以完成 B 的初始化，最后也完成 A 的初始化，这样就避免出现循环依赖。

##### 问题一：为什么需要上面的 二级 Map ？
- 避免重复处理： 因为通过 三级 Map获取 Bean 会有相关 SmartInstantiationAwareBeanPostProcessor#getEarlyBeanReference 的处理，避免重复处理，处理后返回的可能是一个代理对象
- 例如在循环依赖中一个 Bean 可能被多个 Bean 依赖， A -> B（也依赖 A） -> C -> A，当你获取 A 这个 Bean 时，后续 B 和 C 都要注入 A，没有上面的 二级 Map的话，三级 Map 保存的 ObjectFactory 实现类会被调用两次，会重复处理，可能出现问题，这样做在性能上也有所提升
##### 问题二：为什么不直接调用这个 ObjectFactory#getObject() 方法放入 二级Map 中，而需要上面的 三级 Map？
- AOP：对于不涉及到 AOP 的 Bean 确实可以不需要 singletonFactories（三级 Map），但是 Spring AOP 就是 Spring 体系中的一员，如果没有singletonFactories（三级 Map），意味着 Bean 在实例化后就要完成 AOP 代理，这样违背了 Spring 的设计原则。
- Spring 是通过 AnnotationAwareAspectJAutoProxyCreator 这个后置处理器在完全创建好 Bean 后来完成 AOP 代理，而不是在实例化后就立马进行 AOP 代理。如果出现了循环依赖，那没有办法，只有给 Bean 先创建代理对象，但是在没有出现循环依赖的情况下，设计之初就是让 Bean 在完全创建好后才完成 AOP 代理。
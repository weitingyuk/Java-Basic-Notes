## Spring事务失效的 8 大原因
用 Spring 的 @Transactional 注解控制事务有哪些不生效的场景？


但是我觉得还是总结得不够全，今天栈长我再总结一下，再延着这位粉丝的总结再补充完善一下，不用说，我肯定也不见得总结全，但希望可以帮忙有需要的人。

### 1. 数据库引擎不支持事务
这里以 MySQL 为例，其 MyISAM 引擎是不支持事务操作的，InnoDB 才是支持事务的引擎，一般要支持事务都会使用 InnoDB。

从 MySQL 5.5.5 开始的默认存储引擎是：InnoDB，之前默认的都是：MyISAM，所以这点要值得注意，底层引擎不支持事务再怎么搞都是白搭。

### 2. 没有被 Spring 管理
如下面例子所示：


```
// @Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    public void updateOrder(Order order) {
        // update order
    }
    
}
```

如果此时把 @Service 注解注释掉，这个类就不会被加载成一个 Bean，那这个类就不会被 Spring 管理了，事务自然就失效了。

### 3. 方法不是 public 的
 @Transactional 只能用于 public 的方法上，否则事务不会失效，如果要用在非 public 方法上，可以开启 AspectJ 代理模式。

### 4. 自身调用问题
来看两个示例：


```
@Service
public class OrderServiceImpl implements OrderService {

    public void update(Order order) {
        updateOrder(order);
    }
    
    @Transactional
    public void updateOrder(Order order) {
        // update order
    }
    
}
```

update方法上面没有加 @Transactional 注解，调用有 @Transactional 注解的 updateOrder 方法，updateOrder 方法上的事务管用吗？

再来看下面这个例子：

```
@Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    public void update(Order order) {
        updateOrder(order);
    }
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void updateOrder(Order order) {
        // update order
    }
    
}
```

这次在 update 方法上加了 @Transactional，updateOrder 加了 REQUIRES_NEW 新开启一个事务，那么新开的事务管用么？

这两个例子的答案是：不管用！

因为它们发生了自身调用，就调该类自己的方法，而没有经过 Spring 的代理类，默认只有在外部调用事务才会生效，这也是老生常谈的经典问题了。

这个的解决方案之一就是在的类中注入自己，用注入的对象再调用另外一个方法，这个不太优雅，另外一个可行的方案可以参考《Spring 如何在一个事务中开启另一个事务？》这篇文章。

### 5. 数据源没有配置事务管理器

```
@Bean
public PlatformTransactionManager transactionManager(DataSource dataSource) {
    return new DataSourceTransactionManager(dataSource);
}
```

如上面所示，当前数据源若没有配置事务管理器，那也是白搭！

### 6. 不支持事务
来看下面这个例子：


```
@Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    public void update(Order order) {
        updateOrder(order);
    }
    
    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public void updateOrder(Order order) {
        // update order
    }
    
}
```

Propagation.NOT_SUPPORTED： 表示不以事务运行，当前若存在事务则挂起，详细的可以参考《事务隔离级别和传播机制》这篇文章。

都主动不支持以事务方式运行了，那事务生效也是白搭！

### 7. 异常被吃了
这个也是出现比较多的场景：


```
// @Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    public void updateOrder(Order order) {
        try {
            // update order
        } catch {
            
        }
    }
    
}
```

把异常吃了，然后又不抛出来，事务怎么回滚吧！

### 8. 异常类型错误
上面的例子再抛出一个异常：


```
// @Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    public void updateOrder(Order order) {
        try {
            // update order
        } catch {
            throw new Exception("更新错误");
        }
    }
    
}
```

这样事务也是不生效的，因为默认回滚的是：RuntimeException，如果你想触发其他异常的回滚，需要在注解上配置一下，如：

@Transactional(rollbackFor = Exception.class)
这个配置仅限于 Throwable 异常类及其子类。

#### 总结
本文总结了八种事务失效的场景，其实发生最多就是方法不是 public 的、自身调用、异常被吃、异常抛出类型不对这三个了。
## 简述 Spring bean 的生命周期
### 一. 主要四个阶段

- 实例化 Instantiation -createBeanInstance()
- 属性赋值 Populate - populateBean() 
- 初始化 Initialization - initializeBean() 
- 销毁 Destruction  - disposableBean() 

### 二. 扩展点

##### 1. 实例化和属性赋值先后有下面三个方法
他们在类 **InstantiationAwareBeanPostProcessor**,该类是BeanPostProcessor的子接口，通过beanName来进行个性化定制bean:
- postProcessBeforeInstantiation(Class beanClass, String beanName)：在bean实例化之前调用
- postProcessProperties(PropertyValues pvs, Object bean, String beanName)：在bean实例化之后、设置属性前调用
- postProcessAfterInstantiation(Class beanClass, String beanName)：在Bean实例化之后调用


## 2. **BeanPostProcessor**该接口有两个方法，作用于初始化阶段的前后

- postProcessBeforeInitialization(Object bean, String beanName)：在初始化之前调用此方法
- postProcessAfterInitialization(Object bean, String beanName)：在初始化之后调用此方法



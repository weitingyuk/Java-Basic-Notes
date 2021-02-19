## Spring MVC 的原理和流程
### Spring MVC 的原理
- Spring Web MVC是一种基于Java的实现了Web MVC设计模式的**请求驱动类型**的轻量级Web框架，即使用了MVC架构模式的思想，将web层进行职责解耦，基于请求驱动指的就是使用**请求-响应模型**。
- Spring Web MVC使用了前端控制器模式来进行设计，再根据请求映射规则分发给相应的页面控制器（动作/处理器）进行处理。

### Spring MVC 的流程
1. 用户发送请求至前端控制器DispatcherServlet
1. DispatcherServlet收到请求调用处理器映射器HandlerMapping。
1. 处理器映射器根据请求url找到具体的处理器，生成处理器执行链HandlerExecutionChain(包括处理器对象和处理器拦截器)一并返回给DispatcherServlet。
1. DispatcherServlet根据处理器Handler获取处理器适配器HandlerAdapter执行HandlerAdapter处理一系列的操作，如：参数封装，数据格式转换，数据验证等操作
1. 执行处理器Handler(Controller，也叫页面控制器)。
1. Handler执行完成返回ModelAndView
1. HandlerAdapter将Handler执行结果ModelAndView返回到DispatcherServlet
1. DispatcherServlet将ModelAndView传给ViewReslover视图解析器
1. ViewReslover解析后返回具体View
1. DispatcherServlet对View进行渲染视图（即将模型数据model填充至视图中）。
1. DispatcherServlet响应用户。

### Spring MVC 的好处
- 让我们能非常简单的设计出干净的Web层和进行更简洁的Web层的开发；
- 非常容易与其他视图技术集成，如Velocity、FreeMarker等等，因为模型数据不放在特定的API里，而是放在一个Model里；
- 非常灵活的数据验证、格式化和数据绑定机制，能使用任何对象进行数据绑定，不必实现特定框架的API；


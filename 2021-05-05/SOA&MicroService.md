## SOA 与微服务

### SOA 与微服务的关联：
从本质上看，这两种技术所做的都是同一件事：将一个较大的问题分解为多个较小的问题。

### SOA 与微服务区别
#### SOA
- SOA（Service Oriented Architecture）面向服务架构，它可以通过网络对松散耦合的粗粒度应用组件进行分布式部署、组合和使用。
- SOA可以看作是B/S模型、XML（标准通用标记语言的子集）/Web Service技术之后的自然延伸。
#### 微服务

- 微服务架构的系统是一个分布式的系统，按业务进行划分为独立的服务单元，解决单体系统的不足，同时也满足越来越复杂的业务需求。
- 微服务就是将一个单体架构的应用按业务划分为一个个的独立运行的程序即服务，它们之间通过协议(例如HTTP)进行通信（也可以采用消息队列来通信），可以采用不同的编程语言，使用不同的存储技术，自动化部署（如Jenkins）减少人为控制，降低出错概率。
- 因为服务数量越多，管理起来越复杂，因此采用集中化管理。例如Eureka，Zookeeper等都是比较常见的服务集中化管理框架。

#### 区别
微服务架构 = 80%的SOA服务架构思想 + 100%的组件化架构思想 + 80%的领域建模思想

- SOA是一种设计方法，其中包含多个服务， 服务之间通过相互依赖最终提供一系列的功能。一个服务通常以独立的形式存在与操作系统进程中，各个服务之间通过网络调用。
- 微服务是在SOA上做的升华，微服务架构强调的一个重点是“业务需要彻底的组件化和服务化”，原有的单个业务系统会拆分为多个可以独立开发、设计、运行的小应用。这些应用之间通过服务完成交互和集成。


### SOA架构特点
1. 系统集成
- 站在系统的角度，解决企业系统间的通信问题，把原先散乱、无规划的系统间的网状结构，梳理成规整、可治理的系统间星形结构，这一步往往需要引入一些产品，比如 ESB、以及技术规范、服务管理规范； 这一步解决的核心问题是有序。
ESB：企业服务总线

2. 系统的服务化
- 站在功能的角度，把业务逻辑抽象成可复用、可组装的服务，通过服务的编排实现业务的快速再生。其目的是把原先固有的业务功能转变为通用的业务服务，实现业务逻辑的快速复用；这一步解决的核心问题是复用。

3. 业务的服务化
- 站在企业的角度，把企业职能抽象成可复用、可组装的服务；把原先职能化的企业架构转变为服务化的企业架构，进一步提升企业的对外服务能力；前面两步都是从技术层面来解决系统调用、系统功能复用的问题。第三步，则是以业务驱动把一个业务单元封装成一项服务。这一步解决的核心问题效率。

### 微服务架构特点
1. 通过服务实现组件化
- 开发者不再需要协调其它服务部署对本服务的影响。

2. 按业务能力来划分服务和开发团队
- 开发者可以自由选择开发技术，提供 API 服务。

3. 去中心化
- 每个微服务有自己私有的数据库持久化业务数据；
- 每个微服务只能访问自己的数据库，而不能访问其它服务的数据库；
- 某些业务场景下，需要在一个事务中更新多个数据库。这种情况也不能直接访问其它微服务的数据库，而是通过对于微服务进行操作；
- 数据的去中心化，进一步降低了微服务之间的耦合度，不同服务可以采用不同的数据库技术（SQL、NoSQL等）。在复杂的业务场景下，如果包含多个微服务，通常在客户端或者中间层（网关）处理。
- 
4. 基础设施自动化
- JavaEE部署架构，通过展现层打成WAR包，业务层划分到JAR包最后部署为EAR一个大包，而微服务则改变了这种方式，把应用拆分成为一个一个的单个服务，应用Docker技术，不依赖任何服务器和数据模型，是一个全栈应用，可以通过自动化方式独立部署，每个服务运行在自己的进程中，通过轻量的通讯机制联系，经常是基于HTTP资源API，这些服务基于业务能力构建，能实现集中化管理。
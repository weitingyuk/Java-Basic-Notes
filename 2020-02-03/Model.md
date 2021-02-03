# JVM 中内存模型是怎样的，简述新生代与老年代的区别？

## JVM的内存结构大概分为：

- 堆（Heap）：线程共享。所有的对象实例以及数组都要在堆上分配。回收器主要管理的对象。
- 方法区（Method Area）：线程共享。存储类信息、常量、静态变量、即时编译器编译后的代码。
- 方法栈（JVM Stack）：线程私有。存储局部变量表、操作栈、动态链接、方法出口，对象指针。
- 本地方法栈（Native Method Stack）：线程私有。为虚拟机使用到的Native 方法服务。如Java使用c或者c++编写的接口服务时，代码在此区运行。
- 程序计数器（Program Counter Register）：线程私有。有些文章也翻译成PC寄存器（PC Register），同一个东西。它可以看作是当前线程所执行的字节码的行号指示器。指向下一条要执行的指令。

## 新生代与老年代的区别
### 新生代（New Generation）
- 大多数情况下Java程序中新建的对象都从新生代分配内存
- 新生代由Eden Space和两块相同大小的Survivor Space（通常又称为S0和S1或From和To）构成，
- 可通过-Xmn参数来指定新生代的大小，也可通过-XX:SurvivorRatio来调整Eden Space及Survivor Space的大小。
- 不同的GC方式会以不同的方式按此值来划分Eden和Survivor Space，有些GC方式还会根据运行状况来动态调整Eden、S0、S1的大小。 
### 老年代（Old Generation/Tenuring Generation）

- 用于存放新生代中经过多次垃圾回收仍然存活的对象，例如缓存对象，新建的对象也有可能在老年代上直接分配内存。
- 主要有两种状况（由不同的GC实现来决定）：
    - 一种为大对象，可通过在启动参数上设置-XX:PretenureSizeThreshold=1024（单位为字节，默认值为0）来代表当对象超过多大时就不在新生代分配，而是直接在老年代分配(此参数在新生代采用Parallel Scavenge GC时无效)，Parallel Scavenge GC会根据运行状况决定什么对象直接在老年代上分配内存；
    - 另一种为大的数组对象，且数组中无引用外部对象。
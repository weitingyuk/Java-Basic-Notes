## 为什么需要Java类加载器
Java类不会一次全部加载到内存中，而是在应用程序需要时加载。此时，JRE调用Java类加载器，这些类加载器将类动态地加载到内存中。

## Java类加载的特点
1. 不是所有的类都由一个类加载器加载。
1. Java类加载流程是根据类的类型和类的路径，将决定加载特定类的类加载器。
1. 可以使用getClassLoader()方法知道加载类的类加载器。
1. 所有类都是基于它们的名称加载的，如果没有找到这些类中的任何一个，那么它将返回NoClassDefFoundError或ClassNotFoundException。

## Java类加载流程
类加载的过程：加载 -> 连接（验证、准备、解析）-> 初始化-> [使用 -> 卸载]
- 1）加载：查找并导入类的字节码，根据这些字节码创建Class对象
- 2）链接：其中分为三步
    - ①校验：检查导入的字节码的完整性，正确性、安全性。
    - ②准备：为静态域分配存储空间
    - ③解析：将符号引用转折为直接引用（非必需）
- 3）初始化：初始化静态变量并执行静态域代码

## Java中类加载器的类型
Java类加载器有三种类型：应用程序类加载器,扩展类加载器,引导类加载

---
注意：类加载器委托层次结构模型总是按照**应用程序类加载器->扩展类加载器->引导类加载器**的顺序运行。引导类加载器总是被赋予更高的优先级，其次是扩展类加载器，然后是应用程序类加载器。

---

#### 引导类加载器(BootStrap ClassLoader)
- 当JVM启动的时候调用引导类加载。
- 它不是java类。它的工作是加载第一个pure Java类加载器。
- 引导类加载器从 **rt.jar** 加载类基础类库.
- 引导类加载器没有任何父类加载器。它也被称为原始类装入器。

#### 扩展类加载器(Extension ClassLoader:)
- 扩展类加载器是Bootstrap类加载器的子类
- 从相应的**JDK扩展库**中加载核心java类的扩展。
- 它从jre/lib/ext目录或系统属性指向的任何其他目录加载文件java.ext.dirs文件.

#### 系统类加载器(System ClassLoader)
- 应用程序类加载器也称为系统类加载器。
- 它加载在环境变量CLASSPATH、-CLASSPATH或-cp命令行选项中找到的应用程序类型类。
- 应用程序类加载器是扩展类加载器的子类。

## Java类加载器的功能原理

Java类加载器有三个原则，它们是：

#### 1.委托模型
类加载器：缓存 -> 子系统 -> system -> extension -> bootstrap -> findclass（current）
- 类加载器基于委托模型将类加载到Java文件中。
- ClassLoader始终遵循委托层次结构原则。
1. 每当JVM遇到一个类时，它都会检查该类是否已经加载。
1. 如果类**已经在方法区域中加载**，那么JVM继续执行。
1. 如果类不在方法区域中，那么JVM要求Java类加载器子系统加载该特定类
2. 类加载器子系统将控制权移交给应用程序类加载器。
1. 应用程序类加载器将请求委托给扩展类加载器，扩展类加载器又将请求委托给引导类加载器。
1. 引导类加载器将在引导类路径（JDK/JRE/LIB）中搜索。如果类可用，则加载它；如果不可用，则将请求委托给扩展类加载器。
1. 扩展类加载器在扩展类路径（JDK/JRE/LIB/EXT）中搜索类。如果类可用，则加载它；如果不可用，则将请求委托给应用程序类加载器。
1. 应用程序类加载器在应用程序类路径中搜索类。如果类可用，则加载它；如果不可用，则生成ClassNotFoundException异常。

#### 2.可见性原则
- 可见性原则：**父类加载器加载的类对子类加载器可见，但子类加载器加载的类对父类加载器不可见**。
- 假设一个class已由扩展类加载器加载，则该类仅对扩展类加载器和应用程序类加载器可见，而对引导类加载器不可见。
如果再次尝试使用Bootstrap ClassLoader加载该类，它会给出一个异常java.lang.ClassNotFoundException类.

#### 3.唯一性属性
Uniquesness属性确保类是唯一的，并且类没有重复。这也确保了父类装入器装入的类不会被子类装入器装入。如果父类加载器找不到类，那么当前实例将自己尝试这样做。

## Java.lang.ClassLoader方法
- 在JVM请求类之后，需要遵循几个步骤来加载类。类是按照委托模型加载的，但是有一些重要的方法或函数在加载类时起着至关重要的作用。

#### 1. loadClass（String name，boolean resolve）
这个方法用于加载JVM引用的类。它以类的名称作为参数。这是loadClass（String，boolean）类型。
#### 2. defineClass（）
defineClass（）方法是最终方法，不能重写。此方法用于将字节数组定义为类的实例。如果类无效，则抛出ClassFormatError。
#### 3. findClass（String name）
此方法用于查找指定的类。此方法只查找但不加载类。
#### 4. findLoadedClass（String name）
这个方法用于验证JVM引用的类以前是否加载过。
#### 5. Class.forNameClass（String name，boolean initialize，ClassLoader）
此方法用于加载类以及初始化类。这个方法还提供了选择任何一个类加载器的选项。如果ClassLoader参数为NULL，则使用引导类加载器。
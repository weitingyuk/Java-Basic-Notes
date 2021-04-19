## JVM、JRE、JDK有什么联系与区别？

- JVM是java虚拟机，能够将 class 文件中的字节码指令进行识别并调用操作系统向上的 API 完成动作。
- JRE是java运行时环境，它主要包含两个部分，jvm 的标准实现和 Java 的一些基本类库。它相对于 jvm 来说，多出来的是一部分的 Java 类库。换句话说，JRE包含JVM。
- JDK是java开发工具包，它集成了 jre 和一些好用的小工具。例如：javac.exe，java.exe，jar.exe 等。JDK包含JRE。
- 所以总得来说，JDK>JRE>JVM。
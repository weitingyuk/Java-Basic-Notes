## error 和 exception 的区别是什么
### 1.异常分类
Throwable是Java中所有错误和异常的超类。它的下一级是Error和Exception

### 1.1 Error（错误）
- Error是指程序运行时系统的内部错误和资源耗尽错误。程序不会抛出该类对象。
- 如果出现了Error，代表程序运行时JVM出现了重大问题，比如常见的OutOfMemoryError（OOM），这时应当告知用户并尽量让程序安全结束。

### 1.2 Exception（异常）
Exception是指程序可以自身处理的异常。Exception又分为检查异常（CheckedException）和运行异常（RuntimeException）:

- CheckedException：检查异常一般是外部错误，都发送在编译阶段，是我们在编码时应当可以预计会发生的异常情况，编译器通常会提示我们去捕获这些异常并进行处理。
    - 我们可以通过try-catch来捕获或者throws语句来抛出，否则编译器会提示不通过。
    - 常见的有：FileNotFoundException，SQLException，ClassNotFoundException。
- RuntimeException：运行异常一般是Java虚拟机正常运行期间抛出的异常的超类，程序中可以处理这些异常，也可以不处理这些异常，编译器并不会提示我们来处理这些异常。
    - 这些异常通常都是编码出现了错误导致的，我们应当尽量避免出现这些异常。
    - 常见的有：NullPointerException（空指针），IndexOutOfBoundsException（下标越界），ClassCastException（类型强制转换异常）。
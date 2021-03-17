## String，StringBuffer，StringBuilder 之间有什么区别
- String 字符串常量(final修饰，不可被继承)，String是常量，当创建之后即不能更改
- StringBuffer 字符串变量（线程安全），其也是final类别的，不允许被继承
- StringBuilder 字符串变量（非线程安全）
#### 区别：
- StringBuffer 是线程安全的，它的所有公开方法都是同步的，StringBuilder 是没有对方法加锁同步的，所以毫无疑问，StringBuilder 的性能要远大于 StringBuffer。
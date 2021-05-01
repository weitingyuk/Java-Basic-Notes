## 简述 Java 中的自动装箱与拆箱

### 一. 什么是自动装箱和拆箱
1. 自动装箱：Java自动将原始类型值转换成对应的对象，如将int的变量转换成Integer对象。
2. 自动拆箱: 将Integer对象转换成int类型值。
3. 因为这里的装箱和拆箱是自动进行的非人为转换，所以就称作为自动装箱和拆箱。

##### 有哪些类型？
原始类型byte,short,char,int,long,float,double和boolean对应的封装类为Byte,Short,Character,Integer,Long,Float,Double,Boolean。


### 二.如何实现 
1. 自动装箱时编译器调用valueOf将原始类型值转换成对象。
1. 自动拆箱时，编译器通过调用类似intValue(),doubleValue()这类的方法将对象转换成原始类型值。


### 三. 实践
- 因为基础数据类型的优点是会缓存一些基本的数据类型值，所以效率更高，一般建议尽量使用基本数据类型。
- 使用缓存的策略是因为缓存的这些对象都是经常使用到的，防止每次自动装箱都创建一次对象的实例。
- 其中只有double和float没有使用缓存，每次都是new一个新对象。

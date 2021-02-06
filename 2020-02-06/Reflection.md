# 简述 Java 的反射机制及其应用场景

#### 什么是反射？
- 在运行时检查或修改方法、类和接口的行为。

#### 反射特点
- 反射的类在java.lang.reflect package下面。
- 反射为我们提供了有关对象所属的类的信息，以及可以使用该对象执行的类的方法。
- 通过反射，我们可以在运行时调用方法。
- 反射可用于获取以下信息–
    - getClass（）方法用于获取对象所属类的名称。
    - 构造函数getConstructors（）方法用于获取对象所属类的公共构造函数。
    - 方法getMethods（）方法用于获取对象所属类的公共方法。

### 反射的优点：
- 可扩展性：可以使用外部的、用户定义的类，通过使用类路径创建对象的实例。
- 调试和测试工具：调试工具使用反射属性检查类上的私有成员。

### 反射的缺点：
- 性能开销：反射操作的性能比非反射操作慢，在性能要求高的系统中应该避免使用反射操作。
- 暴露内部：反射代码打破了抽象，因此可能会随着平台的升级而改变行为。

```java
import java.lang.reflect.Method;
import java.lang.reflect.Field;
import java.lang.reflect.Constructor;
class Demo
{
  public static void main(String args[]) throws Exception
  {
    Test obj = new Test();
    Class cls = obj.getClass();
    System.out.println("The name of class is " + cls.getName());
    Constructor constructor = cls.getConstructor();
    System.out.println("The name of constructor is " + constructor.getName());
    System.out.println("The public methods of class are : ");
    Method[] methods = cls.getMethods();
    for (Method method:methods){
      System.out.println(method.getName());
    }
    Method methodcall1 = cls.getDeclaredMethod("method2", int.class);
    methodcall1.invoke(obj, 19);
    Field field = cls.getDeclaredField("s");
    field.setAccessible(true);
    field.set(obj, "JAVA");
    Method methodcall2 = cls.getDeclaredMethod("method1");
    methodcall2.invoke(obj);
    Method methodcall3 = cls.getDeclaredMethod("method3");
    methodcall3.setAccessible(true);
    methodcall3.invoke(obj);
  }
}
class Test
{
  private String s;
  public Test()  {  s = "Tingyu-Test"; }
  public void method1()  {
    System.out.println("The string is " + s);
  }
  public void method2(int n)  {
    System.out.println("The number is " + n);
  }

  private void method3() {
    System.out.println("Private method3 invoked");
  }
}
```
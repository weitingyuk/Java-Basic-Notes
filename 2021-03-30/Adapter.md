## 简述装饰者模式以及适配器模式
### 一. 装饰者模式
#### 1. 定义
装饰模式是在不改变对象结构的情况下，动态的给该对象添加一些功能的模式。属于结构型模式。
##### 特点：
-  1，不改变原类文件。
-  2，不使用继承。
-  3，动态扩展。


#### 2. 使用场景
- 需要在**运行时动态的**给一个对象增加额外的职责时候
- 需要给一个现有的类增加职责，但是又**不想通过继承的方式**来实现的时候（应该优先使用组合而非继承），或者通过继承的方式不现实的时候（可能由于排列组合产生类爆炸的问题）。

#### 3. 典型应用
##### Java I/O中的装饰者模式
使用 Java I/O 的时候总是有各种输入流、输出流、字符流、字节流、过滤流、缓冲流等等各种各样的流，因为整个Java IO体系都是基于字符流(InputStream/OutputStream) 和 字节流(Reader/Writer)作为基类，加上各种功能进行装饰的。
##### spring cache 中的装饰者模式
##### spring cache 中的装饰者模式
对ServletRequest进行了包装，ServletRequestWrapper 是第一层包装，HttpServletRequestWrapper 通过继承进行包装，增加了 HTTP 相关的功能，SessionRepositoryRequestWrapper 又通过继承进行包装，增加了 Session 相关的功能

#### 4. 举例

```
制作咖啡为例，咖啡制作过程是动态变化的。
例如有的需要原味咖啡，有的需要加奶咖啡，有的需要加糖咖啡，而有的需要先加奶再加糖咖啡，而有的需要先加糖再加奶的咖啡，。。。
这是一个排列组合问题
```
首先我们有一个ICoffee接口，里面有一个制作咖啡的接口方法makeCoffee()。要进行装饰的类 OriginalCoffee 和装饰者基类CoffeeDecorator（一般为抽象类）实现了此接口。CoffeeDecorator类里面持有一个ICoffee引用，我们第一步会把要装饰那个原始对象赋值给这个引用，那样在装饰者类中才可以调用到那个被装饰的对象的方法。MilkDecorator 和SugarDecorator 都继承至CoffeeDecorator， 都是具体的装饰者类。

##### 具体实现
##### 第一步：先声明一个原始对象的接口

```
public interface ICoffee {
    void makeCoffee();
}
```

##### 第二步：构建我们的原始对象
此处为原味咖啡对象，它实现了ICoffee接口。

```
public class OriginalCoffee implements ICoffee {
    @Override
    public void makeCoffee() {
        System.out.print("原味咖啡 ");
    }
}
```

##### 第三步：构建装饰者抽象基类
它要实现与原始对象相同的接口ICoffee，其内部持有一个ICoffee类型的引用，用来接收被装饰的对象


```
public abstract class CoffeeDecorator implements ICoffee {
    private  ICoffee coffee;
    public CoffeeDecorator(ICoffee coffee){
        this.coffee=coffee;
    }

    @Override
    public void makeCoffee() {
        coffee.makeCoffee();
    }
}
```

##### 第四步：构建各种装饰者类
他们都继承至装饰者基类 CoffeeDecorator。此处生成了两个，一个是加奶的装饰者,另一个是加糖的装饰者。


```
public class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(ICoffee coffee) {
        super(coffee);
    }
    @Override
    public void makeCoffee() {
        super.makeCoffee();
        addMilk();
    }
    private void addMilk(){
           System.out.print("加奶 ");
     }    
}
public class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(ICoffee coffee) {
        super(coffee);
    }
    @Override
    public void makeCoffee() {
        super.makeCoffee();
        addSugar();
    }
    private void addSugar(){
           System.out.print("加糖");
     } 
}
```

##### 第五步：客户端使用


```
public static void main(String[] args) {
        //原味咖啡
        ICoffee coffee=new OriginalCoffee();
        coffee.makeCoffee();
        System.out.println("");

        //加奶的咖啡
        coffee=new MilkDecorator(coffee);
        coffee.makeCoffee();
        System.out.println("");

        //先加奶后加糖的咖啡
        coffee=new SugarDecorator(coffee);
        coffee.makeCoffee();
    }
```
#### 5. 优缺点
###### 优点
- 采用装饰者模式扩展对象的功能比采用继承更加灵活
- 可以设计出多个不同的具体装饰者，创造出不同行为的组合

###### 缺点
- 装饰者模式增加了许多子类，如果过度使用会使得程序变得很复杂。

### 二. 适配器模式
那么我们什么时候使用这个模式呢？
#### 1. 定义
定义一个包装类，用于包装不兼容接口的对象

```
包装类 = 适配器Adapter；
被包装对象 = 适配者Adaptee = 被适配的类
```
#### 2. 使用场景
把一个类的接口变换成客户端所期待的另一种接口，从而使原本接口不匹配而无法一起工作的两个类能够在一起工作。


```
适配器模式的形式分为：类的适配器模式 & 对象的适配器模式
```
#### 3. 典型应用
Java语言的数据库连接工具JDBC，JDBC给出一个客户端通用的抽象接口，每一个具体数据库引擎（如SQL Server、Oracle、MySQL等）的JDBC驱动软件都是一个介于JDBC接口和数据库引擎接口之间的适配器软件
#### 4. 举例子
如果去美国，我们随身带的电器是无法直接使用的，因为美国的插座标准和中国不同，所以，我们需要一个适配器：
实现
##### 目标接口（客户端调用的接口）


```

public interface Target {
    //客户端需要请求处理的方法
    public void request();
}
```

##### 源接口（需要被适配的接口）

```
//源接口（已经存在的接口）
//需要被转换的对象
//这个接口需要重新配置以适应目标接口
public class Adaptee {

    public void specifiRequest() {
        System.out.println("源接口对象调用源接口中的方法");
    }
}
```

##### 适配器




```
public class Adapter implements Target {

    //持有源接口对象
    private Adaptee adaptee;

    /**
     * 构造方法，传入需要被适配的对象
     * @param adaptee
     */
    public Adapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }

    /**
     * 重写目标接口的方法，以适应客户端的需求
     */
    @Override
    public void request() {
        //调用源接口的方法
        System.out.println("适配器包装源接口对象，调用源接口的方法");
        adaptee.specifiRequest();
    }
}
```

##### 客户端

```
public class Client {
    public static void main(String[] args){

        //创建源对象（被适配的对象）
        Adaptee adaptee = new Adaptee();
        //利用源对象对象一个适配器对象，提供客户端调用的方法
        Adapter adapter = new Adapter(adaptee);
        System.out.println("客户端调用适配器中的方法");
        adapter.request();

    }
}
```

#### 5. 优缺点
##### 优点
目标类和适配者类解耦，增加了类的透明性和复用性，同时系统的灵活性和扩展性都非常好，更换适配器或者增加新的适配器都非常方便，符合“开闭原则”

##### 缺点
过多的使用适配器，会让系统非常零乱，不易整体进行把握。比如，明明看到调用的是A接口，其实内部被适配成了B接口的实现


### 三. 适配器模式 VS  装饰器模式
#### 适配器模
- 持有的是待适配的对象，实现的是目标接口，两个不一样
- 扩展了待适配类新的功能
- 适配器模式拓展了新的功能
#### 装饰器模式
- 持有对象和继承的对象一般是同一个类或接口
- 装饰模式中装饰过的类力求与原来对外接口一致，这使得在调用方看来，装饰过后的类与之前没有什么区别，只是某一方面功能增强了
- 装饰模式增强了原有功能
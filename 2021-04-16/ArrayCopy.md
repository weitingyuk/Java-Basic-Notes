## Java 如何高效进行数组拷贝
#### 1、for循环
- 很灵活，但是代码不够简洁.
#### 2、System.arraycopy()
- 可以看到是native方法：native关键字说明其修饰的方法是一个原生态方法，方法对应的实现不是在当前文件，而是在用其他语言（如C和C++）实现的文件中。 可以将native方法比作Java程序同Ｃ程序的接口。

```
public static native void arraycopy(Object src,  int  srcPos,
           Object dest, int destPos,int length);
```


#### 3、Arrays.copyOf()
下面是源码，可以看到本质上是调用的arraycopy方法。，那么其效率必然是比不上 arraycopy的


```
public static int[] copyOf(int[] original, int newLength) {
   int[] copy = new int[newLength];
   System.arraycopy(original, 0, copy, 0,
   Math.min(original.length, newLength));
   return copy;
}
```

#### 4、clone()
- 返回的是Object，需要强制转换。 一般用clone效率是最差的，


#### 总结：

- 当数组大小比较小的时候for循环的效率最高，完胜其他方法的效率
- 当数组大小在1W-100W的时候 clone 效率最高，System.arraycopy也不差，for循环的效率比较糟糕
- 当数组大小比较大的时候，1亿 for循环效率最高，clone效率最慢
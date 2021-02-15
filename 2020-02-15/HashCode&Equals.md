## hashcode 和 equals 方法的联系

### 联系
当你重写一个类的equals方法的时候，一定要重写hashCode。
### 原因
如果你不这样做的话，就会违反了Object.hashCode的通用规范，这通常会导致一些hash类的问题，比如HashMap,HashSet以及HashTable。

### 覆盖equals方法时总要覆盖Hashcode
下面是Object的规范
1. 对象的equals方法中比较的属性的值没有改变时，其返回的hashcode也不能改变。
1. 如果equals为true，则其hashCode必须相等
1. hashcode相等，equals不一定相同，因为hash存在hash碰撞

### 什么是equals
- equals是Obejct提供的通用方法，作用是比较两个实例逻辑上值是否相等，在自己设计的类中要考虑是否需要复写equlas方法，以及遵守equals方法的规范约定。

### 什么情况下需要提供equals方法？
- 如果类具有自己特有的逻辑相等的该类（不是对象等同的概念），并且它的超类也没有复写equals方法实现你的期许比较逻辑，则需要提供equals方法。如果是值类对比就要考虑提供equals方法，例如Date、Integer这种对象。

### 覆盖equals需要遵循的约定
1. 自反性：对于非null的引用值a，a.equals(a) = true。
1. 对称性：a.equals(b)于b.equals(a)，结果一样。
1. 传递性：a.equals(b) = true,b.equals(c)=true，则a.equals(c)也要为true。
1. 一致性：对于非null的引用a，b，如果equals比较的内容值没有改变，则a.equals(b)的值多次比较应该保持一致。

# 第一讲 谈谈对Java平台理解
## Java语言特性
- 书写一次，到处运行 Write once,run anywhere
- 垃圾收集 GC
- Java源代码通过JVM内嵌解释器将字节码转换成最终执行的机器代码，但是Oralce Hotsopt JVM提供JIT(Just-In-Time)编译器，部分代码在运行时将热点代码编译成机器码
# 第二讲 Exception和Error区别
均集成自Throwable类，只有Throwable类实例才可以抛出(throw)或者抛出(catch).
## Exception
- 程序正常运行中可以预料的意外情况，可能并且应该被捕获，并进行相应处理
- 分为可检查(checked）异常和不检查(unchecked)异常。可检查异常在源代码里显示捕获，属于编译期检查的一部分。不检查异常即为运行时异常，通常可以通过编码的方式避免
## Error
- 正常情况下不会出现的错误，绝大部分会导致程序处于非正常状态
## 异常类继承关系
（图）
## Exception使用注意事项
- 尽量不要捕获Exception这样的通用异常，一般情况下不要捕获Throwable或者Error
- 不要隐藏异常，需要抛出或者记录到log中
- try-catch会带来额外的性能开销，往往会影响到JVM的代码优化，捕获异常时仅捕获必要的代码段
- Java每实例化一个Exception，都会对当前栈进行快照
# 第三讲：final、finally、finalize的区别
## 区别
- final可以用来修饰类、方法、变量，修饰类表示类不可以继承拓展，修饰变量表示不可以修改，修饰方式表示不可以重写
- finally是Java保证重点代码一定被执行的一种机制。一般应用于try-finally或者try-catch-finally中关闭数据库连接，关闭文件等操作
- finalize方法是java.lang.Object的一个方法，设计目的是为了在垃圾收集前完成特定资源的回收。已不推荐使用，在JDK9中已被标记为deprecated。finalize的垃圾回收行为是不可预测的。
## 需要注意的点
- final不是immutable（不可变得），例如:
```java
final List<String> strList = new ArrayList<>();
strList.add("hello");
strList.add("world");
```
strList的引用不可以被赋值，但是strList对象的行为不受影响。

## Immutable类实现方法
- 将class自身声明为final
- 将所有成员变量定义为private和final，并且不要实现setter方法
- 构造对象时使用深度拷贝初始化
- 如果确实需要实现getter方法或者其他可能会返回内部状态的方法，使用copy-on-write原则创建私有的copy。
## finalize替代方案
使用java.lang.ref.Cleaner替换原来的finalize实现
# 第四讲 强引用、软应用、弱引用、幻想引用区别 
不同的引用类型主要体现在对象不同的可达性状态和垃圾收集的影响。
## 强引用：Strong Reference
普通对象引用，表明对象还处于活着的状态，垃圾回收器不会去回收这种对象。只要超过引用的作用域或者将强引用赋值为null，就可以被垃圾回收。
## 软应用：Soft Reference
- 一种相对弱化的引用，可以让一些对象豁免垃圾回收，只有当JVM认为内存不足时才会试图回收软引用指向的对象。
- JVM会确保抛出OutOfMemoryError之前清理软引用对象。
- 软引用通常用来实现内存敏感的缓存。
## 弱引用： Weak Reference
不能使对象豁免垃圾回收，仅提供一种访问弱引用对象的途径。可以用于构建一种没有特定约束的关系。
## 幻象引用（虚引用）
仅仅提供一种确保对象被finalize之后做某些事情的机制。可以用幻象引用监控对象的创建和销毁。
## 对象可达性状态流转分析
下图为对象生命周期和不同可达性状态，以及不同状态可能的改变关系：

![2018-12-28.21.11.01-对象创建过程.png](https://raw.githubusercontent.com/jerryjoejj/yosoro-pic/master/img/2018-12-28.21.11.01-%E5%AF%B9%E8%B1%A1%E5%88%9B%E5%BB%BA%E8%BF%87%E7%A8%8B.png)
## Java对象不同可达性级别
### 强可达：Strongly Reachable
对象可以通过一个或者多个线程可以不通过各种引用访问到的情况。
### 软可达：Softly Reachable
只能通过软应用才能访问到的对象状态
### 弱可达：Weakly Reachable
只能通过弱引用访问，十分接近finalize的状态的时机，当弱引用被清除的时候就符合finalize的条件了。
### 幻象可达：Phantom Reachable
对象引用finalize过了，只有幻象引用指向对象
### 可达性改变
- java.lang.ref.Reference#get()方法可以获取到初幻象引用外所有对象，重新指向强引用，人为改变了对象的引用状态。对于软、弱引用，垃圾收集可能存在二次确认的问题，保证弱引用状态的对象没有改变为强引用。get方法智慧返回null。
- 错误的保持了强引用（eg：赋值了static变量），对象可能没有机会变回类似弱引用的可达性状态，会产生内存泄漏。
## 引用队列(Reference Quece)使用
## 显示影响软引用垃圾收集
使用-XX:SoftRefLRUPolicyMSPerMB参数控制软引用垃圾收集，最终取决了JVM的实现。
# 第五讲 String、StringBuffer、StringBuilder区别
## 三者区别
- String是典型的Immutable类，被声明为final class，所有属性也是final的。由于它的不可变性，类似拼接、裁剪字符串的方法都将会创建新的的String对象。
- StringBulider为解决String在拼接过程中产生过多的中间对象而设计的一个类，使用append或者app方法，将字符串添加到已有序列的末尾或者指定位置。StringBuilder本质上是线程安全的可修改的字符序列。StringBuilder通过synchronized实现线程安全，性能开销大。
- StringBuffer能力上基本上和StringBuilder一样，但是StringBuffer是线程不安全的。


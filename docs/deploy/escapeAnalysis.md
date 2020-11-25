可以搜索微信公众号【Jet 与编程】查看更多精彩文章

原文发布于自己的博客平台【[http://www.jetchen.cn/EscapeAnalysis/](http://www.jetchen.cn/EscapeAnalysis/)】

----

Java 中对象的创建一般会由堆内存去分配内存空间来进行存储，在堆内存空间不足的时候，GC 便会对堆内存进行垃圾回收，如果 GC 运行的次数过多，便会影响程序的性能，所以 **“逃逸分析”** 由此诞生，它的目的就是判断哪些对象是可以存储在栈内存中而不用存储在堆内存中的，从而让其随着线程的消逝而消逝，进而减少了 GC 发生的频率，这也是常见的 JVM 优化技巧之一。

----


## 什么是逃逸分析

> “Java 中的对象是否都分配在堆内存中？”  
> ——“不尽然”

前文简单提到了，如果对象都是分配在堆内存中，那么随着对象数量的增加，必然会涉及到 GC 的频繁运行，所以为了缓解上述情况，**“逃逸分析”** 由此诞生。


逃逸分析（Escape Analysis）简单来讲就是，Java Hotspot 虚拟机可以分析新创建对象的使用范围，并决定是否在 Java 堆上分配内存的一项技术。

在方法中创建对象之后，如果这个对象除了在方法体中还在其它地方被引用了，此时如果方法执行完毕，由于该对象有被引用，所以 GC 有可能是无法立即回收的，此时便成为 **内存逃逸现象**。

> **逃逸** 是一个动词，比如 A 从 B 中逃逸，那么此时这个 A 指的就是方法中创建的对象，B 指的就是这个方法体，即可以简单理解成这个对象逃逸出这个方法体。

## 逃逸状态


一个对象有三种逃逸状态：

1. **全局逃逸**（GlobalEscape）：即一个对象的作用范围逃出了当前方法或者当前线程，  
 > 一般有以下几种场景：  
 > ① 对象是一个静态变量  
 > ② 对象是一个已经发生逃逸的对象  
 > ③ 对象作为当前方法的返回值
2. **参数逃逸**（ArgEscape）：即一个对象被作为方法参数传递或者被参数引用，但在调用过程中不会发生全局逃逸，这个状态是通过被调方法的字节码确定的。  
3. **没有逃逸**：即方法中的对象没有发生逃逸。  

``` java
public class EscapeAnalysisTest {

    public static Object globalVariableObject;

    public Object instanceObject;

    public void globalVariableEscape(){
        globalVariableObject = new Object();  // 静态变量，外部线程可见，发生逃逸
    }

    public void instanceObjectEscape(){
        instanceObject = new Object();  // 赋值给堆中实例字段，外部线程可见，发生逃逸
    }
    
    public Object returnObjectEscape(){
        return new Object();   // 返回实例，外部线程可见，发生逃逸
    }

    public void noEscape(){
        Object noEscape = new Object();   // 仅创建线程可见，对象无逃逸
    }

}
```


## 逃逸分析的优势


- 开启逃逸分析：`-XX:+DoEscapeAnalysis`  
- 关闭逃逸分析：`-XX:-DoEscapeAnalysis`  
- 显示分析结果：`-XX:+PrintEscapeAnalysis`

逃逸分析的作用，就是筛选出没有发生逃逸的对象，从而对它们进行以下三方面的优化：


#### 同步消除（锁消除）

因为同步锁是非常消耗性能的，所以当编译器确定一个对象没有发生逃逸时，它便会移除该对象的同步锁。

在 JDK1.8 中是默认开启的，但是要建立在已开启逃逸分析的基础之上。

- 开启锁消除：`-XX:+EliminateLocks`  
- 关闭锁消除：`-XX:-EliminateLocks`

#### 标量替换


首先要明白标量和聚合量，基础类型和对象的引用可以理解为标量，它们不能被进一步分解。而能被进一步分解的量就是聚合量，比如：对象。

对象是聚合量，它又可以被进一步分解成标量，将其成员变量分解为分散的变量，这就叫做标量替换。

这样，如果一个对象没有发生逃逸，那压根就不用创建它，只会在栈或者寄存器上创建它用到的成员标量，节省了内存空间，也提升了应用程序性能。

标量替换在 JDK1.8 中也是默认开启的，但是同样也要建立在已开启逃逸分析的基础之上。

- 开启标量替换：`-XX:+EliminateAllocations`  
- 关闭标量替换：`-XX:-EliminateAllocations`  
- 显示标量替换详情：`-XX:+PrintEliminateAllocations`

#### 栈内存分配

栈内存分配很好理解，在上文中提过，就是将原本分配在堆内存上的对象转而分配在栈内存上，这样就可以减少堆内存的占用，从而减少 GC 的频次。


## 逃逸分析测试

代码如下，大致思路就是 for 循环 1 亿次，循环体内调用外部的 allot() 方法，而 allot() 方法的作用就是简单创建一个对象，但是这个对象是内部的，所以是未逃逸的，所以理论上 JVM 是会进行优化的，我们拭目以待。并且我们会对比开启和关闭逃逸分析之后各自程序的运行时间：

``` java
/**
 * @ClassName: EscapeAnalysisTest
 * @Description: http://www.jetchen.cn 逃逸分析 demo
 * @Author: Jet.Chen
 * @Date: 2020/11/23 14:26
 * @Version: 1.0
 **/
public class EscapeAnalysisTest {

    public static void main(String[] args) {
        long t1 = System.currentTimeMillis();
        for (int i = 0; i < 100000000; i++) {
            allot();
        }
        long t2 = System.currentTimeMillis();
        System.out.println(t2-t1);
    }

    private static void allot() {
        Jet jet = new Jet();
    }

    static class Jet {
        public String name;
    }

}
```

上面就是我们进行逃逸分析测试的代码， mian() 方法末尾有一个线程暂停，目的是为了观察此时 JVM 中的内存情况。

#### Step 1：测试开启逃逸

由于环境是 jdk1.8，默认开启了逃逸分析，所以直接运行，得到结果如下，程序耗时 3 毫秒：

![](http://blogsource.chenkaikai.com/uploads/2020/11/EscapeAnalysis07.png)

此时线程是处于睡眠状态的，我们观察下内存情况，发现堆内存中一共新建了 11 万个 Jet 对象。

![](http://blogsource.chenkaikai.com/uploads/2020/11/EscapeAnalysis08.png)

#### Step 2：测试关闭逃逸

我们关闭逃逸分析再来运行一次（使用 `java -XX:-DoEscapeAnalysis EscapeAnalysisTest` 来运行代码即可），得到结果如下，程序耗时 400 毫秒：

![](http://blogsource.chenkaikai.com/uploads/2020/11/EscapeAnalysis09.png)

此时我们观察下内存情况，发现堆内存中一共新建了 3 千多万个 Jet 对象。

![](http://blogsource.chenkaikai.com/uploads/2020/11/EscapeAnalysis10.png)

所以，无论是从代码的执行时间（3 毫秒 VS 400 毫秒），还是从堆内存中对象的数量（11 万个 VS 3 千万个）来分析，在上述场景下，开启逃逸分析是有正向益的。


#### Step 3：测试标量替换

我们测试下开启和关闭 **标量替换**，如下图：

![](http://blogsource.chenkaikai.com/uploads/2020/11/EscapeAnalysis11.png)

由上图我们可以看出，在上述极端场景下，开启和关闭标量替换对于性能的影响也是满巨大的，另外，同时也验证了标量替换功能生效的前提是逃逸分析已经开启，否则没有意义。

#### Step 4：测试锁消除

测试锁消除，我们需要简单调整下代码，即给 allot() 方法中的内容加锁处理，如下：

``` Java
private static void allot() {
    Jet jet = new Jet();
    synchronized (jet) {
        jet.name = "jet Chen";
    }
}
```

然后我们运行测试代码，测试结果也很明显，在上述场景下，开启和关闭锁消除对程序性能的影响也是巨大的。

![](http://blogsource.chenkaikai.com/uploads/2020/11/EscapeAnalysis12.png)

## 总结

逃逸分析的原理理解起来其实很简单，但 JVM 在实际应用过程中，还是有诸多因素需要考虑的。

比如，逃逸分析不能在静态编译时进行，必须在 JIT 里完成。原因大致是：与 Java 的动态性有冲突。因为你可以在运行时，通过动态代理改变一个类的行为，此时，逃逸分析是无法得知类已经变化了。总之就是：因为只有当收集到足够的运行数据时，JVM 才可以更好地判断对象是否发生了逃逸。（参考大佬的解释：[https://www.zhihu.com/ques....](https://www.zhihu.com/question/27963717)）

当然，逃逸分析并不是没有劣势的，因为逃逸分析是需要消耗一定的性能去执行分析的，所以说如果方法中的对象全都是处于逃逸状态，那么就没有起到优化的作用，从而就白白损失了这部分的性能消耗。


![公众号](http://blogsource.chenkaikai.com/cli_500px.png)
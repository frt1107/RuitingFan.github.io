---
layout: post
title:  "sychronized实现原理浅析"
date:   2020-06-28 10:57:01 +0800
categories: JAVA
tag: JAVA



---


* content
{:toc}


## 1. Sychronized用法

### 简介

Sychronized是Java并发中的一种用法，可看作一种悲观锁实现共享资源的互斥访问。

其作用主要有3个：

1. **原子性：**确保线程互斥访问同步代码；
2. **可见性：**保证共享变量的修改能及时可见，其实是通过Java内存模型中的 **“对一个变量unlock操作之前，必须要同步到主内存中；如果对一个变量进行lock操作，则将会清空工作内存中此变量的值，在执行引擎使用此变量前，需要重新从主内存中load操作或assign操作初始化变量值**” 来保证的；
3. **有序性：**有效解决重排序问题，即 “**一个unlock操作先行发生(happen-before)于后面对同一个锁的lock操作**”。

### 使用

**Sychronized可以把任意一个非null对象作为“锁”**，在HotSpot JVM中，被称作**对象监视器**（Object Monitor）。（之前看《疯狂Java讲义》，也称其为监视器）

其用法主要有3种：

1. **修饰实例方法：**监视器锁即是对象实例（this）；
2. **修饰静态方法：**监视器锁即是对象的Class实例，因为Class数据存在于永久代，因此静态方法锁相当于该类的一个全局锁；
3. **修饰代码块：**监视器锁即是括号括起来的对象实例。

synchronized 内置锁 是一种 对象锁（锁的是对象而非引用变量），作用粒度是对象 ，可以用来实现对 临界资源的同步互斥访问 ，是 可重入 的。其可重入最大的作用是避免死锁，如：子类同步方法调用了父类同步方法，如没有可重入的特性，则会发生死锁。

## 2. Sychronized原理

测试代码：

```java
package sychronizedtt;

public class TestSychronized implements Runnable {
    static final TestSychronized instance = new TestSychronized();
    static int i = 0;

    public TestSychronized() {
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
    }

    public void run() {
        // 修饰代码块
        synchronized(instance) {
            for(int j = 0; j < 100; ++j) {
                ++i;
            }

        }
    }

    // 修饰方法
    public synchronized void setI(int i) {
        TestSychronized.i = i;
    }
}
```

查看JVM字节码信息，步骤如下：

```
1. javac xx.java	//编译指定类
2. javap -v xx.class	//查看JVM反编译信息
```

测试代码的JVM字节码信息如下：

```
public class sychronizedtt.TestSychronized implements java.lang.Runnable
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #11.#35        // java/lang/Object."<init>":()V
   #2 = Fieldref           #9.#36         // sychronizedtt/TestSychronized.b:I
   #3 = Class              #37            // java/lang/Thread
   #4 = Fieldref           #9.#38         // sychronizedtt/TestSychronized.instance:Lsychronizedtt/TestSychronized;
   #5 = Methodref          #3.#39         // java/lang/Thread."<init>":(Ljava/lang/Runnable;)V
   #6 = Methodref          #3.#40         // java/lang/Thread.start:()V
   #7 = Methodref          #3.#41         // java/lang/Thread.join:()V
   #8 = Fieldref           #9.#42         // sychronizedtt/TestSychronized.i:I
   #9 = Class              #43            // sychronizedtt/TestSychronized
  #10 = Methodref          #9.#35         // sychronizedtt/TestSychronized."<init>":()V
  #11 = Class              #44            // java/lang/Object
  #12 = Class              #45            // java/lang/Runnable
  #13 = Utf8               instance
  #14 = Utf8               Lsychronizedtt/TestSychronized;
  #15 = Utf8               i
  #16 = Utf8               I
  #17 = Utf8               b
  #18 = Utf8               <init>
  #19 = Utf8               ()V
  #20 = Utf8               Code
  #21 = Utf8               LineNumberTable
  #22 = Utf8               main
  #23 = Utf8               ([Ljava/lang/String;)V
  #24 = Utf8               Exceptions
  #25 = Class              #46            // java/lang/InterruptedException
  #26 = Utf8               run
  #27 = Utf8               StackMapTable
  #28 = Class              #44            // java/lang/Object
  #29 = Class              #47            // java/lang/Throwable
  #30 = Utf8               setb
  #31 = Utf8               (I)V
  #32 = Utf8               <clinit>
  #33 = Utf8               SourceFile
  #34 = Utf8               TestSychronized.java
  #35 = NameAndType        #18:#19        // "<init>":()V
  #36 = NameAndType        #17:#16        // b:I
  #37 = Utf8               java/lang/Thread
  #38 = NameAndType        #13:#14        // instance:Lsychronizedtt/TestSychronized;
  #39 = NameAndType        #18:#48        // "<init>":(Ljava/lang/Runnable;)V
  #40 = NameAndType        #49:#19        // start:()V
  #41 = NameAndType        #50:#19        // join:()V
  #42 = NameAndType        #15:#16        // i:I
  #43 = Utf8               sychronizedtt/TestSychronized
  #44 = Utf8               java/lang/Object
  #45 = Utf8               java/lang/Runnable
  #46 = Utf8               java/lang/InterruptedException
  #47 = Utf8               java/lang/Throwable
  #48 = Utf8               (Ljava/lang/Runnable;)V
  #49 = Utf8               start
  #50 = Utf8               join
{
  static final sychronizedtt.TestSychronized instance;
    descriptor: Lsychronizedtt/TestSychronized;
    flags: ACC_STATIC, ACC_FINAL

  static int i;
    descriptor: I
    flags: ACC_STATIC

  int b;
    descriptor: I
    flags:

  public sychronizedtt.TestSychronized();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: iconst_0
         6: putfield      #2                  // Field b:I
         9: return
      LineNumberTable:
        line 3: 0
        line 6: 4

  public static void main(java.lang.String[]) throws java.lang.InterruptedException;
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=3, args_size=1
         0: new           #3                  // class java/lang/Thread
         3: dup
         4: getstatic     #4                  // Field instance:Lsychronizedtt/TestSychronized;
         7: invokespecial #5                  // Method java/lang/Thread."<init>":(Ljava/lang/Runnable;)V
        10: astore_1
        11: new           #3                  // class java/lang/Thread
        14: dup
        15: getstatic     #4                  // Field instance:Lsychronizedtt/TestSychronized;
        18: invokespecial #5                  // Method java/lang/Thread."<init>":(Ljava/lang/Runnable;)V
        21: astore_2
        22: aload_1
        23: invokevirtual #6                  // Method java/lang/Thread.start:()V
        26: aload_2
        27: invokevirtual #6                  // Method java/lang/Thread.start:()V
        30: aload_1
        31: invokevirtual #7                  // Method java/lang/Thread.join:()V
        34: aload_2
        35: invokevirtual #7                  // Method java/lang/Thread.join:()V
        38: return
      LineNumberTable:
        line 8: 0
        line 9: 11
        line 10: 22
        line 11: 26
        line 12: 30
        line 13: 34
        line 14: 38
    Exceptions:
      throws java.lang.InterruptedException

  public void run();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=4, args_size=1
         0: getstatic     #4                  // Field instance:Lsychronizedtt/TestSychronized;
         3: dup
         4: astore_1
         5: monitorenter
         6: iconst_0
         7: istore_2
         8: iload_2
         9: bipush        100
        11: if_icmpge     28
        14: getstatic     #8                  // Field i:I
        17: iconst_1
        18: iadd
        19: putstatic     #8                  // Field i:I
        22: iinc          2, 1
        25: goto          8
        28: aload_1
        29: monitorexit
        30: goto          38
        33: astore_3
        34: aload_1
        35: monitorexit
        36: aload_3
        37: athrow
        38: return
      Exception table:
         from    to  target type
             6    30    33   any
            33    36    33   any
      LineNumberTable:
        line 18: 0
        line 19: 6
        line 20: 14
        line 19: 22
        line 22: 28
        line 23: 38
      StackMapTable: number_of_entries = 4
        frame_type = 253 /* append */
          offset_delta = 8
          locals = [ class java/lang/Object, int ]
        frame_type = 250 /* chop */
          offset_delta = 19
        frame_type = 68 /* same_locals_1_stack_item */
          stack = [ class java/lang/Throwable ]
        frame_type = 250 /* chop */
          offset_delta = 4

  public synchronized void setb(int);
    descriptor: (I)V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: iload_1
         2: putfield      #2                  // Field b:I
         5: return
      LineNumberTable:
        line 26: 0
        line 27: 5

  static {};
    descriptor: ()V
    flags: ACC_STATIC
    Code:
      stack=2, locals=0, args_size=0
         0: new           #9                  // class sychronizedtt/TestSychronized

```

**修饰代码块：**

JVM字节码信息包括1个monitorenter和2个monitorexit：

1. monitorenter：每个对象都是一个监视器锁（monitor）。当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权，过程如下：

   ```
   a. 如果monitor的进入数为0，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者；
   b. 如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1；
   c. 如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权。
   ```

2. monitorexit：执行monitorexit的线程必须是objectref所对应的monitor的所有者。指令执行时，monitor的进入数减1，如果减1后进入数为0，那线程退出monitor，不再是这个monitor的所有者。其他被这个monitor阻塞的线程可以尝试去获取这个 monitor 的所有权。

monitorexit指令出现了2次：第1次是同步代码块正常释放锁的一个标志；如果同步代码块中出现Exception或者Error，则会调用第二个monitorexit指令来保证释放锁。

通过上面两段描述，我们应该能很清楚的看出Synchronized的实现原理，Synchronized的语义底层是通过一个monitor的对象来完成，其实wait/notify等方法也依赖于monitor对象，这就是为什么只有在同步的块或者方法中才能调用wait/notify等方法，否则会抛出java.lang.IllegalMonitorStateException的异常的原因。

------

**修饰方法：**

相对于普通方法，其常量池中多了 `ACC_SYNCHRONIZED` 标示符。JVM就是根据该标示符来实现方法的同步的：

```
当方法调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先获取monitor，获取成功之后才能执行方法体，方法执行完后再释放monitor。在方法执行期间，其他任何线程都无法再获得同一个monitor对象。
```



题外话：

sychronized底层原理应该比这些还要复杂，这只是初步的浅析，后续有时间还会继续钻研。

------

Java的书我目前有四五本，入门的书、程序员皆知的从入门到静通、疯狂Java讲义、effective Java等。

入门的书太浅了，基本是语法类的，对于Java程序员而言完全不够。

之前主要阅读教材是李刚的《疯狂Java讲义》，以为很厚一本书应该覆盖内容比较全，可能讲的也会深一些，但实际上只能说Java知识点确实很多，现在工作之后发现这本书确实讲的还是比较浅的内容，基本停留在使用层面，原理层面知识也有但相对较少，如果要理解原理性知识还需要另找其他途径。

《effective Java》是effective系列的，effective系列书籍基本都是与语言特性相关的使用的小trick，分条呈现，effective系列口碑都很好，最近也在看，但是是英译中的，翻译的有点像文言文，很多内容阅读起来都比较吃力。

所以目前阶段还是清扫知识盲区，不会的或者深入的知识还是在网上找博客，这样知识就比较不成体系，也有弊端。最近也一直在纠结和思考学习的方式方法，但是只要学习就能进步，“越努力，越幸运”！


title: JAVA 基础
date: 2018-11-30 14.00
tags:

------
### [JAVA 内存](https://github.com/francistao/LearningNotes/blob/master/Part1/Android/Android%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F%E6%80%BB%E7%BB%93.md)
  - 静态存储区：存放静态数据，全局 static 数据和常量，编译器已经分配内存并且在程序运行期间都存在。
  - 栈区：存放方法内部的局部变量，在方法执行结束后局部变量的内存就被释放，效率高但是分配内存容量有限。
  - 堆区：存放 new 出来的对象实例，这个实例不使用时就会被 GC 回收。
  ```JAVA
  public class Sample {
    //s1 和 0 内存都存放在堆中
      int s1 = 0;
      //mSample1 和 mSample1 所指的对象内存都存放在堆中
      Sample mSample1 = new Sample();

      public void method() {
          // s2 存在于栈中
          int s2 = 1;
          //引用 mSample2 存放于栈中，但 mSample2 指的对象内存存放于堆中
          Sample mSample2 = new Sample();
      }
  }
  //引用 mSample3存放于栈中，但是 mSample3 所指对象存放于堆中，包括对象内部的成员变量 s1，mSample1 都在堆中
  Sample mSample3 = new Sample();
  ```
  **结论：局部变量的基本数据类型和引用存放于栈，引用所指的对象和类的成员变量存放于堆中（包括引用和引用指的对象）。**
  - Java 的内存管理
    1. 内存的分配由程序员来完成，即程序调用 new 方法或通过反射来分配内存。
    2. 内存的回收是由垃圾回收器（GC）完成的，对象不再被引用，该对象的内存就被释放回收。
  - JAVA 内存泄漏
    1. 简单介绍：指的是在程序中，存在相对程序逻辑来说已经无用的对象，但是由于对象一直被引用者导致无法被垃圾回收器释放回收，这些对象会在程序运行期间一直占着内存。
    2. **内存泄漏的原因**
      - **静态集合** 引用的所有对象 Object 在程序运行期间都无法释放。
      - 集合里面的对象属性被修改后再调用 remove() 方法无效（如 hashcode 不同无法 remove）。
      - **监听器没有释放** ，导致该监听的实体没有被释放。
      - **数据库连接，网络连接和 IO 连接** ，即涉及流操作都没有显式调用 close() 方法，GC 不会回收。
      - **内部类和外部模块的引用**，非静态的内部类和匿名类会隐式地持有一个他们外部类的引用，当外部类对象已经相对程序无作用时，非静态得内部类和匿名类引用着外部类，导致外部类对象无法被回收释放。
      - **单例模式** ， 单例模式在初始化后在程序运行期间都是存在的，如果单例模式持有外部对象得引用，则会导致外部对象无法释放回收。
---
### JAVA 集合
- Map
  主要的实现类有：HashMap， HashTable， LinkedHashMap，TreeMap。
  1. HashMap根据 Key 的 HashCode 来存储数据，只允许一个 Key 为 Null，但允许多条记录为 Null，HashMap 线程不安全。可以使用 Collections.synchronizedMap() 方法来使HashMap 具有线程安全，或者直接使用 **ConcurrentHashMap** 代替。
  2. HashTable 和 HashMap 的功能差不多，但是它是性能安全的。
  3. LinkedHashMap 是 HashMap 的子类，保存插入顺序，也就是说它是有序的，可以通过 Iterator 迭代器遍历 LinkedHashMap。
  4. TreeMap实现 SortedMap 接口，内部有序，顺序是按照 Key 的升序排序，也可以自己实现排序的比较器（Comparable 接口）。用 Iterator 遍历也是有序的。
- [HashMap](https://tech.meituan.com/java_hashmap.html)
  1.  HashMap 按照 **Lazy-Load** 来进行初始化，通过 resize() 来进行初始化一个 HashMap 和对 HashMap 进行扩容。
  2. 键值对在 HashMap 内的位置 index 取决于位运算 **i = (n -1) & hash（n 为 table 长度）**;
  3. 如果内存空间很多而又对时间效率要求很高，可以降低负载因子 Loadfactor 的值，门限值降低，table 数组就容易扩容，但是每个数组上的链表就会相对短，查询就快；相反，如果内存空间紧张而对时间效率要求不高，可以增加负载因子 loadFactor 的值，（loadFactor 可以大于1，但是建议不超过 0.75，默认值 0.75 符合大多数场景）。
  4. 而当链表长度太长（ **默认超过8** ）时，链表就转换为 **红黑树** ，利用红黑树快速增删改查的特点提高 HashMap 的性能。
  5. **门限值（threshold） = 负载因子(loadfactor) * 容量（capacity）**，在元素个数超过门限值时（size > threshold），调整 Map 的大小，table 的长度使用的是 2 次幂扩展(指长度扩为原来 2 倍)，同理根据门限值的计算公式可以计算出门限值也是原来 2 倍。 **扩容非常耗性能，所以在使用 HashMap 时最好给一个大致的 size，避免频繁扩容**。
  6. 扩容对应的键的 hash 计算
    - jdk 7：重新计算 hash 值来获取索引值。
    - jdk 8：只需要看看原来的hash值新增的那个bit是1还是0就好了，是 **0** 的话索引 **不变** ，是 **1** 的话索引变成 **“原索引+oldCap”**

    ![img](https://github.com/0HongTao0/Blog/blob/master/pic/HashMap_%E4%B8%8D%E5%90%8CJDK%E6%89%A9%E5%AE%B9%E6%AF%94%E8%BE%83.png?raw=true)
  7. 线程安全问题：
    HashMap 是线程不安全的，为什么呢？
    并发的多线程使用场景中使用 HashMap 可能造成死循环，因为在 resize 过程中会导致出现环形链表。
---
### JAVA 并发

- 如何创建多线程？

  主要有 2 种方法

  1. 创建一个子类继承（extends） Thread 类，然后重写 run（）方法。
  2. 创建一个子类实现（implement）Runnable 接口，然后重写 run（）方法。

- 简述线程的五种状态

  1. 线程的五种状态分别是：新建（New），就绪（Runnable），运行（Running），终止（Dead），阻塞（Blocked）

  2. 线程没被阻塞从新建状态到终止状态的过程：从线程被 new 创建开始，线程就进入**新建**状态，当程序调用 Thread.start() 方法的时候，线程进入**就绪**状态，因为 JAVA 线程的调度是抢占式调度，所以在这个时候线程有可能正在运行也可能没有运行，当操作系统分配时间片给线程的时候，线程进入**运行**状态，当 Thread.run() 方法执行到最后一条语句并返回的时候，线程自然死亡，进入**终止**状态。

  3. 线程被阻塞即从**就绪**状态转至**阻塞**状态，导致线程进入阻塞状态的原因主要有 3 个，**同步阻塞**，当线程获取一个被其他线程所持有的对象锁，线程进入阻塞状态。线程执行 **Thread.sleep()** 方法，以及 **Thread.wait()** 方法。当线程在阻塞状态被激活的时候，线程进入**就绪**状态，等待系统调度进入**运行**状态。

     ![img](https://upload-images.jianshu.io/upload_images/5545289-1c4500c4a5adaafe.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/527/format/webp)

- 如何中断正在执行的线程

  1. 执行完 run（）方法的最后一条语句并由 return 返回时，线程终止。
  2. 当线程run（）方法种出现了没有捕获分异常时，线程终止。

  没有方法可以强制终止线程，但是可以用 Thread.interrupt() 方法来请求终止线程。线程有个中断状态的 Boolean 标志，不停地检查该标志，线程阻塞无法检查中断状态所以不能在线程阻塞的时候调用 Thread.interrupt() 方法（会抛出 InterruptedException）。

- 锁的相关理解

  1. 锁是防止程序代码块中受并发访问而造成不安全的干扰，即保证任何时刻只能有一个线程执行被保护的代码。
  2. 大多数情况下是通过 synchronized 关键字来实现锁，在 JAVA 中每一个对象都存在一个锁，**synchronized 用在方法上**，是锁住当前对象的当前方法，多个线程访问这个对象的该方法会被阻塞；**synchronized(this)** 锁住的是当前对象的，多个线程访问这个对象会被阻塞；同理可得 **synchronized(object)** 锁的是 object 对象，多个线程访问该对象会被阻塞；（注意：锁的对象不能是 final 类，因为经过修改的 final 类会是一个新对象，无法获取同一个锁）
  3. 也可以通过 JAVA 提供的 Lock 类来实现锁。

---
### JAVA 代理
  - 静态代理：代码 **运行之前** 代理类的 Class 编译文件就已经存在。
  实现思路：通过代理者和被代理者实现同一接口，并且代理者种存在一个被代理者的实例，即调用方通过代理者间接调用被代理者的方法。
  - 动态代理：代码 **运行过程** 通过反射机制直接生成代理者对象。
  实现思路：通过 java.lang.reflect 的 Proxy 实现，有 3 个参数。
  ClassLoder: 类加载器
  Class<?>[]: 对象组
  InvocationHandler: **调用处理器,无论何时调用代理对象的方法调用处理器的 invoke 方法都会被调用。**
  ```java
    public static Object newProxyInstance(ClassLoader loader,Class<?>[] interfaces, InvocationHandler h){}

    public class DynamicProxy implements InvocationHandler{
      private Object obj;//被代理的类引用

      public DynamicProxy(Object obj){
        this.obj = obj;
      }

      @Override
      public Object invoke(Object proxy, Method method, Object[] args) throw Throwable{
        Object result = method.invoke(obj, args);
        return result;
      }
    }
  ```

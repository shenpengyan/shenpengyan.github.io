---
layout:     post
title:      "面试投行的20个Java问题"
subtitle:   "top-20-java-interview-questions-with-answers"
date:       2018-11-07 12:00:00
author:     "沈歌"
header-img: "img/home-bg-o.jpg"
tags:
        - 面试
---


> 如果你需要准备面试，可以看一下这篇博客中20个为Java开发人员准备的面试投行的问题。

大量的Java开发人员面试例如巴克莱银行(Barclays)、瑞士信贷集团(Credit Suisse)、花旗银行(Citibank)这样的投行的Java开发岗位，但是大多数人都不知道会被问什么问题。

这篇文章中，我将分享一些对于3年经验以上的程序员会被问的最多的问题。

对于两年及两年以下Java开发经历的人，投行一般不会通过社招招聘，一般只有可能在毕业时候通过校招进去。

实际面试的时候并不保证一定会被问到这些问题，而且实际上，大概率问不到，但是通过这篇文章你能够知道大概会被问什么类型的问题。而且你准备的越充分，面试的时候表现的会越好。

另外，如果这20个问题你觉得不够的话，可以看两篇文章：[电话面试的40个Java面试问题](http://www.java67.com/2015/03/top-40-core-java-interview-questions-answers-telephonic-round.html) 和 [200+ Java 面试问题](http://bit.ly/2CupaSL)。

话不多说，进入正题。接下来我们开始看我从朋友和大学同学那里收集到的他们面试投行遇到的问题。

### Java程序员投行面试问题

#### 问题1： 多线程环境下使用`HashMap`有什么问题，什么时候使用`get()`方法会进入死循环？

**答**：没什么问题，会不会出问题取决于你怎么用。例如，如果你在一个线程内[初始化一个HashMap](http://www.java67.com/2016/01/how-to-initialize-hashmap-with-values-in-java.html)，所有线程只是读取数据，那么没什么问题。例如`Map`
包含配置信息，服务启动就不会更改。真正有问题的情况下是至少一个线程对`HashMap`做了改动，例如：增加、更新或者移除任何的键值对。因为`put()`操作会引起re-sizing，有可能导致死循环，所以应该使用[Hashtable](http://javarevisited.blogspot.com/2012/01/java-hashtable-example-tutorial-code.html)或者[ConcurrentHashMap](http://javarevisited.blogspot.com/2013/02/concurrenthashmap-in-java-example-tutorial-working.html),后面这个更好一些。

#### 问题2：如果你不重写`hashCode()`方法，会有什么后果吗？


**答**：这是一个好问题。根据我得理解，一个差的hashcode方法会导致[HashMap的频繁碰撞](http://javarevisited.blogspot.sg/2016/01/how-does-java-hashmap-or-linkedhahsmap-handles.html), 然后导致往`hashMap`中添加一个对象的时候耗时增加。

从[Java 8](https://javarevisited.blogspot.com/2018/08/top-5-java-8-courses-to-learn-online.html)开始, key碰撞比之前Java版本的Key碰撞对性能影响要小一些，在大于某一个阈值后，[二叉树](http://www.java67.com/2016/08/binary-tree-inorder-traversal-in-java.html)会取代[链表](http://javarevisited.blogspot.sg/2017/07/top-10-linked-list-coding-questions-and.html#axzz4xXS86IVo)，链表最坏情况下`O(n)`的性能问题会减少到二叉树的`O(logN)`。

#### 问题3：Java里面所有的不变的属性需要设置为final吗？

**答**：没有必要，你可以实现相同的功能通过以下操作：设为非final的private 变量，且只有在构造函数中才能修改。不设set方法，如果是一个可变对象，不要泄露任何指向这个对象的引用。

[设置一个引用变量为final](https://javarevisited.blogspot.com/2016/09/21-java-final-modifier-keyword-interview-questions-answers.html) 只能确保这个变量不会被赋予一个不同的引用，但是你仍然可以改变引用变量的属性值。

这是面试官想要听到的一个点。如果你想要知道更多Java中引用变量的知识，推荐加入Udemy的课程[Complete Java Masterclass](https://click.linksynergy.com/fs-bin/click?id=JVFxdTr9V80&subid=0&offerid=323058.1&type=10&tmpid=14538&RD_PARM1=https%3A%2F%2Fwww.udemy.com%2Fjava-the-complete-java-developer-course%2F)

#### 问题4：String的substring()的实现原理

**答**：substring取原来string的一部分创建一个新的对象。这个问题主要想问的是开发者是否熟悉substring可能导致的[内存泄露](https://pluralsight.pxf.io/c/1193463/424552/7490?u=https%3A%2F%2Fwww.pluralsight.com%2Fcourses%2Fjava-understanding-solving-memory-problems)风险。

直到Java1.7， substring 拥有原来的字符数组的引用，这意味着即使是五字符这么小的字符串，也可能会导致一个1GB字符数组无法被垃圾回收因为有一个强引用。

这个问题在Java1.7中已经被修复，原来的字符数组不会被引用，但是会导致创建substring耗时会有点长，以前时间复杂度是`O(1)`, Java 7之后时间复杂度是`O(n)`。

#### 问题5：写一个单例模式的临界区代码([答案](http://javarevisited.blogspot.sg/2014/05/double-checked-locking-on-singleton-in-java.html))

**答**: 这个问题实际上是想让候选人写一个[双重校验锁](http://www.java67.com/2015/09/thread-safe-singleton-in-java-using-double-checked-locking-pattern.html)。

记得使用[volatile变量](http://javarevisited.blogspot.sg/2011/06/volatile-keyword-java-example-tutorial.html)
确保单例[线程安全](http://www.java67.com/2016/04/why-double-checked-locking-was-broken-before-java5.html)

这是使用双重校验锁写的线程安全的单例代码：

```java
public class Singleton {
    private static volatile Singleton _instance;
    /**
     * Double checked locking code on Singleton
     * @return Singelton instance
     */
    public static Singleton getInstance() {
        if (_instance == null) {
            synchronized (Singleton.class) {
                if (_instance == null) {
                    _instance = new Singleton();
                }
            }
        }
        return _instance;
    }
}
```

与此同时，最好能够知道典型的设计模式，比如单例模式、工厂模式、装饰模式等，如果你对这个感兴趣，[Design Pattern library](https://pluralsight.pxf.io/c/1193463/424552/7490?u=https%3A%2F%2Fwww.pluralsight.com%2Fcourses%2Fpatterns-library) 这个不错。

![](https://cdn-images-1.medium.com/max/906/1*MTeZ0WlAo-_9AAQrGS-CCw.jpeg)

#### 问题6：Java中如何处理写存储过程或者读存储过程时遇到的错误？

**答**： 这是Java面试中最难的问题之一。我的回答是一个存储过程应该在操作错误时返回错误码，但是如果存储过程本身出问题，捕获`SQLException` 是唯一选择。

[Effective Java：3rd Edition](https://www.amazon.com/Effective-Java-3rd-Joshua-Bloch/dp/0134685997/?tag=javamysqlanta-20) 中对于Java的异常和捕获有很多好的建议，值得一读。

![image](https://cdn-images-1.medium.com/max/906/1*p_TBjBE24E87PMZuv_koUA.png)

#### 问题7：`Executor.submit()`和`Executer.execute()`有什么区别？

**答** 这个面试问题来自于我的这篇文章[50个多线程面试问题](http://javarevisited.blogspot.sg/2014/07/top-50-java-multithreading-interview-questions-answers.html#axzz4jaJmaqbE)。随着对于Java开发人员的并发技能要求的增加，
这个题目越来越受欢迎。

答案是，前者返回一个`Future`对象，可以用于找到工作线程的运行结果。

在异常处理上也不一样，在任务抛出异常时，如果是通过`execute()`提交的，会抛出无需捕获的异常（如果你没有特殊处理，会打印错误栈道System.err）。如果是通过`submit()`提交的，任何异常，无论是不是checked exception，都是返回的一部分，Future.get将把异常包在`ExecutionExeption`中，向上层抛出。

如果你想学习任何Future, Callable,异步计算和提高并发编程技巧，建议你学习这个课程[Java Concurrency Practice in Bundle](https://learning.javaspecialists.eu/courses/concurrency-in-practice-bundle?affcode=92815_johrd7r8).

这是一个基于[Brian Goetz](https://medium.com/@briangoetz)独立编写的[并发编程实践](http://www.amazon.com/dp/0321349601/?tag=javamysqlanta-20) 的高级课程。这个课程绝对对得起你付出的钱和时间。并发编程很难而且有很多技巧，书和课程结合起来一起学是不错的方式。

![image](https://cdn-images-1.medium.com/max/906/1*9d4d_Pne_2jSwdCi9aG9NQ.png)

#### 问题8： 工厂模式和抽象工厂模式有什么区别？([答案](http://javarevisited.blogspot.sg/2013/01/difference-between-factory-and-abstract-factory-design-pattern-java.html))

**答**：抽象工场模式提供一个多层级的抽象。考虑不同的工厂继承自同一个抽象工厂，代表基于工厂的不同对象结构的创建，例如，`AutomobileFactory,UserFactory, RoleFactory`等都继承自`AbstractFactory`。每一个独立的工厂代表那种类型物体的创造器。

如果你想要学习更多关于抽象工厂设计模式，我建议你看[Java设计模式](https://click.linksynergy.com/fs-bin/click?id=JVFxdTr9V80&subid=0&offerid=323058.1&type=10&tmpid=14538&RD_PARM1=https%3A%2F%2Fwww.udemy.com%2Fdesign-patterns-java%2F) 这个课程，提供了优秀的真实案例帮你更好的理解设计模式。

这里是一个工厂模式和抽象工厂模式的UML图：

![image](https://cdn-images-1.medium.com/max/906/1*zoSC-TjwFjirb_ShdCnfYQ.jpeg)

如果你想要更多选择，也可以看这个课程:[5个设计模式](https://javarevisited.blogspot.com/2018/02/top-5-java-design-pattern-courses-for-developers.html)


#### 问题9： 什么是单例？整个方法使用`synchronized`和只有临界区使用`synchronized`哪个好？([答案](http://javarevisited.blogspot.com/2012/12/how-to-create-thread-safe-singleton-in-java-example.html))

**答**：Java中的单例是指在整个Java应用中一个类只有一个实例。例如, `java.lang.Runtime`是一个单例类。

创建一个单例在Java 4之前非常难，但是自从Java 5 引入了[枚举：enum](https://javarevisited.blogspot.com/2011/08/enum-in-java-example-tutorial.html), 它变得非常容易了。

你可以看我的这篇文章[如何在Java中创建线程安全的单例](http://javarevisited.blogspot.sg/2012/12/how-to-create-thread-safe-singleton-in-java-example.html)，这里使用了枚举和双重校验锁的方式，这个题主要就是想问这个。

#### 问题10： 你能基于Java 4，Java5里面的HashMap如何迭代取值？

这个问题有点棘手，但是一般是使用`while`或者`for`循环。Java里面迭代Map有四种方式。一种是使用`keySet()`，迭代每一个key的时候使用`get()`方法去取value，但是有点慢。第二种方法是使用`entrySet()`。然后使用`for each`循环或者`Iterator.hashNext()`方法来迭代取值即可。 （keySet， entrySet和foreach， Iterator进行组合，所以是4种。）

这个方法比较好，因为在每次迭代时，`key` 和 `value` 都已经取出来了，你不需要调用`get()`方法去取value，使用`get()`方法当你从一个桶里面的大的链表中取数据，时间复杂度是O(n)。

你可以在我的博客[4种方法迭代Java Map](http://javarevisited.blogspot.com/2011/12/how-to-traverse-or-loop-hashmap-in-java.html) 中查看细节和示例代码。


#### 问题11：什么时候重写 `hashCode()` 和 `equals()` 方法？（[答案](http://javarevisited.blogspot.com/2013/08/10-equals-and-hashcode-interview.html)）

**答**：当你需要的时候，尤其是你想要通过业务逻辑校验两个对象是否相等，而不是通过两个对象是否执行同一地址。例如两个员工对象在`emp_id` 相等的时候相等，即使它们是通过不同的代码创建出来的两个不同对象。

另外，如果你使用一个对象作为`HashMap`的key，你必须[重写](http://www.java67.com/2013/04/example-of-overriding-equals-hashcode-compareTo-java-method.html)这两个方法。

作为java equals-hashcode约束的一部分，你当你重写equals的时候，你必须重写hashcode. 否则你不能再Set，Map这样的类里面使用，因为他们一来于`equals()`方法来保证逻辑正确性。

你也可以看我的这篇文章看理解重写这两个方法可能导致的问题: [java equals中的5个技巧](http://javarevisited.blogspot.com/2011/02/how-to-write-equals-method-in-java.html)

#### 问题12：在重写`hashCode()`方法的时候你遇到哪些问题？

如果你不重写equals方法，equals和hashcode中的约束不会生效。根据该约束，两个对象通过`equals()`相等，一定有相同的hashcode。

在这种情况下，另一个对象可能返回一个不同的`hashCode`并存储在该位置，将破坏[HashMap类](http://www.java67.com/2013/02/10-examples-of-hashmap-in-java-programming-tutorial.html)的不可变，因为它不支持重复的key。

当你使用`put()`方法添加对象时，它迭代之前在map中那个桶位置的所有的`Map.Entry`对象，并且更新到新值。如果Map已经包含了那个Key，如果hashCode没有重写，这个机制不会起作用。

如果你想要学习更多关于Java集合（Map, Set）中`equals()`和`hashCode()`方法的作用，建议你看一下这个课程[Java基础：集合](https://pluralsight.pxf.io/c/1193463/424552/7490?u=https%3A%2F%2Fwww.pluralsight.com%2Fcourses%2Fjava-fundamentals-collections).

![image](https://cdn-images-1.medium.com/max/906/1*HVbOHlpmI5PGNgWUpgpI5A.jpeg)

#### 问题13：synchroize `getInstance()`方法的临界区和synchronize 整个`getInstance()`方法哪个好？([答案](http://javarevisited.blogspot.com/2014/05/double-checked-locking-on-singleton-in-java.html))

**答**： 答案是只synchronize 临界区。因为如果你锁了整个方法，每次调用这个方法，都必须等，即使你并不是在创建对象。

换句话说,[synchronization](http://javarevisited.blogspot.sg/2011/04/synchronization-in-java-synchronized.html#axzz4sZOoYUxv) 只需要在你创建对象的时候生效。一旦对象被创建，不需要任何同步。实际上，这种方法耗时很少。同步方法耗时是只同步临界区的10到20倍。

这是[单例模式](https://javarevisited.blogspot.com/2011/03/10-interview-questions-on-singleton.html)的UML图：

![image](https://cdn-images-1.medium.com/max/906/1*ySVB-r0Nj4m9P9i8Lz_dgw.png)

顺便提一句，有几种方法可以创建线程安全的单例，包括枚举，在这个问题里面我们也能提一下、

如果你想多学点，可以看这个免费课[学习Java创建型设计模式](http://bit.ly/2xZnIDC)


#### 问题14：在`HashMap`的`get()`操作中，`equals()`方法和`hashCode()`方法什么时候起作用？（[答案](https://javarevisited.blogspot.com/2017/08/top-10-java-concurrenthashmap-interview.html#axzz5ITbIGRsU)）

这个问题是前面问题的更进一步，候选人需要知道一但你提`hashCode`,很有可能被问`HashMap`里面的应用。

一但你提供一个key对象，hashcode方法会被调用用来计算桶位置。一个桶包含一个链表，每一个`Map.Entry`对象使用`equals()`方法来看是否已经存在相同key的value。

强烈推荐你阅读我的博客[Java中HashMap如何工作](http://javarevisited.blogspot.sg/2011/02/how-hashmap-works-in-java.html), 可以帮助你学习这个主题。

![image](https://cdn-images-1.medium.com/max/906/1*P5LcQWTCtU8VMPkpaA77eQ.jpeg)


#### 问题15： Java中如何避免死锁([答案](http://javarevisited.blogspot.sg/2015/10/133-java-interview-questions-answers-from-last-5-years.html))

**答**: 死锁发生是因为两个线程试图获取被对方持有的资源。但是要想发生这种情况，必须满足以下四个条件：

1. 相互排斥——至少一个进程必须处于非共享模式
2. 保持并等待——必须有一个进程持有一个资源并等待另一个资源
3. 没有抢占—— 资源不能被抢占
4. 循环等待 —— 存在进程集合

通过中断循环等待可以避免死锁。可以通过在代码中指定获取和释放锁的顺序来达到这一目的。

如果多个锁通过一致的顺序被获取和释放，不会有互相等待对方释放锁的情况。

你可以看我的博客[如何避免死锁](https://javarevisited.blogspot.com/2018/08/how-to-avoid-deadlock-in-java-threads.html), 查看示例代码和更加详细的解释。 同时推荐[在通用Java模式上使用并发和多线程](https://pluralsight.pxf.io/c/1193463/424552/7490?u=https%3A%2F%2Fwww.pluralsight.com%2Fcourses%2Fjava-patterns-concurrency-multi-threading) 这个课程来更好的理解多线程模式。

#### 问题16：双引号直接创建字符串和使用new()创建字符串有什么区别？

**答**: 使用new()创建String对象，实例被创建在[堆](https://javarevisited.blogspot.com/2011/05/java-heap-space-memory-size-jvm.html#axzz5SDsAfcC8)中, 不会被添加到String池中，当通过[字面量](http://www.java67.com/2014/08/difference-between-string-literal-and-new-String-object-Java.html) 创建时，会被放到堆中的永久区的String池中。

`String str = new String("Test")` 不会把str放到String池中，我们需要调用`String.intern()`方法，会把它放到String池中。

当我们使用String字面量创建String对象时，如通过String s = "Test", java会自动放入String池中。

另外，如果我们把"Test"这样的String字面量传进去，也会创建另外一个对象:"Test" 在[String池](http://javarevisited.blogspot.sg/2016/07/difference-in-string-pool-between-java6-java7.html)

这是我的知识盲区直到读者在我的[博客](http://javarevisited.blogspot.com/) 中给我提建议，如果想学习更多关于String字面量和String对象的知识，看[这里](http://java67.blogspot.sg/2014/08/difference-between-string-literal-and-new-String-object-Java.html)

![image](https://cdn-images-1.medium.com/max/906/1*4s5ZMfvuiIxI-4rBM_oubg.png)

#### 问题17：什么是不可变对象？你可以写一个不可变类吗？([答案](http://javarevisited.blogspot.in/2013/03/how-to-create-immutable-class-object-java-example-tutorial.html))

不可变对象是指Java类的对象一单被创建，不能被修改。任何不可变对象对象的修改在创建时候就已经完成，例如，[Java中String是不可变的](http://javarevisited.blogspot.sg/2010/10/why-string-is-immutable-in-java.html)

大多数不可变类是[final](https://javarevisited.blogspot.com/2011/12/final-variable-method-class-java.html)的, 这样可以防止因子类重写方法而导致不可变失效。

你也可以实现相同的功能通过让成员非final但是[private](http://javarevisited.blogspot.sg/2012/10/difference-between-private-protected-public-package-access-java.html)，且除了构造方法任何其他方法无法修改。

另外，要确保没有暴露不可变对象的内部，尤其是它包含可变成员的时候。

同时，当你从客户端接收到可变的对象时，例如`java.util.Date`, 使用[clone() 方法](http://javarevisited.blogspot.sg/2013/09/how-clone-method-works-in-java.html) 来获取一个独立的拷贝，防止恶意修改可变对象带来的风险。

相同的优化需要在返回一个可变成员时执行。返回另一个独立拷贝给客户端；不要返回可变对象的原始引用。你也可以看我的这篇博客[Java中如何创建一个不可变对象](http://javarevisited.blogspot.sg/2013/03/how-to-create-immutable-class-object-java-example-tutorial.html), 这里有按步骤的引导和示例代码。


#### 问题18：不用性能分析工具，给出一个简单的办法找到一个方法运行耗时

**答**：请求前和请求后记录时间，计算时间差值。如果一个方法耗时太小可能显示0毫秒，那么可以让方法变的足够大，比如重复执行足够多次。算总时间。

#### 问题19：当你使用Object作为HashMap里面的key的时候，哪两个方法需要实现？

为了在`hashMap`或者`hashtable`中把对象作为key，它必须实现[equals](http://www.java67.com/2012/11/difference-between-operator-and-equals-method-in.html)和[hashcode](http://javarevisited.blogspot.sg/2015/01/why-override-equals-hashcode-or-tostring-java.html#axzz55oDxm8vv)方法。

你也可以阅读[Java在HashMap中是如何工作的](http://javarevisited.blogspot.sg/2011/02/how-hashmap-works-in-java.html)，了解相关实现细节。


#### 问题20：如何防止客户端直接实例化你的具体类？例如你有一个`Cache`接口和两个实现类：`MemoryCache`和`DiskCache`。如何确保没有任何这两个类的对象通过new()关键字被创建出来？

**答**：在我给答案之前，自己研究一下这个问题。我相信你可以找到正确答案，从代码维护的角度来，控制你的类是非常重要的。

### 现在你已经准备好Java面试了

这是一些最通用的关于[数据结构](http://www.java67.com/2018/06/data-structure-and-algorithm-interview-questions-programmers.html)和[算法](http://www.java67.com/2018/06/data-structure-and-algorithm-interview-questions-programmers.html)的问题，他们可以帮忙你准备投行的技术面试。祝你好运！


#### 拓展阅读

1. [The Complete Java Masterclass](https://click.linksynergy.com/fs-bin/click?id=JVFxdTr9V80&subid=0&offerid=323058.1&type=10&tmpid=14538&RD_PARM1=https%3A%2F%2Fwww.udemy.com%2Fjava-the-complete-java-developer-course%2F)
2. [Java Fundamentals: The Java Language](https://pluralsight.pxf.io/c/1193463/424552/7490?u=https%3A%2F%2Fwww.pluralsight.com%2Fcourses%2Fjava-fundamentals-language)
3. [Core Java SE 9 for the Impatient](https://www.amazon.com/Core-Java-SE-Impatient-2nd/dp/0134694724?tag=javamysqlanta-20)
4. [200+ Java Interview questions](http://bit.ly/2CupaSL)

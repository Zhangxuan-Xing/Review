  <!-- GFM-TOC -->
* [一、基础介绍](#一基础介绍)  
* [二、线程安全性](#二线程安全性)  
* [三、对象的共享](#三对象的共享)  
* [四、总结笔记](#四总结笔记)
* <!-- GFM-TOC -->
# 一、基础介绍  

- 线程是Java语言中不可或缺的重要功能，能使复杂的异步代码变得更简单，从而极大地简化了复杂系统的开发

- 加入操作系统来实现多个程序的同时执行，主要基于以下原因：
 > 资源利用率：某程序等待时，可以运行另一个程序，提高资源利用率  
 公平性：不同程序对于资源有同等的使用权  
 便利性：每个程序执行一个任务比编写一个程序来完成所有任务更容易实现  

- 串行编程模型的优势在于直观性和简单性，模仿了人类的工作方式：每次只做一件事，做完一件再做下一件

- 线程允许在同一个进程中同时存在多个程序控制流，会共享进程范围内的资源

- 线程也称为轻量级进程，同一个进程的所有线程都将共享进程的内存地址空间，这些线程都能访问相同的变量并在同一个堆上分配对象  

- 线程将大部分的异步工作流转换成串行工作流，每个工作流在一个单独的线程中运行，并在特定的同步位置进行交互

- 线程可以提高用户界面的响应灵敏度，提升资源利用率和系统吞吐量，还可以简化JVM的实现（垃圾收集器通常在一个或多个专门的线程中进行）

- 单线程服务程序必须使用非阻塞I/O，每个请求都拥有自己的处理线程，那么在处理某个请求时发生的阻塞将不会影响其他请求的处理

- 在没有充足同步的情况下，多个线程中的操作执行顺序是不可预测，甚至会产生奇怪的结果

- 竞态条件：设备或系统出现不恰当的执行时序，而得到不正确的结果

- 安全性：永远不发生糟糕的事情

- 活跃性：某件正确的事情最终会发生，当某个操作无法继续执行下去时，就会发生活跃性问题

- 在多线程程序中，频繁的上下文切换操作会带来极大的开销引发性能问题

- 每个Java应用程序都会使用线程，如JVM启动时，将为JVM的内部任务（例如 垃圾收集、终结操作等）创建后台线程，并创建一个主线程来运行main方法

- 在代码中将不可避免地访问应用程序状态，因此所有访问这些状态的代码路径都必须是线程安全的，如：
 - Timer
 - Servlet和JavaServer Page
 - 远程方法调用
 - Swing和AWT 

---

# 二、线程安全性  
- 在构建稳健的并发程序时，必须正确地使用线程和锁

- 编写线程安全的代码，核心在于要对**状态**访问操作进行管理，特别是对**共享**和**可变**的状态的访问

- 对象的状态是指存储在状态变量（例如实例或静态域）中的数据，可能包括其他依赖对象的域

- “共享”意味着可以由多个线程同时访问，“可变”意味着变量的值在其生命周期内可以变化

- 一个线程是否需要是线程安全的，取决于它是否被多个线程访问

- 当多个线程访问某个状态变量并且有一个线程执行写入操作时，必须采用同步机制来协同这些线程对变量的访问（Java中的关键字是synchronized，“同步”术语还包括volatile类型、显示锁、原子变量）
  ​      

![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2001.png)   


![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2002.png)   

- 程序状态的封装性越好，就越容易实现程序的线程安全性，也越容易维护

- 完全由线程安全类构成的程序不一定就是线程安全的，在线程安全类中也可以包含非线程安全的类

- 线程安全性的核心概念是正确性，也就是说，某个类的行为和其规范完全一样

- 正确性：所见即所知，当多个线程访问某个类时，这个类始终都能表现出正确的行为，那么就称这个类是线程安全的
  ​      

![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2003.png)   

- 在线程安全类的对象实例上执行的任何串行或并行操作都不会使对象处于无效状态
  ​      

![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2004.png)   


![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2005.png)   

- 大多数Servlet都是无状态的，极大地降低了在实现Servlet线程安全性时的复杂性

- 竞态条件的本质：基于一种可能失效的观察结果来做出判断或者执行某个计算

- 与大多数并发错误一样，竞态条件并不总是会产生错误，还需要某种不恰当的执行时序
  ​      

![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2006.png)   

- 复合操作：包含了一组必须以原子方式执行的操作以确保线程安全性
  ​      

![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2007.png)   


![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2008.png)   

- Java提供了一种内置的锁机制来支持原子性：同步代码块。同步代码块包含两部分：一个作为锁的对象引用，一个作为由这个锁保护的代码块

- 每个Java对象都可以用作一个实现同步的锁，这些锁称为内置锁或监视器锁。线程在进入同步代码块之前会自动获得锁，退出后自动释放锁

- Java内置锁相当于一种互斥体，最多只有一个线程能持有这种锁

- 内置锁：代码块以原子方式执行，多个线程在执行时互不干扰

- 重入：锁的操作粒度是线程，而不是调用，进一步提升了加锁行为的封装性

- 重入锁实现重入性：每个锁关联一个线程持有者和计数器，当计数器为0时表示该锁没有被任何线程持有，那么任何线程都可能获得该锁而调用相应的方法；当某一线程请求成功后，JVM会记下锁的持有线程，并且将计数器置为1；此时其它线程请求该锁，则必须等待；而该持有锁的线程如果再次请求这个锁，就可以再次拿到这个锁，同时计数器会递增；当线程退出同步代码块时，计数器会递减，如果计数器为0，则释放该锁

- 同一线程在调用自己类中其他synchronized方法/块或调用父类的synchronized方法/块都不会阻碍该线程的执行，就是说同一线程对同一个对象锁是可重入的

- 锁能使其保护的代码路径以串行的形式来访问
  ​      

![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2009.png)   

- 对象的内置锁与其状态之间没有必然的联系
  ​      

![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2010.png)   

- 一种常见的加锁约定是，将所有的可变状态都封装在对象内部，并通过对象的内置锁对所有访问可变状态的代码路径进行同步，使得在该对象上不会发生并发访问，例如Vector和其他的同步集合类都采用了这种模式

- 并非所有数据都需要锁的保护，只有被多个线程同时访问的可变数据才需要通过锁来保护
  ​      

![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2011.png)   

- 只是将每个方法作为同步方法，不足以确保复合操作是原子的，还容易产生活跃性或性能问题
  ​      

![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2012.png)   


![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2013.png)   
​     
---

# 三、对象的共享  
- 编写并发程序的关键问题在于在访问共享的可变状态时，需要进行正确的管理
- **3.1 可见性**  
  
> 可见性是一种复杂的属性
> 
> 为了确保多个线程之间对内存写入操作的可见性，必须使用同步机制
> 
> 在某个线程中无法检测到重排序情况，那么就无法确保线程中的操作是按照程序制定的顺序来执行的
> 
> 只要有数据再多个线程之间共享，就使用正确的同步
> 
> **3.1.1 失效数据**
> 通过对get和set方法，可以使成为线程安全的类，仅对set同步是不够的

![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2014.png)   
​     
> 
> **3.1.2 非原子的64位操作**
> 当线程在没有同步的情况下读取变量时，可能会得到一个失效值，但至少这个值是由之前某个线程设置的值，而不是一个随机值 —— 这种安全性保证也称为 最低安全性
> 
> 最低安全性适用于绝大多数变量，存在一个例外：非volatile类型的64位数值变量（double 和 long）
> 
> 在多线程程序中使用共享且可变的long和double等类型的变量是不安全的，除非用volatile来声明或者锁来保护
> 
> **3.1.3 加锁与可见性**
> 内置锁可以用于确保某个线程以一种可预测的方式来查看另一个线程的执行结果

![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2015.png)   
​     
​        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2016.png)   
​     
> 
> **3.1.4 Volatile变量**
> 用来确保将变量的更新操作通知到其他线程
> 
> volatile变量不会被缓存在寄存器或者对其他处理器不可见的地方
> 
> volatile变量是一种比sychronized关键字更轻量级的同步机制
> 
> 从内存可见性的角度来看，写入volatile变量相当于退出同步代码块，而读取volatile变量相当于进入同步代码块 

![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2017.png)   
​     
> volatile变量的典型用法：检查某个状态标记以判断是否退出循环

![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2018.png)   
​     
> volatile语义不足以确保递增操作（count++）的原子性
> 
> 加锁机制既可以确保可见性又可以确保原子性，而volatile变量只能确保可见性

![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2019.png)   
​     

- **3.2 发布与逸出**  
  
> 发布：使对象能够在当前作用域之外的代码中使用，例如将一个指向该对象的引用保存到其他代码可以访问的地方，或者在某一个非私有的方法中返回该引用，或者将引用传递到其他类的方法中
> 
> 当某个不应该发布的对象被发布时，这种情况称为逸出
> 
> 当发布一个对象时，在该对象的非私有域中引用的所有对象同样会被发布
> 
> 无论其他线程对已发布的引用是否执行什么操作，误用该引用的风险始终存在
> 
> 不要在构造过程中使this引用逸出，常见错误是在构造函数中启动一个线程。在构造函数中调用一个可改写的实例方法（既不是私有方法，也不是终结方法），同样会导致this引用在构造过程中逸出
> 
> 在构造函数中注册一个事件监听器或启动线程，可以使用一个私有的构造函数和一个公共的工厂方法，从而避免不正确的构造过程

- **3.3 线程封闭**  
> 当访问共享的可变数据时，通常需要同步
> 
> 一种避免使用同步的方式就是不共享数据，如果仅在单线程内访问数据，就不需要同步，该技术称为 线程封闭
> 
> 应用场景：Swing的可视化组件 和 JDBC的Connection对象
> 
> **3.3.1 Ad-hoc线程封闭**
> 维护线程封闭性的职责完全由程序实现来承担，一般要将某个特定的子系统实现为一个单线程子系统
> 
> **3.3.2 栈封闭**
> 在栈封闭中，只能通过局部变量才能访问对象。局部变量的固有属性之一就是封闭在执行线程中，位于执行线程的栈中，其他线程无法访问栈
> 
> 对于基本类型的局部变量，由于任何方法都无法获得对基本类型的引用，不会破坏栈封闭性
> 
> 如果在线程内部上下文中使用非线程安全的对象，那么该对象仍然是线程安全的
> 
> **3.3.3 ThreadLocal类**
> ThreadLocal 使线程中的某个值与保存值的对象关联起来，提供了get和set等访问接口或方法，为每个使用该变量的线程都存有一个副本，总可以获得最新值
> 
> ThreadLocal对象通常用于防止对可变的单实例变量或全局变量进行共享（以JDBC的Connection对象为代表）
> 
> 当某个频繁执行的操作需要一个临时对象，例如一个缓存区，而同时又希望避免在每次执行时都重新分配该临时变量，可以使用ThreaLocal
> 
> 特定于线程的值保存在Thread对象中，当线程终止后，这些值会作为垃圾回收
> 
> ThreadLocal变量类似于全局变量，能降低代码的可重用性，并在类之间引入隐含的耦合性


- **3.4 不变性**
> 使用不可变对象也可以满足同步需求
> 
> 如果某个对象在被创建之后，其状态不能被修改，那么这个对象就称为不可变对象，线程安全性时不可变对象的固有属性之一
> 
> 不可变对象一定是线程安全的，只有一种状态
> 
> 可以安全地共享和发布不可变对象，无需创建保护性的副本
> 
> 不可变性并不等于将对象中所有的域都声明为final类型，即使对象所有域都是final类型的，对象也可能是可变的，因为在final类型的域中可以保存对可变对象的引用

![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2020.png)   
​     
​        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2021.png)   
​     
> *在初始化时，构造代码块优先于构造函数*
> 
> 保存在不可变对象的程序状态仍可以更新，可以通过一个保存新状态的实例来“替换”
> 
> **3.4.1 Final域**
> final类型的域是不能修改的，但如果final域所引用的对象是可变的，那么这些被引用的对象是可以修改的

![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2022.png)   
​     
> 对于在访问和更新多个相关变量时出现的竞争条件问题时，可以通过将这些变量全部保存在一个不可变对象中来消除，如果是一个可变的对象，就必须使用锁来确保原子性
> 
> **3.5 安全发布**
> 如果多个线程共享对象，就必须确保安全地进行共享
> 
> 不正确的发布，会让正确的对象被破坏
> 
> 某个对象的引用对其他线程是可见的，不意味着对象状态对于使用对象的线程来说一定是可见的
> 
> 为了确保对象状态一致，必须使用同步

![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2023.png)   
​     
​        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2024.png)   
​     
​        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2025.png)   
​     
> 对于可变对象，不仅在发布对象时需要使用同步，在每次对象访问时同样需要使用同步来确保后续修改操作的可见性

![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/2026.png)   
​     

# 四、总结笔记

## 1）基础

1. 银行案例：只对写加锁，不对读加锁，可能会有脏读问题，即  读到正在写的数据
2. 一个同步方法可以调用另一个同步方法，一个线程已经拥有某个对象的锁，再次申请时仍可以得到该对象的锁（也就是说synchronized获得的锁是可以重入的，获得过一次后还可以再获得，还有一种情况是子类中使用父类）
3. 在程序执行过程中，如果出现异常，默认锁会释放，所以在并发处理过程中要格外注意异常，不然会出现不一致的情况；如果不想释放，需要catch进行处理
4. volatile（保证可见性）不能保障多个线程共同修改变量带来的不一致问题，不能完全代替synchronized（保证原子性和可见性）
5. AtomXXX类是原子性的，但不能保证多个方法连续调用是原子性的  
6. 锁的粒度和效率有直接关系
7. 锁定对象o，如果o的属性发生改变，不影响锁的使用；如果o变成另一个对象，则锁定的对象会发生改变
8. synchronized锁的是堆里面的东西，不是栈里面，也不是锁代码块
9. 不要以字符串常量作为锁定对象
10. wait会释放锁（sleep不释放锁），notify不会释放锁，前提要先锁定对象；此外，需要先启动等待线程
11. 使用Latch门闩替代wait  notify来进行通知的好处是简单，也可以指定等待时间（使用await 和 countdown方法来替换wait和notify）。CountDownLatch不需要锁定对象，当count值为0时继续执行线程
12. 当不涉及同步时，只是涉及通信时，使用synchronized + wait/notify太重了
13. 同步方法和非同步方法可以同时调用
14. ReentrantLock用于替代synchronized，需要注意的是必须要手动释放锁（经常经常在finally中进行锁的释放），可以设定tryLock时间；可以调用lockInterruptibly方法，对线程interrupt方法做出响应，在一个线程等待的过程中，可以打断；可以指定为公平锁（默认的synchronized都是不公平锁）
15. wait在大多数情况下和while结合用，而不是if，因为要保证唤醒后的同步、不给其他线程机会；大多数情况下，用的是notifyall而不是notify
16. 与wait和notifyall相比，Lock和Condition能比较精确地确定线程
17. ThreadLocal ：线程局部变量，自己维护自己。ThreadLocal用空间换时间，synchronized用时间换空间，比如hibernate的session就存在于ThreadLocal中
18. [单例模式与多线程的案例](https://www.cnblogs.com/zhaoyan001/p/6365064.html)
19. ArrayList是不同步的，Vector是同步的，但要注意判断和操作是否都是原子性的

## 2）并发包

20. ConcurrentHashMap:只锁定16段的一段，而不是全部，所以理论上比HashTable、Collections.synchronizedXXX（将不加锁的  返回为加锁的，拿到的是带锁的）效率高，在并发性高的时候推荐使用

21. ConcurrentSkipListhMap([跳表说明](https://blog.csdn.net/sunxianghuang/article/details/52221913))：高并发并且排序，插入会慢一点，但是查询效果好

22. CopyOnWrite：写时复制容器，因为不存在脏读的情况，所以读不加锁；多线程环境下，写时效率低，读时效率高，适合写少读多的环境

23. ConcurrentLinkedQueue ：offer相当于add，返回是一个Boolean值，可以知道有没有加入成功，是一个无界队列

24. BlockingQueue：

    （1）LinkedBQ：无界、阻塞

    （2）ArrayBQ：有界，其中put()满了会阻塞，add()会抛异常，offer()会返回是否加入成功

    （3）LinkedTransferQueue：有一个transfer()方法，如果有消费者，生产者直接交给消费者；如果找不到消费者，就会阻塞 — 即必须被消费掉

    （4）SynchronusQueue：容量为0（一般需要先启动消费者），只能用put（）而不能用add（），阻塞等待消费

25. DelayQueue：put的元素必须实现Delay接口，可以用于定时执行任务

## 3）线程池

### 3.1 Executor

1. 是一个接口 interface
2. 有一个execute方法，可以调用Runnable的run方法，用来执行某个任务

### 3.2  ExecutorService

1. 继承自Executor
2. 可以调用Runnable（execute）或者Callable（submit）

### 3.3  Callable

1. 和Runnable很像，它有一个call（）方法，有返回值（Runnable无返回值）
2. 可以抛异常（Runnable不抛异常）

### 3.4  Executors

1. 操作Executor的工具类
2. 操作工厂方法和工具方法

### 3.5  ThreadPool

1. 线程池有一个等待的维护队列，一般是BQ

2. 创建线程的方式掌握 - **大部分都是通过ExecutorThreadPool支撑**：

   ① newFixedThreadPool：可以定个数

   ![](D:\githubXuan\Review\Picture\08.png)

   ② newCachedThreadPool：如果线程里有空闲线程，任务之间丢给空闲线程；如果没有，就新建一个线程。超过指定时间(默认60s)的空闲线程会收回，可以自定

   ![](D:\githubXuan\Review\Picture\09.png)

   ③ newSingleThreadPool：一个线程，保证了线程的顺序性

   ![](D:\githubXuan\Review\Picture\010.png)

   ④ newScheduledThreadPool：定时任务，线程可以复用

   ![](D:\githubXuan\Review\Picture\011.png)

   ⑤ newWorkStealingPool：工作窃取，当多线程中的某个执行完自己任务时，会主动的去别的任务队列中拿任务执行。是精灵线程（守护线程），主线程不阻塞的话，看不到输出，是对ForkJoinPool的封装升级

   ⑥ forkjoin

### 3.6 Future

![](D:\githubXuan\Review\Picture\07.png)

### 3.7  ForkJoin

1. 先切后合，递归任务

2. 有无返回值

   ```java
   public abstract class RecursiveTask<V> extends ForkJoinTask<V>;
   public abstract class RecursiveAction extends ForkJoinTask<Void>;
   ```

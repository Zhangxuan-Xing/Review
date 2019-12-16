  <!-- GFM-TOC -->
* [一、前言](#一前言)  
* [二、基础](#二基础)
* [三、内存模型和线程安全](#三内存模型和线程安全)  
* [四、无锁](#四无锁)
* [五、JDK并发包](#五JDK并发包)
* [六、锁优化](#六锁优化)
* [七、JDK8新支持](#七JDK8新支持)
* <!-- GFM-TOC -->
# 一、前言 

1. 并行的必要性

   ①摩尔定律失效；

   ②业务模型的需要，如HTTP服务器，为每一个Socket连接新建一个处理线程，让不同的线程承担不同的任务

2. 同步与异步

   ![](D:\githubXuan\Review\Picture\01.png)

3. 临界区

   ![](D:\githubXuan\Review\Picture\02.png)

4. 阻塞（Blocking）和非阻塞（Non-Blocking）
   阻塞和非阻塞通常用来形容多线程间的相互影响

   比如一个线程占用了临界区资源，那么其它所有需要这个资源的线程就必须在这个临界区中进行等待，等待会导致线程挂起，这种情况就是阻塞。此时，如果占用资源的线程一直不愿意释放资源，那么其它所有阻塞在这个临界区上的线程都不能工作
   非阻塞允许多个线程同时进入临界区

5. 乐观锁和悲观锁

   **5.1 乐观锁**

   乐观锁是一种乐观思想，即认为读多写少，遇到并发写的可能性低，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，采取在写时先读出当前版本号，然后加锁操作（比较跟上一次的版本号，如果一样则更新），如果失败则要重复读-比较-写的操作

   Java中的乐观锁基本都是通过CAS操作实现的，CAS是一种更新的原子操作，比较当前值跟传入值是否一样，一样则更新，否则失败



   **5.2 悲观锁**

   悲观锁是就是悲观思想，即认为写多，遇到并发写的可能性高，每次去拿数据的时候都认为别人会修改，所以每次在读写数据的时候都会上锁，这样别人想读写这个数据就会block直到拿到锁

   Java中的悲观锁就是Synchronized, AQS框架下的锁则是先尝试CAS乐观锁去获取锁，获取不到，才会转换为悲观锁，如RetreenLock

6. 死锁、饥饿和活锁

   | 类型 |                             说明                             |
   | :--: | :----------------------------------------------------------: |
   | 死锁 |               A需要B，B需要C，C需要A…形成闭圈                |
   | 饥饿 | 某一个或者多个线程因为种种原因无法获得所需要的资源，导致一直无法执行 |
   | 活锁 |                      类似于电梯遇人情景                      |

7. 并发级别

   |  类别  |  级别  |                             说明                             |
   | :----: | :----: | :----------------------------------------------------------: |
   |  阻塞  |  阻塞  |           当一个线程进入临界区后，其他线程必须等待           |
   | 非阻塞 | 无障碍 | 无障碍是一种最弱的非阻塞调度，自由出入临界区。无竞争时，有限步内完成操作；有竞争时，回滚数据（乐观）。不能保证不会死锁 |
   | 非阻塞 |  无锁  |                无障碍，保证有一个线程可以胜出                |
   | 非阻塞 | 无等待 |         要求所有的线程都必须在有限步内完成，且无饥饿         |

8. 相关定律

   |            定律             |                             内容                             |
   | :-------------------------: | :----------------------------------------------------------: |
   | Amdahl定律（阿姆达尔定律）  | 定义了串行系统并行化后的加速比的计算公式和理论上限；加速比定义：加速比=优化前系统耗时/优化后系统耗时 |
   | Gustafson定律（古斯塔夫森） |          说明处理器个数，串行比例和加速比之间的关系          |

---

# 二、基础  

## 1）常规知识

1. 新建线程的四种方式

   1）继承Thread类创建线程

   2）实现Runnable接口创建线程

   3）使用Callable和Future创建线程

   4）使用线程池例如用Executor框架

2. 中断线程

   1）Thread.interrupt()    中断线程

   2）Thread.isInterrupted()    判断是否被中断

   3）Thread.interrupted()    判断是否被中断，并清除当前中断状态

3. 挂起（suspend）和继续执行（resume）线程
   1）suspend() 不会释放锁
   2）如果加锁发生在resume()之前 ，则死锁发生

   ![](D:\githubXuan\Review\Picture\04.png)

4. 等待线程结束（join）和谦让(yeild)：不要在Thread实例上使用 wait()和notify()方法 

## 2）守护线程

1. 在后台默默地完成一些系统性的服务，比如垃圾回收线程、JIT线程就可以理解为守护线程

2. 当一个Java应用内，只有守护线程时，Java虚拟机就会自然退出

3. 示例

   ```java
   Thread t=new DaemonT();
   t.setDaemon(true);
   t.start();
   ```

## 3）线程优先级

高优先级的线程更容易再竞争中获胜

```java
public final static int MIN_PRIORITY = 1;
public final static int NORM_PRIORITY = 5;
public final static int MAX_PRIORITY = 10;
Thread high=new HightPriority();
LowPriority low=new LowPriority();
high.setPriority(Thread.MAX_PRIORITY);
low.setPriority(Thread.MIN_PRIORITY);
low.start();
high.start();
```

## 4）同步操作

1. synchronized指定加锁对象：

   ① 对给定对象加锁，进入同步代码前要获得给定对象的锁

   ```java
   public void run() {
   	for(int j=0;j<10000000;j++){
   			synchronized(instance){
   				i++;
   			}
   	}
   }
   ```

   ② 直接作用于实例方法：相当于对当前实例加锁，进入同步代码前要获得当前实例的锁

   ```java
   public synchronized void increase(){
   	i++;
   }
   ```

   ③ 直接作用于静态方法：相当于对当前类加锁，进入同步代码前要获得当前类的锁

   ```java
   public static synchronized void increase(){
   	i++;
   }
   ```

2. Object.wait() Obejct.notify()

   ![](D:\githubXuan\Review\Picture\03.png)

---

# 三、内存模型和线程安全 
## 1）原子性

原子性是指一个操作是不可中断的。即使是在多个线程一起执行的时候，一个操作一旦开始，就
不会被其它线程干扰【如i++就不是原子性的】

## 2）有序性

一条指令的执行是可以分为很多步骤的：

1. 取指 IF译码和取寄存器操作数 ID
2. 执行或者有效地址计算 EX
3. 存储器访问 MEM
4. 写回 WB

## 3）可见性

可见性是指当一个线程修改了某一个共享变量的值，其他线程是否能够立即知道这个修改

## 4）线程安全

指某个函数、函数库在多线程环境中被调用时，能够正确地处理各个线程的局部变量，使程序功
能正确完成

---

# 四、无锁

## 1）无锁类的CAS原理

CAS算法的过程是这样：它包含3个参数CAS(V,E,N)。V表示要更新的变量，E表示预期值，N表示新值。仅当V
值等于E值时，才会将V的值设为N，如果V值和E值不同，则说明已经有其他线程做了更新，则当前线程什么
都不做。最后，CAS返回当前V的真实值。CAS操作是抱着乐观的态度进行的，它总是认为自己可以成功完成
操作。

当多个线程同时使用CAS操作一个变量时，只有一个会胜出，并成功更新，其余均会失败。

失败的线程不会被挂起，仅是被告知失败，并且允许再次尝试，当然也允许失败的线程放弃操作。

基于这样的原理，CAS操作即时没有锁，也可以发现其他线程对当前线程的干扰，并进行恰当的处理。

## 2）无锁类的使用

### 2.1  AtomicInteger

### 2.2 Unsafe

1. 可以用来在任意内存地址位置处读写数据，支持一些CAS原子操作。Unsafe被JDK广泛应用于java.nio和并发包等实现中，这个不安全的类提供了一个观察HotSpot JVM内部结构并且可以对其进行修改，但是不建议在生产环境中使用。

2. Unsafe对象不能直接通过new Unsafe()或调用Unsafe.getUnsafe()获取，原因如下：（1）不能直接new Unsafe()，原因是Unsafe被设计成单例模式，构造方法是私有的；（2）不能通过调用Unsafe.getUnsafe()获取，因为 getUnsafe 被设计成只能从引导类加载器(bootstrap class loader)加载

3. 是非安全的操作，比如：根据偏移量设置值、底层的CAS操作

4. park()：当遇到线程终止时，会直接返回(不同于Thread.sleep，Thread.sleep遇到thread.interrupt()会抛异常)

   unpark()：无法恢复处于sleep中的线程，只能与park配对使用，因为unpark发放的许可只有park能监听到

   **unpark函数可以先于park调用，这个正是它们的灵活之处**

### 2.3  AtomicReference

对引用进行修改，是一个模板类，抽象化了数据类型。AtomicReference和AtomicInteger非常类似，不同之处就在于AtomicInteger是对整数的封装，而AtomicReference则对应普通的对象引用。也就是它可以保证你在修改对象引用时的线程安全性。

### 2.4  AtomicStampedReference

[解决ABA问题](https://www.cnblogs.com/java20130722/p/3206742.html)

### 2.5  AtomicIntegerArray

支持无锁的数组

### 2.6  AtomicIntegerFieldUpdater

让普通变量也享受原子操作，注意如下：

1. 字段必须是volatile类型的，在线程之间共享变量时保证立即可见
2. Updater只能修改它可见范围内的变量。因为Updater使用反射得到这个变量。如果变量不可见，就会出错。
   比如如果score申明为private，就是不可行的。对于父类的字段，子类是不能直接操作的，尽管子类可以访问父类的字段
3. 由于CAS操作会通过对象实例中的偏移量直接进行赋值，只能是实例变量，不能是类变量，也就是说不能加static关键字
4. 只能是可修改变量，不能使final变量，因为final的语义就是不可修改。实际上final的语义和volatile是有冲突的，这两个关键字不能同时存在
5. 对于AtomicIntegerFieldUpdater和AtomicLongFieldUpdater只能修改int/long类型的字段，不能修改其包装类型（Integer/Long）。如果要修改包装类型就需要使用AtomicReferenceFieldUpdate

----

# 五、JDK并发包  

## 1）同步控制工具

### 1.1  .ReentrantLock(互斥锁  独占锁)

1. 可重入：单线程可以重复进入，但要重复退

2. 可中断：lockInterruptibly()

3. 可限时：超时不能获得锁，就返回false，不会永久等待构成死锁

4. 公平锁先来先得，默认是非公平锁

   ```java
   public ReentrantLock(boolean fair)
   public static ReentrantLock fairLock = new ReentrantLock(true)；
   ```

### 1.2  Condition

类似于 Object.wait()和Object.notify()，与ReentrantLock结合使用

Condition的作用是对锁进行**更精确**的控制。Condition中的await()方法相当于Object的wait()方法，Condition中的signal()方法相当于Object的notify()方法，Condition中的signalAll()相当于Object的notifyAll()方法。不同的是，Object中的wait(),notify(),notifyAll()方法是和同步锁synchronized关键字捆绑使用的；而Condition是需要与["互斥锁"/"共享锁"](http://www.cnblogs.com/skywang12345/p/3496101.html)捆绑使用的

### 1.3   Semaphore

共享锁，运行多个线程同时临

### 1.4  ReadWriteLock

ReadWriteLock是JDK5中提供的读写分离锁。

读-读  ：不互斥、不阻塞；
读-写  写-写  ：互斥、阻塞。

### 1.5 .CountDownLatch

倒数计时器（可以设数）：一种典型的场景就是火箭发射。在火箭发射前，为了保证万无一失，往往还要进行各项设备、仪器的检查。只有等所有检查完毕后，引擎才能点火。这种场景就非常适合使用CountDownLatch。它可以使得点火线程，等待所有检查线程全部完工后，再执行

![](D:\githubXuan\Review\Picture\05.png)

### 1.6  .CyclicBarrier

**循环栅栏**：Cyclic意为循环，也就是说这个计数器可以反复使用。比如，假设我们将计数器设置为10。那么凑齐第一批1
0个线程后，计数器就会归零，然后接着凑齐下一批10个线程

![](D:\githubXuan\Review\Picture\06.png)

### 1.7   LockSupport

1. 提供线程阻塞原语

2. 主要接口
   LockSupport.park();
   LockSupport.unpark(t1);

3. 与suspend()比较

   不容易引起线程冻结

4. 中断响应
   能够响应中断，但不抛出异常。
   中断响应的结果是，park()函数的返回，可以从Thread.interrupted()得到中断标志

## 2）线程池

待更

---

# 六、锁优化

## 1） 思路和方法

### 6.1.1  减少锁持有时间

```java
public synchronized void syncMethod(){
	othercode1();
	mutextMethod();
	othercode2();
}
// 转为
public void syncMethod2(){
	othercode1();
	synchronized(this){
		mutextMethod();
	}
	othercode2();
}
```

### 6.1.2  减小锁粒度

2.1 将大对象，拆成小对象，大大增加并行度，降低锁竞争

2.2 偏向锁，轻量级锁成功率提高

2.3  ConcurrentHashMap：在减小锁粒度后， ConcurrentHashMap允许若干个线程同时进入

### 6.1.3  锁分离

1. 根据功能进行锁分离，如读写分离

   |      |   读锁   |   写锁   |
   | :--: | :------: | :------: |
   | 读锁 |  可访问  | 不可访问 |
   | 写锁 | 不可访问 | 不可访问 |

2. 思想可以延伸，只要操作互不影响，锁就可以分离，如 **LinkedBlockingQueue**

### 6.1.4  锁粗化

通常情况下，为了保证多线程间的有效并发，会要求每个线程持有锁的时间尽量短，即在使用完
公共资源后，应该立即释放锁。只有这样，等待在这个锁上的其他线程才能尽早的获得资源执行
任务。但是，凡事都有一个度，如果对同一个锁不停的进行请求、同步和释放，其本身也会消耗
系统宝贵的资源，反而不利于性能的优化

```java
public void demoMethod(){
	synchronized(lock){
		//do sth.
	}
	//做其他不需要的同步的工作，但能很快执行完毕
	synchronized(lock){
		//do sth.
	}
}
//转为
public void demoMethod(){
	//整合成一次锁请求
	synchronized(lock){
		//do sth.
	//做其他不需要的同步的工作，但能很快执行完毕
	}
}

//或如下
for(int i=0;i<CIRCLE;i++){
	synchronized(lock){
    }
}
//转为
synchronized(lock){
	for(int i=0;i<CIRCLE;i++){
	}
}
```



### 6.1.5  锁消除

在即时编译器时，如果发现不可能被共享的对象，则可以消除这些对象的锁操作

## 2）虚拟机内的锁优化

**前提知识点：对象头**

1. Mark Word，对象头的标记，32位
2. 描述对象的hash、锁信息，垃圾回收标记，年龄
   – 指向锁记录的指针
   – 指向monitor的指针
   – GC标记
   – 偏向锁线程ID

### 6.2.1  偏向锁

1. 大部分情况是没有竞争的，所以可以通过偏向来提高性能
2. 所谓的偏向，就是偏心，即锁会偏向于当前已经占有锁的线程
3. 将对象头Mark的标记设置为偏向，并将线程ID写入对象头Mark
4. 只要没有竞争，获得偏向锁的线程，在将来进入同步块，不需要做同步
5. 当其他线程请求相同的锁时，偏向模式结束
   XX:+UseBiasedLocking
6. 默认启用
   在竞争激烈的场合，偏向锁会增加系统负担

### 6.2.2  轻量级锁

1. 普通的锁处理性能不够理想，轻量级锁是一种快速的锁定方法。
2. 如果对象没有被锁定
   – 将对象头的Mark指针保存到锁对象中
   – 将对象头设置为指向锁的指针（在线程栈空间中）
3. 如果轻量级锁失败，表示存在竞争，升级为重量级锁（常规锁）
4. 在没有锁竞争的前提下，减少传统锁使用OS互斥量产生的性能损耗
5. 在竞争激烈时，轻量级锁会多做很多额外操作，导致性能下降

### 6.2.3  自旋锁

1. 当竞争存在时，如果线程可以很快获得锁，那么可以不在OS层挂起线程，让线程做几个空操作（
   自旋）
2. JDK1.6中-XX:+UseSpinning开启
3.  JDK1.7中，去掉此参数，改为内置实现
4. 如果同步块很长，自旋失败，会降低系统性能
5. 如果同步块很短，自旋成功，节省线程挂起切换时间，提升系统性能

### 6.2.4 小结

内置于JVM中的获取锁的优化方法和获取锁的步骤

1. 偏向锁可用会先尝试偏向锁
2. 轻量级锁可用会先尝试轻量级锁
3. 以上都失败，尝试自旋锁
4. 再失败，尝试普通锁，使用OS互斥量在操作系统层挂起

[其他补充](https://blog.csdn.net/zqz_zqz/article/details/70233767)

----

# 七、JDK8新支持

## **CompletableFuture**

### 1）完成后得到通知

```java
01 public static class AskThread implements Runnable {
02		 CompletableFuture<Integer> re = null;
03 
04 public AskThread(CompletableFuture<Integer> re) {
05 		this.re = re;
06 }
07 
08 @Override
09 public void run() {
10 		int myRe = 0;
11 		try {
12 			myRe = re.get() * re.get();
13		 } catch (Exception e) {
14 		}
15 		System.out.println(myRe);
16 		}
17 }
18 
19 public static void main(String[] args) throws InterruptedException {
20 		final CompletableFuture<Integer> future = new CompletableFuture<>();
21 		new Thread(new AskThread(future)).start();
22 			// 模拟长时间的计算过程
23 			Thread.sleep(1000);
24 			// 告知完成结果
25 			future.complete(60);
26 }
```

### 2）异步执行

### 3）工厂方法

```java
static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier);
static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor);
static CompletableFuture<Void> runAsync(Runnable runnable);
static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor);
```

### 4）流式调用

### 5）组合多个CompletableFuture

```java
public <U> CompletableFuture<U> thenCompose(Function<? super T, ? extends CompletionStage<U>> fn)
```

##  StampedLock

### 1）读写锁的改进，读不阻塞写

### 2）实现思想

不会进行无休止的自旋，会在在若干次自旋后挂起线程
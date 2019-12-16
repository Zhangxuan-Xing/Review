<!-- GFM-TOC -->
* [一、基础介绍](#一基础介绍)  
* [二、演进过程](#二演进过程) 
* [三、数据库](#三数据库)  
* [四、构建Java中间件](#四构建Java中间件)  
* [五、服务框架](#五服务框架) 
<!-- GFM-TOC -->
  
# 一、基础介绍  
  
- 定义有两个重点：
 > 组件分布在网络计算机上  
  组件之间仅仅通过消息传递来通信并协调行动

- 分布式的示意图如下图所示，从用户视角来看，用户面对的就是一个服务器，提供用户需要的服务
   
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1001.png)   
   
- 分布式系统一定是由多个节点组成的系统，一般来说，一个节点就是一台计算机；这些节点不是孤立的，而是互相连通的；最后，这些连通的节点上部署了组件，并且相互之间的操作会有协同

- 分布式系统出现的原因：
 > 升级单机处理能力的性价比越来越低  
  单机处理能力存在瓶颈  
  出于稳定性和可用性的考虑

- 组成计算机的五要素：输入设备、输出设备、运算器、控制器和存储器，存储器又分为内存和外存。计算机断电时，内存数据会丢失，外存无影响

- 对于存储数据的容器或者对象，有线程安全和不安全之分，对于线程不安全的容器或对象，一般可以通过加锁或者通过CopyOnWrite方式来控制并发访问。使用加锁方式时，如果数据在多线程中的读写比例很高，一般会采用读写锁而非简单的互斥锁

- ConcurrentHashMap是线程安全的，那是在他们的内部操作，其外部操作还是需要自己来保证其同步的，特别是静态的ConcurrentHashMap,其有更新和查询的过程，要保证其线程安全，需要syn一个不可变的参数才能保证其原子性

- 一般来说，能原子性地获取需要的多个锁，或者注意调整对多个锁的获取顺序，能比较好的避免死锁

- 线程是属于进程的，一个进程内的多个线程共享了进程的内存空间，而多个进程之间的内存空间是独立的

- 多进程区别于单进程的两个特点：
 > 资源控制更容易实现  
  多进程中的单个进程问题，不会造成整体的不可用
 ---
### 网络IO实现方式
- BIO 即Blocking IO，采用阻塞的方式实现，模式简单
   
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1002.png)   
   

- NIO 即 Nonblocking IO，基于事件驱动思想，NIO的一个明显好处是不需要为每个Socket套接字分配一个线程，可以在一个线程中处理多个Socket套接字相关的工作。采用的是Reactor模式，如下图所示  
   
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1003.png)   
      
Reactor会管理所有的handler，并且把出现的事件交给相应的Handler去处理。  
  
在NIO方式下，不是用单个线程去应对单个Socket套接字，而是统一通过Reactor对所有客户端的Socket套接字的事件做处理，然后派发到不同的线程中，如下图所示  
   
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1004.png)   
  
- AIO 即 AsynchronousIO，也就是异步IO，采用Proactor模式。AIO与NIO的最大差别是NIO在有通知时可以进行相关操作，例如读和写，AIO在有通知时表示相关操作已经完成  
   
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1005.png)   
  
- 分布式系统中的输入输出设备可以分为两类，一类是互相连接的多个节点，另一种是传统意义上的人机交互设备

- 控制器的主要作用就是协调和控制节点之间的动作和行为
 - **使用硬件负载均衡的请求调用**：请求发起方需要确定谁来处理  
    
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1006.png)   
  
 - **使用LVS的请求调用**：透明代理，发起请求和处理请求都会以为是中间代理方处理（不足：1. 增加网络的开销，一方面是流量，另一方面是延迟；2.透明代理处于请求必经路上，如果代理出问题，所有请求都会受影响）  
   
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1007.png)   
  
 - **采用名称服务的直连方式的请求调用**：请求发起方和请求处理方直接相连，名称服务起到了地址交换的作用，原来在透明代理上的工作被拆分到了名称服务和发起请求的机器上了，减少了中间路径以及可能的额外带宽消耗，但代码升级变得复杂   
   
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1008.png)   
  
 - **采用规则服务控制路由的请求直连调用**：对规则进行处理从而进行请求处理服务机器选择的代码逻辑，名称服务是通过和请求处理的机器交互来获得地址的，而规则服务器本身不需要和机器交互，只负责把规则提供给请求发起的机器    
   
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1009.png)   
  
 - **Master+Worker的方式**：存在一个Master节点管理任务，由其把任务分配给不同的Worker去进行处理，这个方式更多的是任务的分配和管理  
   
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1010.png)   
  
- 在单机系统中，运算器是具体的电子元件；在分布式系统中，运算器是由多个节点来组成的

- 构成运算器的多个节点在控制器的配合下，对外提供服务，构成了分布式系统中的运算器

- 使用规则服务器来分配任务可能存在的最大问题是任务分配不均衡，用Master节点的方式对任务的分配可以做得更好些
 ---
### 分布式系统中的存储器  
- **单机的Key-Value服务**：最基础  
   
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1011.png)   
  
- **代理服务**:一般可以根据请求的Key进行划分  
   
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1012.png)   
  
- **名称服务**：名称服务用于管理在线的KV存储服务器，并且把地址传到应用服务器这边，应用服务器会和KV存储器直接联系  
   
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1013.png)   
  
- **规则服务**:有两种实施经验，一种是通过规则服务器的配合，完成固定的Sharding策略，另一种是对等看待多台KV存储服务器  
   
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1014.png)   
  
- **Master控制**：相比名称服务，Master返回的是对应的KV存储服务器地址，而不是所有地址  
   
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1015.png)   
  ---       
- 在分布式系统中，整个系统的一部分有问题而其他部分正常，称为故障独立性

- 如果某个角色或者功能只有某台单机在支撑，那么这个节点称为单点，发生故障称为单点故障。需要尽量避免单点，尽量保证功能使集群完成的，如果不能变成集群实现，有另外两种选择：
 > 做好单点备份，便于恢复  
  降低单点故障的影响范围，这种方式更多的是转移和交换（数据库为典型）
 ---
  
# 二、架构演进   
  
- 大型网站是一种很常见的分布式系统，中间件系统也是在大型网站的架构变化中出现并发展的

- 对于大型网站，高并发的访问量和海量数据二者缺一不可。此外，本身业务和系统的复杂度也是考察的指标

- 架构演进图  
   
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1017.png)   
  
   
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1018.png)   
  
只是在应用的配置中把数据库的地从本机改到了另一台机器上，对开发、测试、部署没什么影响，环境当前的系统压力  
   
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1019.png)   
  
应用从单机变为集群的优化方式，存在两个问题：
 > 用户对于两个应用服务器的选择问题：1.DNS解决；2.增加负载均衡设备解决
 > Session问题
   
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1020.png)   
  
- HTTP协议本身是无状态的，需要基于HTTP协议支持会话状态的机制：这样的机制可以使Web服务器从多次单独的HTTP请求中看到“会话”，也就是知道哪些请求是来自哪个会话的
 > 实现方式：在会话开始时，分配一个唯一的会话标识SessionID，通过Cookie把这个标识告诉浏览器，以后每次请求的时候，浏览器都会带上这个会话标识  

- 不同的会话有独立的存储，保存不同会话的信息

- Seesion问题如下图所示  
     
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1021.png)   
  
 > Session Sticky：在负载均衡器上做“手脚”，让同样的Session 请求每次都发送到同一个服务端处理，利于针对Session进行服务端本地的缓存  
 > 
 >> 问题：
 >>>1.如果Web服务器宕机或者重启，数据会丢失  
 >>>2.会话需要进行应用层的解析，开销较大  
 >>>3.负载均衡器变成了一个有状态的节点，与无状态的节点相比，内存消耗更大，容灾方面更麻烦   
     
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1022.png)   
  
 >Seesion Replication：服务器之间增加会话数据的同步，如今一般的应用容器都支持该方式，通过复制解决问题，但不适合集群机器较多的场景
 >
 >> 问题：
 >>>1.同步Session数据造成带宽的开销
 >>>2.每台Web服务器都要保存Session数据，如果集群的Session数据很多，保存的数据占用会很严重  
     
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1023.png)   
  
  
 >Seesion 数据集中存储：把Session数据集中存储起来，不同Web服务器从同样的地方来获取Session，保证了不同Web服务器读取的Session数据都是一样的
 >
 >> 问题：
 >>>1.引入了网络操作，存在时延和不稳定性
 >>>2.如果集中存储Session的机器或集群有问题，会影响应用    
     
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1024.png)   
  
  
 >Cookie Based：通过Cookie来传递Session数据
 >
 >> 问题：
 >>>1.Cookie长度限制  
 >>>2.安全性：让服务端数据到了外部网络和客户端  
 >>>3.带宽消耗：数据中心的整体外部带宽的消耗  
 >>>4.性能影响：每次HTTP请求和响应都要带有Session数据      
     
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1025.png)   
  
- **Session Sticky 和 Session 集中存储  对于大型网站来说，是比较好的选择方案**
 ---
  
# 三、数据库  
  
### 读写分离  
 
- 采用数据库作为读库 

>  增加数据库作为读库，只提供读服务，如下图所示   
     
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1026.png)   
  
>  问题：数据库一般都提供了数据复制的功能，但对于数据复制，需要考虑数据复制时延问题，以及复制过程中数据的源和目标之间的映射关系以及过滤条件的支持问题  
  
>  对于应用来说，不同情况选择不同的数据库源，写操作要走主库，事务中的读也要走主库

- 搜索引擎其实是读库 
     
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1028.png)   
  
> 搜索集群的使用方式和读库的使用方式是一样的，只是构建索引的过程基本是我们自己来实现的，可以从两个维度进行划分：一种是按照全量/增量划分，一种是按照实时/非实时划分
> 
> 整体来说，搜索引擎的技术解决了站内搜索时某些场景下读的外套，提供了更好的查询效率，和使用读库非常类似
  
- 加速数据读取的利器 —— 缓存  
     
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1027.png)   
  
1. 数据缓存  
 > 大型系统中的数据缓存主要用于分担数据库的读的压力，缓存系统一般用来保存和查询键值对，在缓存中放的是“热”数据而不是全部数据  

2. 页面缓存  
 > 对于数据缓存读取的情况，一个很关键的指标是缓存命中率，缓存服务器扩容或者缩容要尽量平滑（一致性Hash会是不错的选择） 
  
### 弥补关系型数据库不足，引入分布式存储系统   
-  常见的分布式存储系统有分布式文件系统、分布式Key-Value系统和分布式数据库
-  分布式存储系统通过集群，提供了一个高容量、高并发量、数据冗余容灾的支持
     
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1029.png)   
  
### 数据库拆分  
- 专库专用，垂直拆分
> 把数据库中不同的业务数据拆分到不同的数据库中  
> 
> 应用需要配置多个数据源，增加了所需的配置，带来了每个数据库的隔离  
     
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1030.png)   
  
- 数据水平拆分
> 把同一个表的数据拆分到两个数据库中  
> 
> 主键处理会变得不同，例如Oracle的Sequence或者Mysql的主键不能简单地继续使用了  
> 
> 一些查询需要从两个数据库中读取，如果需要分页且数据量大，就比较难处理  
     
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1031.png)   
  
### 新挑战   
- 拆分应用：业务拆分和功能拆分  
     
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1032.png)   
  
     
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1033.png)   
  
- 服务化
      
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1034.png)   
  
> 业务功能之间的访问不再是单机内部的方法调用  
> 
> 共享的代码不再是散落在不同的应用中了，放在了各个服务中心  
> 
> 与数据库的交互工作放到了服务中心  
  
### 演进后的网站
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1035.png)   
   
# 四、构建Java中间件  
  
- 中间件为软件应用提供了操作系统所提供的服务之外的服务，能够让软件开发者方便地处理通信、输入和输出，能专注在他们自己应用的部分

- 中间件起到的是桥梁作用，是应用与应用之间的桥梁，也是应用与服务直接的桥梁
  
## 线程池  
- 线程池可以降低线程的开销，在线程执行结束后进行的是回收操作，而不是真正销毁线程

- Java中，主要使用的线程池是ThreadPoolExecutor

- ReentrantLock是java.util.concurrent.locks的一个类，用法类似于修饰代码段的synchronized，不过需要显式地进行unlock，unlock一般放在finally中，以保证一定会释放锁

- synchronized保证了synchronized块中变量的可见性，能控制并发；而volatile则保证了所修饰变量的可见性，不能控制并发

- 同一变量线程间的可见性与多个线程操作互斥是两件事，互斥是提供了操作整体的原子性

- wait、notify和notifyAll是Java的Object对象上的三个方法，调用都必须在对象的synchronized块中
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1036.png)   
   
- 在实践中，对wait的使用一般是嵌在一个循环中，并且会判断相关的数据状态是否到达预期，如果没有则会继续等待，这么做主要是为了防止虚假唤醒  

- **CountDownLatch**：是java.util.concurrent包中的一个类，主要提供的机制是当多个线程都到达了预期状态工作时触发事件，其他线程可以等待这个事件来触发自己后续的工作（等待的线程可以是多个）。CountDownLatch初始化的count为1，这就退化为单一事件了，一般大于1

- **CyclicBarrier** ：循环屏障，可以协同多个线程，让多个线程在这个屏障前等待，直到所有线程都到达了这个屏障时，再一起继续执行后续的动作
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1037.png)   
   
- Semaphore用于管理信号量

- **Exchanger**：用于两个线程之间进行数据交换。线程会阻塞在Exchanger的exchange方法上，直到另一个线程也到了同一个方法，二者进行交换，然后两个线程会继续执行自身相关的代码
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1039.png)   
   
- **Future**：是一个接口，**FutureTask**是一个具体实现类

- Future可以让调用的函数马上返回，继续向下执行，等需要用数据时再用或再来等来数据

- FutureTask帮助实现具体的任务执行以及Future接口中的get等方法的关联

- **并发容器**是线程安全容器的一种，强调的是容器的并发性，也就是说不仅追求线程的安全，还要考虑并发性，提升在容器并发环境下的性能

- 并发容器的思路是尽量不用锁，比较有代表性的是CopyOnWrite和Concurrent开头的几个容器，前者思路是在更改容器的时候把容器写一份进行修改，保证正在读的线程不受影响

- 静态代理：为每个被代理的对象构造对应的代理类

- 动态代理：动态地生成具体委托类的代理类实现对象

- 与静态代理不同，动态代理并不需要为各个委托类逐一实现代理类，只需要为一类代理行为写一个具体的实现类就行了
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1040.png)   
   
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1041.png)   
   
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1042.png)   
   
 ---
  
# 五、服务框架  
- 把应用拆小，方便快速开发，提高稳定性，但是也存在两个问题：
> 数据库连接数的压力仍在
> 
> 系统之间会存在一些重复代码
         
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1043.png)   
 
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1044.png)   
     
- 服务化方案：在应用和底层的数据库、缓存系统、文件系统等系统之间增加了服务层，系统架构更为清晰。从稳定上看，一些散落在多个应用系统中的代码也变成了服务，从而可以由专业的团队进行统一维护，提高代码质量，提高稳定性  
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1045.png)   
     
- **4.2 服务框架的设计与实现**
- **4.2.1 应用从集中式走向分布式所遇到的问题**
> 在没有服务化之前，应用都是通过本地调用方式来使用组件的
> 
> 单机单进程的方法调用其实就只需要把程序计数器指向相应的入口地址，而在多机之间，需要对调用的请求信息进行编码，然后传给远程的节点，解码后再进行真正的调用
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1046.png)   
     
- **4.2.2 服务框架原型**
> 服务框架应该是既包含调用端逻辑又包含服务端逻辑的一个实现
> 
> P104 EXAMPLE
> 
> 一般规则服务器的方式更多地运用在有状态的场景，像数据这种状态要求很高的场景，或者缓存这种尽量要有状态的场景
> 
> 构造请求数据包其实就是把对象变为二进制数据，也就是常说的序列化
> 
> 对于服务端，需要在启动后就进行监听，重点在于持续地接收请求并进行处理
> 
> 在得到具体的服务实例之后，接下来进行服务调用，这一般是通过反射的方式来实现的
  
- **4.2.3 服务调用端的设计与实现**
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1047.png)   
     
> **4.2.3.1 应用与容器的关系**
> 服务框架的部署方式一般有两种：  
> 
>> 一种是把服务框架作为应用的一个依赖包并与应用一起打包：如果要升级服务框架，就需要更新应用本身，因为服务框架是与应用打包放在一起的；并且服务框架没有办法接管classloader，也就不能做一些隔离以及包的实现替换工作
>
>> 另一种是把服务框架作为容器的一部分
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1048.png)   
    
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1049.png)   
    
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1050.png)   
    
>> ClaasLoader是Java中一项非常关键的技术，结构如下图所示。将服务框架自身的类与应用用到的类控制在User-Defined Class Loader级别，这样就实现了相互间的隔离
>
>> 在运行时统一版本时，需要服务框架比应用优先启动，并且把一些需要统一的jar包放到User-Defined Class Loader所公用“祖先”ClassLoader中
          
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/11151.png)   
    
> **4.2.3.2 服务调用者与服务提供者之间通信方式的选择**
> 
>> 采用透明代理与调用者、服务提供者直连的解决方案
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1051.png)   
    
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1052.png)   
    
>> 后者采用了调用者与提供者直接建立连接的方式，引入了一个服务注册查找中心的服务，不需要物理的代理机器  
  
> **4.2.3.3 引入基于接口、方法、参数的路由**
> 
>> 在实际的场景中，一般会用接口作为服务的粒度，也就是说一个服务就是指一个接口的远程实现
>
>> 一般情况下，在一个集群中会提供多个服务，每个服务又有多个方法
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1053.png)   
    
>> 通过路由策略，让其中对于某些服务的请求到一部分机器，让另一些服务的请求到另一部分机器。如下图两个集群的代码其实是完全一样的，是客户端的路由导致了请求的分流
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1054.png)   
    
> **4.2.3.4 多机房场景**  
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1055.png)   
    
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/10551.png)   
    
>> 上述图中，服务注册查找中心会把服务提供者1的所有机器看作是一个集群，尽管分布在两个机房
>
>> 同城机房一般采用光纤直接连接，带宽足够大，延迟也可以接受；如果能避免跨机房调用，能提升系统的稳定性
>
>> 可以在服务注册中心做一些工作来甄别不同机房的调用者集群，给它们不同服务提供者的地址；也可以通过路由来完成，服务注册查找中心给不同机房的调用者相同的服务提供者列表，我们在服务框架内部进行地址过滤，过滤的原则（如何识别机房）一般是基于接口等路由规则进行集中配置管理
>
>> 在实际中，每个机房的网段是不同的，未必每个机房都是对称的（指既有服务调用者又有相应的服务提供者），可以考虑采用虚拟机房的概念，不以物理机房为单位来做路由，把多个机房看作是一个逻辑机房或者把一个物理机房拆分为多个逻辑机房

> **4.2.3.5 服务调用端的流控处理**
>> 流量控制保证系统的稳定性，流控是加载到调用者的控制功能，是为了控制服务提供者的请求的流量
>
>> 一般有两种控制方法： 0-1开关或者设定一个固定的值

> **4.2.3.6 序列化与反序列化处理**
>> Java本身提供了序列化和反序列化的方式，使用简单，需要注意的有： 
> 
>>> 性能与跨语言问题
>
>>> 性能开销
>
>>> 序列化后的长度
>
>> 其实就是 易用性、跨语言、性能、序列化后数据长度等内容

> **4.2.3.7 网络通信实现选择**
> 
>> NIO方式，客户端和服务端的连接时可以复用的，不是每一个请求独占一个连接；BIO方式，调用者和提供者直接会建立大量的连接，会阻塞而使得连接不能得到充分利用
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1056.png)   
    
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1057.png)   
    
>> 同步方式进行远程调用的NIO方式如下图所示  
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1058.png)   
    
> **4.2.3.8 支持多种异步服务调用方式**
> 
>> NIO能完成连接复用以及对调用者的同步调用的支持
>
>> 1 Oneway：只管发送请求而不关心结果，等价于一个不保证可靠传达的同志
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1060.png)   
    
>> 2 Callback：请求方发送请求会后继续执行自己的操作，等对方有响应时进行一个回调。如果不再引入新的线程，回调的执行要么是在IO线程中，要么是在定时任务的线程中，建议用新的线程来执行回调，而不要因为回调本身的代码执行时间久等问题影响了IO线程或定时任务
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1061.png)   
    
>> 3 Future：非常便利的一种方式。通过Future来获取通信结果并直接控制超时，IO线程仍然是从数据队列中得到数据后再进行通信，得到结果后会传给Future
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1062.png)   
    
>> 4 可靠异步：通过消息中间件来保证异步能够在远程被执行
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1063.png)   
    
> **4.2.3.9 使用Future方式对远程服务调用的优化**
> 
>> 一个请求中调用多个远程服务的情况如下
         
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1065.png)   
    
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1066.png)   
    
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1067.png)   
    
>> 上述过程，有一个前提，所调用的服务ABC之间没有相互依赖关系；如果各服务之间存在依赖，只能等到前一个服务返回才能进行后续的服务调用
>
>> 由于有Future方式的支持，且因为底层使用的是NIO方式，因而并行方式没有产生额外的开销
  
- **4.2.4 服务提供端的设计与实现**
         
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/10651.png)   
    
> **4.2.4.1 暴露远程服务**
> 
>> 服务端的工作有两部分，一是对本地服务的注册管理，二是根据进来的请求定位服务并执行

> **4.2.4.2 服务端对请求处理的流程**
> 
>> 无论服务框架以什么方式与应用集成在一起，在启动时都需要监听服务端口。当服务注册都已完成，而且监听端口也准备好时，就只需等着服务调用端的请求进来了
>
>> 请求处理流程如下图所示
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1068.png)   
    
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1069.png)   
  
> **4.2.4.3 执行不同服务的线程池隔离**
> 
>> 服务提供端的工作线程是一个线程池，路由到本地的服务请求会被放入这个线程池执行
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1070.png)   
  
> **4.2.4.4 服务提供端的流控处理**
> 
>> 将执行服务的线程池隔离会带来服务端稳定性的提升，而流控同样是保证服务端稳定性的重要方式
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1071.png)   
  
- **4.2.5 服务升级**  
> 接口不变，只是代码本身进行完善：采用灰度发布的方式验证然后全部发布就可以了
> 
> 修改原有的接口，增加方法：直接增加方法即可
> 
> 修改原有的接口，对接口的某些方法修改调用的参数列表： 1. 对使用原来方法的代码都进行修改，然后和服务端一起发布； 2.通过版本号来解决（常用）； 3.在设计方法上考虑参数的扩展性（可行但不好，意味着采用Map的方式传递参数，不直观且校验复杂）

- 其他
- 要拆分的服务是需要为多方提供公共服务的
- 服务的粒度要根据业务的实际情况来划分
- 服务调用就是为了完成数据的读、写和计算
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1072.png)   
  
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1073.png)   
  
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1074.png)   
  
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1075.png)   
  
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1076.png)   
  
- 服务治理分为 管理服务 和 查看服务 两个方面
- 服务框架与ESB的对比：
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1077.png)   
  
        
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/1078.png)   
  
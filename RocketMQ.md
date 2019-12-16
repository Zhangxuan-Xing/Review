<!-- GFM-TOC -->

* [一、思想目标](#一思想目标)

* [二、路由中心 NameServer](#二路由中心 NameServer)

* [三、数据类型](#三数据类型)

* [四、数据结构](#四数据结构)

* [五、基础应用](#五基础应用)

* [六、Redis 与 Memcached](#六redis-与-memcached)

  <!-- GFM-TOC -->



## 一、思想目标

### 1.思想

1. 基于主题Topic的发布与订阅模式

2. 简单 + 高性能

   > 例：NameServer集群间互不通信，且集群间追求最终一致性而非强一致；消息允许且可能被重复消费，此问题一般交由消费时进行幂等操作。

3. IO高效存储机制

   > 例：追求消息发送的高吞吐量，将存储文件设计成文件组，组内单个文件大小固定，方便引入内存映射机制，通过顺序写提升写性能，同时引入消费队列文件和索引文件来兼顾消息消费和查找。

### 2.目标

#### 2.1 核心功能

​	消息发送、消息存储、消息消费

#### 2.2 架构模式

​	发布订阅模式，基本组件：消息发送者、消息服务器、消息消费、路由发现

#### 2.3 顺序消费

​	原则上可以保证消息有序

#### 2.4 消息过滤

> Broker端过滤：只将消息消费者感兴趣的消息进行发送
>
> 消息消费端过滤：完全由消费者自定义，但无用消息依旧会从Broker传输到消费端

#### 2.5 消息存储

- 考量维度：消息堆积能力 + 消息存储性能
- 维护高性能措施：引入内存映射机制，所有消息顺序存储在同一文件
- 避免无限堆积措施：引入消息文件过期机制和文件存储空间报警机制

#### 2.6 高可用性

​	注意区分是不是单点故障，以及同步刷盘还是异步刷盘；一般情况（Broker 异常 Crash、OS Crash、断电）下同步刷盘机制可以确保不丢失信息，异步刷盘会丢失少量消息。

#### 2.7 至少消费一次

​	通过消费确认机制（ACK）来保证消息至少被消费一次，但可能重复消费。

---

## 二、路由中心 NameServer

### 1.架构设计

​	Broker服务器启动时向所有NameServer注册，Producer在发送消息前从NameServer获取Broker服务器地址列表，然后根据负载算法从列表中选择一台服务器进行消息发送。

​	NameServer与每台Broker服务器保持长连接，间隔10s检测Broker是否存活，如果检测到Broker宕机则从路由注册表中移除。

​	通过部署多台NameServer服务器来实现本身的高可用，彼此间互不通信，换句话说，某一时刻的数据并不会完全相同，但对消息不会造成任何影响。

### 2.启动流程

#### 2.1 解析填充参数

```java
// 创建 NameServerConfig，主要为一些业务参数
final NamesrvConfig namesrvConfig = new NamesrvConfig();
// 创建 NettyServerConfig，主要为一些网络参数
final NettyServerConfig nettyServerConfig = new NettyServerConfig();
nettyServerConfig.setListenPort(9876);
// 在解析启动时，将配置文件或启动命令中的选项值填充至上述对象
// 如“-c configFile”命令配置路径、“-- 属性名 属性值”配置属性值
if (commandLine.hasOption('c')) {
    String file = commandLine.getOptionValue('c');
    if (file != null) {
        InputStream in = new BufferedInputStream(new FileInputStream(file));
        properties = new Properties();
        properties.load(in);
        MixAll.properties2Object(properties, namesrvConfig);
        MixAll.properties2Object(properties, nettyServerConfig);

        namesrvConfig.setConfigStorePath(file);

        System.out.printf("load config properties file OK, %s%n", file);
        in.close();
    }
}

if (commandLine.hasOption('p')) {
    InternalLogger console = InternalLoggerFactory.getLogger(LoggerName.NAMESRV_CONSOLE_NAME);
    MixAll.printObjectProperties(console, namesrvConfig);
    MixAll.printObjectProperties(console, nettyServerConfig);
    System.exit(0);
}
```

2.1.1 NameServerConfig 属性

```java
// 主目录
// 可通过 -Drocketmq.home.dir=path或通过设置环境变量ROCKETMQ_HOME来配置
private String rocketmqHome = System.getProperty(MixAll.ROCKETMQ_HOME_PROPERTY, System.getenv(MixAll.ROCKETMQ_HOME_ENV));
// 存储KV配置属性的持久化路径
private String kvConfigPath = System.getProperty("user.home") + File.separator + "namesrv" +
        File.separator + "kvConfig.json";
// 默认配置文件路径
private String configStorePath = System.getProperty("user.home") + File.separator + "namesrv" +
        File.separator + "namesrv.properties";
private String productEnvName = "center";
// 是否开启集群测试
private boolean clusterTest = false;
// 是否支持顺序消息
private boolean orderMessageEnable = false;
```

2.1.2 NettyServerConfig 属性

```java
// 监听端口，会被初始化为9876
private int listenPort = 8888;
// 业务线程池线程个数
private int serverWorkerThreads = 8;
// public 任务线程池线程个数
private int serverCallbackExecutorThreads = 0;
// IO线程池线程个数
private int serverSelectorThreads = 3;
private int serverOnewaySemaphoreValue = 256;
// 异步消息发送最大并发度（Broker端）
private int serverAsyncSemaphoreValue = 64;
private int serverChannelMaxIdleTimeSeconds = 120;

// 网络socket发送缓存区大小，默认64K
private int serverSocketSndBufSize = NettySystemConfig.socketSndbufSize;
private int serverSocketRcvBufSize = NettySystemConfig.socketRcvbufSize;
// ByteBuffer 是否开启缓存
private boolean serverPooledByteBufAllocatorEnable = true;
```

#### 2.2 创建并初始化实例

```java
public boolean initialize() {
    // 加载KV配置
    this.kvConfigManager.load();
    this.remotingServer = new NettyRemotingServer(this.nettyServerConfig, this.brokerHousekeepingService);
    // 开启网络处理对象
    this.remotingExecutor =
        Executors.newFixedThreadPool(nettyServerConfig.getServerWorkerThreads(), 
                new ThreadFactoryImpl("RemotingExecutorThread_"));
    this.registerProcessor();
    
    // 每隔10s扫描一次Broker，移除不激活状态的Broker
    this.scheduledExecutorService.scheduleAtFixedRate(new Runnable() {

        @Override
        public void run() {
            NamesrvController.this.routeInfoManager.scanNotActiveBroker();
        }
    }, 5, 10, TimeUnit.SECONDS);

    // 每隔10s打印一次KV配置
    this.scheduledExecutorService.scheduleAtFixedRate(new Runnable() {

        @Override
        public void run() {
            NamesrvController.this.kvConfigManager.printAllPeriodically();
        }
    }, 1, 10, TimeUnit.MINUTES);
    ...
}
```

#### 2.3 注册钩子函数并启动服务

```java
// 注册JVM钩子函数并启动服务器，以便监听Broker、生产者网络请求
// 钩子函数：在JVM关闭时会执行其中方法，常用于线程等清理工作
Runtime.getRuntime().addShutdownHook(new ShutdownHookThread(log, new Callable<Void>() {
    @Override
    public Void call() throws Exception {
        controller.shutdown();
        return null;
    }
}));

// 启动服务器
controller.start();
```

### 3.路由注册

#### 3.1 路由元信息

​	一个Topic拥有多个消息队列，一个Broker为每一主题默认创建4个读队列、4个写队列。多个Broker组成一个集群，BrokerName由相同的多台Broker组成M-S架构，brokerId为0代表Master，大于0代表Slave，BrokerLiveInfo中的lastUpadateTimestamp存储上次收到心跳包的时间。

```java
/**
 * 路由实现类
 */
public class RouteInfoManager {
    private static final InternalLogger log = InternalLoggerFactory.getLogger(LoggerName.NAMESRV_LOGGER_NAME);
    private final static long BROKER_CHANNEL_EXPIRED_TIME = 1000 * 60 * 2;
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    
    // Topic消息队列路由信息，消息发送时根据路由表进行负载均衡
    private final HashMap<String/* topic */, List<QueueData>> topicQueueTable;
    // Broker基础信息，包括name、所属集群名称、主备Broker地址等
    private final HashMap<String/* brokerName */, BrokerData> brokerAddrTable;
    // Broker集群信息，存储集群中所有Broker名称
    private final HashMap<String/* clusterName */, Set<String/* brokerName */>> clusterAddrTable;
    // Broker状态信息，NameServer每次收到心跳包时替换该信息
    private final HashMap<String/* brokerAddr */, BrokerLiveInfo> brokerLiveTable;
    // Broker上的FilterServer列表，用于类模式消息过滤
    private final HashMap<String/* brokerAddr */, List<String>/* Filter Server */> filterServerTable;
    ...
}
```

#### 3.2 路由注册

​	通过Broker和NameServer的**心跳功能实现**，Broker启动时向集群中所有NameServer发送心跳语句，**每隔30s**向所有NameServer发送心跳包，NameServer收到心跳包会更新BrokerLiveInfo的lastUpateTimestamp，NameServer**每隔10s**扫描brokerLiveTable，如果**连续120s**没有收到心跳包，则移除该Broker的路由信息并关闭Socket连接。

##### 3.2.1 发送心跳包

​	待写
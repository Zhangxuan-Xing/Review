<!-- GFM-TOC -->

* [一、思想目标](#一思想目标)

* [二、路由中心NameServer](#二路由中心NameServer)

* [三、消息发送](#三消息发送)

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

## 二、路由中心NameServer

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

![路由注册.jpg](https://segmentfault.com/img/bVbzHjJ?w=697&h=391)

##### 3.2.1 发送心跳包

​	遍历NameServer列表，Broker消息服务器依次向NameServer发送心跳包。

```java
this.scheduledExecutorService.scheduleAtFixedRate(new Runnable() {

    @Override
    public void run() {
        try {
            BrokerController.this.registerBrokerAll(true, false, brokerConfig.isForceRegister());
        } catch (Throwable e) {
            log.error("registerBrokerAll Exception", e);
        }
    }
}, 1000 * 10, Math.max(10000, Math.min(brokerConfig.getRegisterNameServerPeriod(), 60000)), TimeUnit.MILLISECONDS);

// registerBrokerAll 方法如下
    public List<RegisterBrokerResult> registerBrokerAll(
        final String clusterName,
        final String brokerAddr,
        final String brokerName,
        final long brokerId,
        final String haServerAddr,
        final TopicConfigSerializeWrapper topicConfigWrapper,
        final List<String> filterServerList,
        final boolean oneway,
        final int timeoutMills,
        final boolean compressed) {

        final List<RegisterBrokerResult> registerBrokerResultList = Lists.newArrayList();
        List<String> nameServerAddressList = this.remotingClient.getNameServerAddressList();
        if (nameServerAddressList != null && nameServerAddressList.size() > 0) {

            final RegisterBrokerRequestHeader requestHeader = new RegisterBrokerRequestHeader();
            requestHeader.setBrokerAddr(brokerAddr);
            requestHeader.setBrokerId(brokerId);
            requestHeader.setBrokerName(brokerName);
            requestHeader.setClusterName(clusterName);
            // master地址，初次请求为空，slave向NameServer注册后返回
            requestHeader.setHaServerAddr(haServerAddr);
            requestHeader.setCompressed(compressed);

            RegisterBrokerBody requestBody = new RegisterBrokerBody();
            requestBody.setTopicConfigSerializeWrapper(topicConfigWrapper);
            // 设置消息过滤服务器列表
            requestBody.setFilterServerList(filterServerList);
            final byte[] body = requestBody.encode(compressed);
            final int bodyCrc32 = UtilAll.crc32(body);
            requestHeader.setBodyCrc32(bodyCrc32);
            final CountDownLatch countDownLatch = new CountDownLatch(nameServerAddressList.size());
            // 遍历所有NameServer列表
            for (final String namesrvAddr : nameServerAddressList) {
                brokerOuterExecutor.execute(new Runnable() {
                    @Override
                    public void run() {
                        try {
                            // 分别向NameServer注册
                            RegisterBrokerResult result = registerBroker(namesrvAddr,oneway, timeoutMills,requestHeader,body);
                            if (result != null) {
                                registerBrokerResultList.add(result);
                            }

                            log.info("register broker[{}]to name server {} OK", brokerId, namesrvAddr);
                        } catch (Exception e) {
                            log.warn("registerBroker Exception, {}", namesrvAddr, e);
                        } finally {
                            countDownLatch.countDown();
                        }
                    }
                });
            }

            try {
                countDownLatch.await(timeoutMills, TimeUnit.MILLISECONDS);
            } catch (InterruptedException e) {
            }
        }

        return registerBrokerResultList;
    }

// registerBroker 方法如下
    private RegisterBrokerResult registerBroker(
        final String namesrvAddr,
        final boolean oneway,
        final int timeoutMills,
        final RegisterBrokerRequestHeader requestHeader,
        final byte[] body
    ) throws RemotingCommandException, MQBrokerException, RemotingConnectException, RemotingSendRequestException, RemotingTimeoutException,
        InterruptedException {
        RemotingCommand request = RemotingCommand.createRequestCommand(RequestCode.REGISTER_BROKER, requestHeader);
        request.setBody(body);
 	
        if (oneway) {
            try {
                this.remotingClient.invokeOneway(namesrvAddr, request, timeoutMills);
            } catch (RemotingTooMuchRequestException e) {
                // Ignore
            }
            return null;
        }

        RemotingCommand response = this.remotingClient.invokeSync(namesrvAddr, request, timeoutMills);
        assert response != null;
        switch (response.getCode()) {
            case ResponseCode.SUCCESS: {
                RegisterBrokerResponseHeader responseHeader =
                    (RegisterBrokerResponseHeader) response.decodeCommandCustomHeader(RegisterBrokerResponseHeader.class);
                RegisterBrokerResult result = new RegisterBrokerResult();
                result.setMasterAddr(responseHeader.getMasterAddr());
                result.setHaServerAddr(responseHeader.getHaServerAddr());
                if (response.getBody() != null) {
                    result.setKvTable(KVTable.decode(response.getBody(), KVTable.class));
                }
                return result;
            }
            default:
                break;
        }

        throw new MQBrokerException(response.getCode(), response.getRemark());
    }
```

##### 3.2.2 处理心跳包

① 加写锁

```java
// 路由注册需加写锁，防止并发修改路由表
this.lock.writeLock().lockInterruptibly();

// 判断Broker所属集群是否存在，如不存在则进行创建，并把brokerName加入其中
Set<String> brokerNames = this.clusterAddrTable.get(clusterName);
if (null == brokerNames) {
    brokerNames = new HashSet<String>();
    this.clusterAddrTable.put(clusterName, brokerNames);
}
brokerNames.add(brokerName);
```

② 维护BrokerData信息

```java
// 根据brokerName获取Broker信息，如不存在就创建并添加至table
BrokerData brokerData = this.brokerAddrTable.get(brokerName);
if (null == brokerData) {
    registerFirst = true;
    brokerData = new BrokerData(clusterName, brokerName, new HashMap<Long, String>());
    this.brokerAddrTable.put(brokerName, brokerData);
}
Map<Long, String> brokerAddrsMap = brokerData.getBrokerAddrs();
//Switch slave to master: first remove <1, IP:PORT> in namesrv, then add <0, IP:PORT>
//The same IP:PORT must only have one record in brokerAddrTable
// 移除当前brokerAdd为Value的值，因为地址+端口不能重复
Iterator<Entry<Long, String>> it = brokerAddrsMap.entrySet().iterator();
while (it.hasNext()) {
    Entry<Long, String> item = it.next();
    if (null != brokerAddr && brokerAddr.equals(item.getValue()) && brokerId != item.getKey()) {
        it.remove();
    }
}

String oldAddr = brokerData.getBrokerAddrs().put(brokerId, brokerAddr);
// 判断是否首次创建，非第一次创建为false
registerFirst = registerFirst || (null == oldAddr);
```

③ 更新信息

```java
if (null != topicConfigWrapper
    && MixAll.MASTER_ID == brokerId) {
    // 如果Broker为Master且配置信息发生变化或首次注册
    if (this.isBrokerTopicConfigChanged(brokerAddr, topicConfigWrapper.getDataVersion())
        || registerFirst) {
        ConcurrentMap<String, TopicConfig> tcTable =
            topicConfigWrapper.getTopicConfigTable();
        if (tcTable != null) {
        // 更新信息
            for (Map.Entry<String, TopicConfig> entry : tcTable.entrySet()) {
                this.createAndUpdateQueueData(brokerName, entry.getValue());
            }
        }
    }
}
```

④ 根据topicConfig创建QueueData，更新topicQueueTable

```java
private void createAndUpdateQueueData(final String brokerName, final TopicConfig topicConfig) {
    QueueData queueData = new QueueData();
    queueData.setBrokerName(brokerName);
    queueData.setWriteQueueNums(topicConfig.getWriteQueueNums());
    queueData.setReadQueueNums(topicConfig.getReadQueueNums());
    queueData.setPerm(topicConfig.getPerm());
    queueData.setTopicSynFlag(topicConfig.getTopicSysFlag());

    List<QueueData> queueDataList = this.topicQueueTable.get(topicConfig.getTopicName());
    if (null == queueDataList) {
        queueDataList = new LinkedList<QueueData>();
        queueDataList.add(queueData);
        this.topicQueueTable.put(topicConfig.getTopicName(), queueDataList);
        log.info("new topic registered, {} {}", topicConfig.getTopicName(), queueData);
    } else {
        boolean addNewOne = true;

        Iterator<QueueData> it = queueDataList.iterator();
        while (it.hasNext()) {
            QueueData qd = it.next();
            if (qd.getBrokerName().equals(brokerName)) {
                if (qd.equals(queueData)) {
                    addNewOne = false;
                } else {
                    log.info("topic changed, {} OLD: {} NEW: {}", topicConfig.getTopicName(), qd,
                        queueData);
                    it.remove();
                }
            }
        }

        if (addNewOne) {
            queueDataList.add(queueData);
        }
    }
}
```

⑤ 更新BrokerLiveInfo

```java
BrokerLiveInfo prevBrokerLiveInfo = this.brokerLiveTable.put(brokerAddr,
    new BrokerLiveInfo(
        System.currentTimeMillis(),
        topicConfigWrapper.getDataVersion(),
        channel,
        haServerAddr));
if (null == prevBrokerLiveInfo) {
    log.info("new broker registered, {} HAServer: {}", brokerAddr, haServerAddr);
}
```

⑥ 注册Broker的过滤Server地址列表

```java
if (filterServerList != null) {
    if (filterServerList.isEmpty()) {
        this.filterServerTable.remove(brokerAddr);
    } else {
        // 一个broker会关联多个filterServer消息过滤服务器
        this.filterServerTable.put(brokerAddr, filterServerList);
    }
}

if (MixAll.MASTER_ID != brokerId) {
    String masterAddr = brokerData.getBrokerAddrs().get(MixAll.MASTER_ID);
    if (masterAddr != null) {
        BrokerLiveInfo brokerLiveInfo = this.brokerLiveTable.get(masterAddr);
        // 找到master节点信息并更新masterAddr属性
        if (brokerLiveInfo != null) {
            result.setHaServerAddr(brokerLiveInfo.getHaServerAddr());
            result.setMasterAddr(masterAddr);
        }
    }
}
```

#### 3.3 路由删除

- 正常情况：broker正常关闭，执行unregisterBroker指令；
- 非正常情况：每隔10s调用scanNotActiveBroker检测lastUpdateTimestamp上次心跳包和当前系统时间的时间差，大于120s就移除该broker，关闭与broker的连接，并同时更新topicQueueTable、brokerAddrTable、brokerLiveTable、filterServerTable。

```java
    /**
     * 在NameServer中每10s执行一次
     */
    public void scanNotActiveBroker() {
        Iterator<Entry<String, BrokerLiveInfo>> it = this.brokerLiveTable.entrySet().iterator();
        // 遍历brokerLiveInfo路由表
        while (it.hasNext()) {
            Entry<String, BrokerLiveInfo> next = it.next();
            long last = next.getValue().getLastUpdateTimestamp();
            // 判断时间差是否大于120s
            if ((last + BROKER_CHANNEL_EXPIRED_TIME) < System.currentTimeMillis()) {
                // 关闭连接
                RemotingUtil.closeChannel(next.getValue().getChannel());
                // 移除
                it.remove();
                log.warn("The broker channel expired, {} {}ms", next.getKey(), BROKER_CHANNEL_EXPIRED_TIME);
                // 删除相关路由信息 - 该方法需要申请写锁
                this.onChannelDestroy(next.getKey(), next.getValue().getChannel());
            }
        }
    }
```

①  申请写锁并维护brokerAddrTable

```java
// 申请写锁，根据brokerAddr从brokerLiveTable、filterServerTable移除
this.lock.writeLock().lockInterruptibly();
this.brokerLiveTable.remove(brokerAddrFound);
this.filterServerTable.remove(brokerAddrFound);
String brokerNameFound = null;
boolean removeBrokerName = false;
Iterator<Entry<String, BrokerData>> itBrokerAddrTable =
    this.brokerAddrTable.entrySet().iterator();
while (itBrokerAddrTable.hasNext() && (null == brokerNameFound)) {
    BrokerData brokerData = itBrokerAddrTable.next().getValue();

    Iterator<Entry<Long, String>> it = brokerData.getBrokerAddrs().entrySet().iterator();
    while (it.hasNext()) {
        Entry<Long, String> entry = it.next();
        Long brokerId = entry.getKey();
        String brokerAddr = entry.getValue();
        if (brokerAddr.equals(brokerAddrFound)) {
            brokerNameFound = brokerData.getBrokerName();
            it.remove();
            log.info("remove brokerAddr[{}, {}] from brokerAddrTable, because channel destroyed",
                brokerId, brokerAddr);
            break;
        }
    }

    // 移除broker后如不再包含其他broker，则在brokerAddrTable中移除对应条目
    if (brokerData.getBrokerAddrs().isEmpty()) {
        removeBrokerName = true;
        itBrokerAddrTable.remove();
        log.info("remove brokerName[{}] from brokerAddrTable, because channel destroyed",
            brokerData.getBrokerName());
    }
}

if (brokerNameFound != null && removeBrokerName) {
    Iterator<Entry<String, Set<String>>> it = this.clusterAddrTable.entrySet().iterator();
    while (it.hasNext()) {
        Entry<String, Set<String>> entry = it.next();
        String clusterName = entry.getKey();
        Set<String> brokerNames = entry.getValue();
        // 从集群中移除
        boolean removed = brokerNames.remove(brokerNameFound);
        if (removed) {
            log.info("remove brokerName[{}], clusterName[{}] from clusterAddrTable, because channel destroyed",
                brokerNameFound, clusterName);
            
            // 如移除后集群不包含broker，则将该集群移除
            if (brokerNames.isEmpty()) {
                log.info("remove the clusterName[{}] from clusterAddrTable, because channel destroyed and no broker in this cluster",
                    clusterName);
                it.remove();
            }

            break;
        }
    }
}
```

② 根据brokerName遍历主题队列

```java
if (removeBrokerName) {
    Iterator<Entry<String, List<QueueData>>> itTopicQueueTable =
        this.topicQueueTable.entrySet().iterator();
    while (itTopicQueueTable.hasNext()) {
        Entry<String, List<QueueData>> entry = itTopicQueueTable.next();
        String topic = entry.getKey();
        List<QueueData> queueDataList = entry.getValue();

        Iterator<QueueData> itQueueData = queueDataList.iterator();
        while (itQueueData.hasNext()) {
            QueueData queueData = itQueueData.next();
            if (queueData.getBrokerName().equals(brokerNameFound)) {
                itQueueData.remove();
                log.info("remove topic[{} {}], from topicQueueTable, because channel destroyed",
                    topic, queueData);
            }
        }

        if (queueDataList.isEmpty()) {
            itTopicQueueTable.remove();
            log.info("remove topic[{}] all queue, from topicQueueTable, because channel destroyed",
                topic);
        }
    }
}
```

③ 释放锁

```java
finally {
    this.lock.writeLock().unlock();
}
```

#### 3.4 路由发现

​	路由发现并不是实时的，当topic路由发生变化后，NameServer不主动推送给客户端，而是由客户端定时拉取最新路由。

```java
public RemotingCommand getRouteInfoByTopic(ChannelHandlerContext ctx,
    RemotingCommand request) throws RemotingCommandException {
    final RemotingCommand response = RemotingCommand.createResponseCommand(null);
    final GetRouteInfoRequestHeader requestHeader =
        (GetRouteInfoRequestHeader) request.decodeCommandCustomHeader(GetRouteInfoRequestHeader.class);

    // 填充 TopicRouteData
    TopicRouteData topicRouteData = this.namesrvController.getRouteInfoManager().pickupTopicRouteData(requestHeader.getTopic());

    if (topicRouteData != null) {
        // 如果为顺序消息，则从KVConfig中获取关于顺序消息的配置填充路由信息
        if (this.namesrvController.getNamesrvConfig().isOrderMessageEnable()) {
            String orderTopicConf =
                this.namesrvController.getKvConfigManager().getKVConfig(NamesrvUtil.NAMESPACE_ORDER_TOPIC_CONFIG,
                    requestHeader.getTopic());
            topicRouteData.setOrderTopicConf(orderTopicConf);
        }

        byte[] content = topicRouteData.encode();
        response.setBody(content);
        response.setCode(ResponseCode.SUCCESS);
        response.setRemark(null);
        return response;
    }

    response.setCode(ResponseCode.TOPIC_NOT_EXIST);
    response.setRemark("No topic route info in name server for the topic: " + requestHeader.getTopic()
        + FAQUrl.suggestTodo(FAQUrl.APPLY_TOPIC_URL));
    return response;
}
```



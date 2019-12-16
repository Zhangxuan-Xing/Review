<!-- GFM-TOC -->
* [一、NoSQL](#一NoSQL)
* [二、Redis概述](#二Redis概述)
* [三、数据类型](#三数据类型)
* [四、数据结构](#四数据结构)
* [五、基础应用](#五基础应用)
* [六、Redis 与 Memcached](#六redis-与-memcached)
* [七、Redis事务](#七Redis事务)
* [八、持久化](#八持久化)
* [九、复制](#九复制)
* [十、键的过期时间](#十键的过期时间)
* [十一、数据淘汰策略](#十一数据淘汰策略)
* [十二、事件](#十二事件)
<!-- GFM-TOC -->

# 一、NoSQL  
  
## 背景  
**1）远离单机时代**  
在90年代，一个网站的访问量一般都不大，用单个数据库完全可以轻松应付。在那个时候，更多的都是静态网页，动态交互类型的网站不多。  
上述架构下，数据存储的瓶颈有以下几种可能：  

- 数据量的总大小 一个机器放不下时
- 数据的索引（B+ Tree）一个机器的内存放不下时
- 访问量(读写混合)一个实例不能承受

**2）Memcached(缓存)+MySQL+垂直拆分**  
随着访问量的上升，几乎大部分使用MySQL架构的网站在数据库上都开始出现了性能问题，web程序不再仅仅专注在功能上，同时也在追求性能。程序员们开始大量的使用缓存技术来缓解数据库的压力，优化数据库的结构和索引。开始比较流行的是通过文件缓存来缓解数据库压力，但是当访问量继续增大的时候，多台web机器通过文件缓存不能共享，大量的小文件缓存也带了了比较高的IO压力。在这个时候，Memcached就自然的成为一个非常时尚的技术产品。  
   
Memcached作为一个独立的分布式的缓存服务器，为多个web服务器提供了一个共享的高性能缓存服务，在Memcached服务器上，又发展了根据hash算法来进行多台Memcached缓存服务的扩展，然后又出现了一致性hash来解决增加或减少缓存服务器导致重新hash带来的大量缓存失效的弊端。    
  
**3）MySQL主从读写分离**  
由于数据库的写入压力增加，Memcached只能缓解数据库的读取压力。读写集中在一个数据库上让数据库不堪重负，大部分网站开始使用主从复制技术来达到读写分离，以提高读写性能和读库的可扩展性。MySQL的master-slave模式成为这个时候的网站标配了。  
  
**4）分表分库+水平拆分+mysql集群**   
在Memcached的高速缓存，MySQL的主从复制，读写分离的基础之上，这时MySQL主库的写压力开始出现瓶颈，而数据量的持续猛增，由于MyISAM使用表锁，在高并发下会出现严重的锁问题，大量的高并发MySQL应用开始使用InnoDB引擎代替MyISAM。  
  
同时，开始流行使用分表分库来缓解写压力和数据增长的扩展问题。这个时候，分表分库成了一个热门技术，是面试的热门问题也是业界讨论的热门技术问题。也就在这个时候，MySQL推出了还不太稳定的表分区，这也给技术实力一般的公司带来了希望。虽然MySQL推出了MySQL Cluster集群，但性能也不能很好满足互联网的要求，只是在高可靠性上提供了非常大的保证。  
  
**5）MySQL的扩展性瓶颈**  
MySQL数据库也经常存储一些大文本字段，导致数据库表非常的大，在做数据库恢复的时候就导致非常的慢，不容易快速恢复数据库。比如1000万4KB大小的文本就接近40GB的大小，如果能把这些数据从MySQL省去，MySQL将变得非常的小。关系数据库很强大，但是它并不能很好的应付所有的应用场景。MySQL的扩展性差（需要复杂的技术来实现），大数据下IO压力大，表结构更改困难，正是当前使用MySQL的开发人员面临的问题。  
  
**综上背景，使用NoSQL的原因如下：**  
今天我们可以通过第三方平台（如：Google,Facebook等）可以很容易的访问和抓取数据。  
用户的个人信息，社交网络，地理位置，用户生成的数据和用户操作日志已经成倍的增加。  
我们如果要对这些用户数据进行挖掘，那SQL数据库已经不适合这些应用了, NoSQL数据库的发展也却能很好的处理这些大的数据。  
  
## 3V 和 3高  

- 3V：海量Volume、多样Variety、实时Velocity  
- 3高：高并发、高可扩、高性能  
  
## 概念  
NoSQL(NoSQL = Not Only SQL)，意即“不仅仅是SQL”，泛指非关系型的数据库。随着互联网web2.0网站的兴起，传统的关系数据库在应付web2.0网站，特别是超大规模和高并发的SNS类型的web2.0纯动态网站已经显得力不从心，暴露了很多难以克服的问题，而非关系型的数据库则由于其本身的特点得到了非常迅速的发展。NoSQL数据库的产生就是为了解决大规模数据集合多重数据种类带来的挑战，尤其是大数据应用难题，包括超大规模数据的存储。
  
（例如谷歌或Facebook每天为他们的用户收集万亿比特的数据），这些类型的数据存储不需要固定的模式，无需多余操作就可以横向扩展。  
  
## 特征  
  
- **易扩展**  
NoSQL数据库种类繁多，但是一个共同的特点都是去掉关系数据库的关系型特性。  
数据之间无关系，这样就非常容易扩展。也无形之间，在架构的层面上带来了可扩展的能力。  

- **大数据量高性能**  
NoSQL数据库都具有非常高的读写性能，尤其在大数据量下，同样表现优秀。  
这得益于它的无关系性，数据库的结构简单。  
一般MySQL使用Query Cache，每次表的更新Cache就失效，是一种大粒度的Cache，在针对web2.0的交互频繁的应用，Cache性能不高。而NoSQL的Cache是记录级的，是一种细粒度的Cache，所以NoSQL在这个层面上来说就要性能高很多了。  
  
- **多样灵活的数据模型**  
NoSQL无需事先为要存储的数据建立字段，随时可以存储自定义的数据格式。而在关系数据库里，增删字段是一件非常麻烦的事情。如果是非常大数据量的表，增加字段简直就是一个噩梦。  
  
- **传统RDBMS VS NOSQL**  
  
RDBMS

- 高度组织化结构化数据
- 结构化查询语言（SQL）
- 数据和关系都存储在单独的表中。
- 数据操纵语言，数据定义语言
- 严格的一致性
- 基础事务


NoSQL

- 代表着不仅仅是SQL
- 没有声明性查询语言
- 没有预定义的模式
- 键值对存储，列存储，文档存储，图形数据库
- 最终一致性，而非ACID属性
- 非结构化和不可预知的数据
- CAP定理
- 高性能，高可用性和可伸缩性
  
## 场景  

- K-V
- Cache
- Persistence  
  
## 模型分类  
  
**聚合模型**  
  
- KV键值
已较为常见，如新浪：BerkeleyDB+redis、美团：redis+tair以及阿里：memcache+redis  

- 文档型数据库（Bson为主）  
CouchDB、MongoDB  
  
**补充**：MongoDB 是一个基于分布式文件存储的数据库。由 C++ 语言编写。旨在为 WEB 应用提供可扩展的高性能数据存储解决方案。  
MongoDB 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。   
  
- 列族  
Cassandra、HBase以及分布式文件系统  
  
- 图形  
1）它不是放图形的，放的是关系比如:朋友圈社交网络、广告推荐系统  
2）社交网络，推荐系统等（专注于构建关系图谱）
如Neo4J, InfoGrid  
  
## CAP  
  
- Consistency（强一致性）
- Availability（可用性）
- Partition tolerance（分区容错性）

**三选二原则**  
CAP理论就是说在分布式存储系统中，最多只能实现上面的两点。  
而由于当前的网络硬件肯定会出现延迟丢包等问题，所以分区容忍性是我们必须需要实现的。  
所以通常只能在一致性和可用性之间进行权衡，没有NoSQL系统能同时保证这三点。  
  
C:强一致性 A：高可用性 P：分布式容忍性  
 CA 传统Oracle数据库  
 AP 大多数网站架构的选择  
 CP Redis、Mongodb  
  
**注意**：分布式架构的时候必须做出取舍。  
*一致性和可用性之间取一个平衡，多余大多数web应用，其实并不需要强一致性*  
  
因此牺牲C换取P，这是目前分布式数据库产品的方向  
对于web2.0网站来说，关系数据库的很多主要特性却往往无用武之地  

---
  
# 二、Redis概述 
    
## 介绍  
Redis:Remote Dictionary Server(远程字典服务器)  
  
Redis 是速度非常快的非关系型（NoSQL）内存键值数据库，可以存储键和五种不同类型的值之间的映射。

键的类型只能为字符串，值支持五种数据类型：字符串、列表、集合、散列表、有序集合。

Redis 支持很多特性，例如将内存中的数据持久化到硬盘中，使用复制来扩展读性能，使用分片来扩展写性能。  
    
是完全开源免费的，用C语言编写的，遵守BSD协议，是一个高性能的(key/value)分布式内存数据库，基于内存运行。支持持久化的NoSQL数据库，是当前最热门的NoSql数据库之一，也被人们称为数据结构服务器。  
  
Redis 与其他 key - value 缓存产品有以下**三个特点**：  

- Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用
- Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储
- Redis支持数据的备份，即master-slave模式的数据备份
   
## 用法  

- 内存存储和持久化：redis支持异步将内存中的数据写到硬盘上，同时不影响继续服务  
- 取最新N个数据的操作，如：可以将最新的10条评论的ID放在Redis的List集合里面  
- 模拟类似于HttpSession这种需要设定过期时间的功能  
- 发布、订阅消息系统  
- 定时器、计数器   
  

## 其他  
  
- **单进程**模型来处理客户端的请求。对读写等事件的响应，是通过对epoll函数的包装来做到的。Redis的实际处理速度完全依靠主进程的执行效率  
- epoll是Linux内核为处理大批量文件描述符而作了改进的epoll，是Linux下多路复用IO接select/poll的增强版本，它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率
- 默认16个数据库，类似数组下表从零开始，初始默认使用零号库
- select命令切换数据库
- dbsize查看当前数据库的key的数量
- flushdb：清空当前库
- Flushall；通杀全部库
- 统一密码管理，16个库都是同样密码，要么都成功，要么都失败
- Redis索引都是从零开始
- 默认端口是6379  
  

---

# 三、数据类型

| 数据类型 | 可以存储的值 | 操作 |
| :--: | :--: | :--: |
| STRING | 字符串、整数或者浮点数 | 对整个字符串或者字符串的其中一部分执行操作</br> 对整数和浮点数执行自增或者自减操作 |
| LIST | 列表 | 从两端压入或者弹出元素 </br> 对单个或者多个元素</br> 进行修剪，只保留一个范围内的元素 |
| SET | 无序集合 | 添加、获取、移除单个元素</br> 检查一个元素是否存在于集合中</br> 计算交集、并集、差集</br> 从集合里面随机获取元素 |
| HASH | 包含键值对的无序散列表 | 添加、获取、移除单个键值对</br> 获取所有键值对</br> 检查某个键是否存在|
| ZSET | 有序集合 | 添加、获取、删除元素</br> 根据分值范围或者成员来获取元素</br> 计算一个键的排名 |
  
[用法全解](http://redisdoc.com/)
   
## STRING
  
String是redis最基本的类型，你可以理解成与Memcached一模一样的类型，一个key对应一个value。
  
String类型是二进制安全的。意思是redis的String可以包含任何数据。比如jpg图片或者序列化的对象 。String类型是Redis最基本的数据类型，一个redis中字符串value最多可以是512M。   
 

```html
> set hello world
OK
> get hello
"world"
> del hello
(integer) 1
> get hello
(nil)
```
  
**案例**  
 
-  set/get/del/append/strlen  

-  incr/decr/incrby/decrby,一定要是数字才能进行加减  

-  getrange/setrange  
*getrange:获取指定区间范围内的值，类似between......and的关系
从零到负一表示全部*  

-  setex(set with expire)键秒值/setnx(set if not exist)  
*setex:设置带过期时间的key，动态设置；setex 键 秒值 真实值；setnx:只有在 key 不存在时设置 key 的值*  

-  mset/mget/msetnx  
*mset:同时设置一个或多个 key-value 对；mget:获取所有(一个或多个)给定 key 的值；msetnx:同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在*  

-  getset(先get再set)  
*getset:将给定 key 的值设为 value ，并返回 key 的旧值(old value)*  
  

## LIST
  
Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素导列表的头部（左边）或者尾部（右边），它的底层实际是个链表  
  

```html
> rpush list-key item
(integer) 1
> rpush list-key item2
(integer) 2
> rpush list-key item
(integer) 3

> lrange list-key 0 -1
1) "item"
2) "item2"
3) "item"

> lindex list-key 1
"item2"

> lpop list-key
"item"

> lrange list-key 0 -1
1) "item2"
2) "item"
```
  
**案例**  
 
-  lpush/rpush/lrange  

-  lpop/rpop  

-  lindex，按照索引下标获得元素(从上到下)
*通过索引获取列表中的元素 lindex key index*  

-  llen  
*setex:设置带过期时间的 key，动态设置；setex 键秒值 真实值；setnx:只有在 key 不存在时设置 key 的值*  

-  lrem key 删N个value  
*从left往right删除2个值等于v1的元素，返回的值为实际删除的数量；LREM list3 0 值，表示删除全部给定的值。零个就是全部值*  

-  ltrim key 开始index 结束index，截取指定范围的值后再赋值给key  
*ltrim：截取指定索引区间的元素，格式是ltrim list的key 起始索引结束索引*  

-  rpoplpush 源列表 目的列表  
*移除列表的最后一个元素，并将该元素添加到另一个列表并返回  

-  lset key index value  

-  linsert key  before/after 值1 值2  
*在list某个已有值的前后再添加具体值*  
   

**性能小结**  
它是一个字符串链表，left、right都可以插入添加；如果键不存在，创建新的链表；如果键已存在，新增内容；如果值全移除，对应的键也就消失了。  
  

链表的操作无论是头和尾效率都极高，但假如是对中间元素进行操作，效率就很惨淡了。  
  
  
## SET
  
Redis的Set是String类型的无序集合，它是通过HashTable实现实现的。  


```html
> sadd set-key item
(integer) 1
> sadd set-key item2
(integer) 1
> sadd set-key item3
(integer) 1
> sadd set-key item
(integer) 0

> smembers set-key
1) "item"
2) "item2"
3) "item3"

> sismember set-key item4
(integer) 0
> sismember set-key item
(integer) 1

> srem set-key item2
(integer) 1
> srem set-key item2
(integer) 0

> smembers set-key
1) "item"
2) "item3"
```
  
**案例**  
 
-  sadd/smembers/sismember   

-  scard，获取集合里面的元素个数  

-  srem key value 删除集合中元素  
*通过索引获取列表中的元素 lindex key index*  

-  srandmember key 某个整数(随机出几个数)  
*从set集合里面随机取出2个；如果超过最大数量就全部取出，如果写的值是负数，比如-3 ，表示需要取出3个，但是可能会有重复值*  

-  spop key 随机出栈  

-  smove key1 key2 在key1里某个值，作用是将key1里的某个值赋给key2  

   
**数学集合类**  

- 差集：sdiff
- 交集：sinter
- 并集：sunion

  
## HASH
  
Redis hash 是一个键值对集合。  
Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象，类似Java里面的Map<String,Object>。  

```html
> hset hash-key sub-key1 value1
(integer) 1
> hset hash-key sub-key2 value2
(integer) 1
> hset hash-key sub-key1 value1
(integer) 0

> hgetall hash-key
1) "sub-key1"
2) "value1"
3) "sub-key2"
4) "value2"

> hdel hash-key sub-key2
(integer) 1
> hdel hash-key sub-key2
(integer) 0

> hget hash-key sub-key1
"value1"

> hgetall hash-key
1) "sub-key1"
2) "value1"
```
  
**案例**  
 
-  hset/hget/hmset/hmget/hgetall/hdel  
-  hlen  
-  hexists key
-  hkeys/hvals 
-  hincrby/hincrbyfloat  
-  hsetnx  
   

## ZSET
  
Sorted Set：有序集合  
Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。zset的成员是唯一的，但分数(score)却可以重复。    

```html
> zadd zset-key 728 member1
(integer) 1
> zadd zset-key 982 member0
(integer) 1
> zadd zset-key 982 member0
(integer) 0

> zrange zset-key 0 -1 withscores
1) "member1"
2) "728"
3) "member0"
4) "982"

> zrangebyscore zset-key 0 800 withscores
1) "member1"
2) "728"

> zrem zset-key member1
(integer) 1
> zrem zset-key member1
(integer) 0

> zrange zset-key 0 -1 withscores
1) "member0"
2) "982"
```
   
  
**案例**  
 
-  zadd/zrange   

-  zrangebyscore key 开始score 结束score  

-  zrem key 某score下对应的value值，作用是删除元素  
*删除元素，格式是zrem zset的key 项的值，项的值可以是多个；zrem key score某个对应值，可以是多个值*  

-  zcard/zcount key score区间/zrank key values值，作用是获得下标值/zscore key 对应值，获得分数  
*zcard ：获取集合中元素个数*  
*zcount ：获取分数区间内元素个数，zcount key 开始分数区间 结束分数区间*  
*zrank： 获取value在zset中的下标位置*  
*zscore：按照值获得对应的分数*  
  
-  zrevrank key values值，作用是正序、逆序获得下标索引值  

-  zrevrange  

-  zrevrangebyscore  key 结束score 开始score  
*zrevrangebyscore zset1 90 60 withscores 分数是反着来的*
  
**补充说明**  
在set基础上，加一个score值  
之前set是k1 v1 v2 v3，现在zset是k1 score1 v1 score2 v2  

## 补充 健（Key）
  
*非五大数据类型*  
  
**案例**  
 
-  keys *
-  exists key的名字，判断某个key是否存在
-  move key db   --->当前库就被移除
-  expire key 秒钟：为给定的key设置过期时间
-  ttl key 查看还有多少秒过期，-1表示永不过期，-2表示已过期
-  type key 查看你的key是什么类型
  

---

# 四、数据结构

## 字典

dictht 是一个散列表结构，使用拉链法保存哈希冲突。

```c
/* This is our hash table structure. Every dictionary has two of this as we
 * implement incremental rehashing, for the old to the new table. */
typedef struct dictht {
    dictEntry **table;
    unsigned long size;
    unsigned long sizemask;
    unsigned long used;
} dictht;
```

```c
typedef struct dictEntry {
    void *key;
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;
} dictEntry;
```

Redis 的字典 dict 中包含两个哈希表 dictht，这是为了方便进行 rehash 操作。在扩容时，将其中一个 dictht 上的键值对 rehash 到另一个 dictht 上面，完成之后释放空间并交换两个 dictht 的角色。

```c
typedef struct dict {
    dictType *type;
    void *privdata;
    dictht ht[2];
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    unsigned long iterators; /* number of iterators currently running */
} dict;
```

rehash 操作不是一次性完成，而是采用渐进方式，这是为了避免一次性执行过多的 rehash 操作给服务器带来过大的负担。

渐进式 rehash 通过记录 dict 的 rehashidx 完成，它从 0 开始，然后每执行一次 rehash 都会递增。例如在一次 rehash 中，要把 dict[0] rehash 到 dict[1]，这一次会把 dict[0] 上 table[rehashidx] 的键值对 rehash 到 dict[1] 上，dict[0] 的 table[rehashidx] 指向 null，并令 rehashidx++。

在 rehash 期间，每次对字典执行添加、删除、查找或者更新操作时，都会执行一次渐进式 rehash。

采用渐进式 rehash 会导致字典中的数据分散在两个 dictht 上，因此对字典的查找操作也需要到对应的 dictht 去执行。

```c
/* Performs N steps of incremental rehashing. Returns 1 if there are still
 * keys to move from the old to the new hash table, otherwise 0 is returned.
 *
 * Note that a rehashing step consists in moving a bucket (that may have more
 * than one key as we use chaining) from the old to the new hash table, however
 * since part of the hash table may be composed of empty spaces, it is not
 * guaranteed that this function will rehash even a single bucket, since it
 * will visit at max N*10 empty buckets in total, otherwise the amount of
 * work it does would be unbound and the function may block for a long time. */
int dictRehash(dict *d, int n) {
    int empty_visits = n * 10; /* Max number of empty buckets to visit. */
    if (!dictIsRehashing(d)) return 0;

    while (n-- && d->ht[0].used != 0) {
        dictEntry *de, *nextde;

        /* Note that rehashidx can't overflow as we are sure there are more
         * elements because ht[0].used != 0 */
        assert(d->ht[0].size > (unsigned long) d->rehashidx);
        while (d->ht[0].table[d->rehashidx] == NULL) {
            d->rehashidx++;
            if (--empty_visits == 0) return 1;
        }
        de = d->ht[0].table[d->rehashidx];
        /* Move all the keys in this bucket from the old to the new hash HT */
        while (de) {
            uint64_t h;

            nextde = de->next;
            /* Get the index in the new hash table */
            h = dictHashKey(d, de->key) & d->ht[1].sizemask;
            de->next = d->ht[1].table[h];
            d->ht[1].table[h] = de;
            d->ht[0].used--;
            d->ht[1].used++;
            de = nextde;
        }
        d->ht[0].table[d->rehashidx] = NULL;
        d->rehashidx++;
    }

    /* Check if we already rehashed the whole table... */
    if (d->ht[0].used == 0) {
        zfree(d->ht[0].table);
        d->ht[0] = d->ht[1];
        _dictReset(&d->ht[1]);
        d->rehashidx = -1;
        return 0;
    }

    /* More to rehash... */
    return 1;
}
```

## 跳跃表

是有序集合的底层实现之一。

跳跃表是基于多指针有序链表实现的，可以看成多个有序链表。

与红黑树等平衡树相比，跳跃表具有以下优点：

- 插入速度非常快速，因为不需要进行旋转等操作来维护平衡性；
- 更容易实现；
- 支持无锁操作。


---

# 五、基础应用  

## 计数器

可以对 String 进行自增自减运算，从而实现计数器功能。

Redis 这种内存型数据库的读写性能非常高，很适合存储频繁读写的计数量。

## 缓存

将热点数据放到内存中，设置内存的最大使用量以及淘汰策略来保证缓存的命中率。

## 查找表

例如 DNS 记录就很适合使用 Redis 进行存储。

查找表和缓存类似，也是利用了 Redis 快速的查找特性。但是查找表的内容不能失效，而缓存的内容可以失效，因为缓存不作为可靠的数据来源。

## 消息队列

List 是一个双向链表，可以通过 lpop 和 lpush 写入和读取消息。

不过最好使用 Kafka、RabbitMQ 等消息中间件。

## 会话缓存

可以使用 Redis 来统一存储多台应用服务器的会话信息。

当应用服务器不再存储用户的会话信息，也就不再具有状态，一个用户可以请求任意一个应用服务器，从而更容易实现高可用性以及可伸缩性。

## 分布式锁实现

在分布式场景下，无法使用单机环境下的锁来对多个节点上的进程进行同步。

可以使用 Reids 自带的 SETNX 命令实现分布式锁，除此之外，还可以使用官方提供的 RedLock 分布式锁实现。

## 发布订阅

先订阅后发布后才能收到消息  
1 可以一次性订阅多个，SUBSCRIBE c1 c2 c3  

2 消息发布，PUBLISH c2 hello-redis

3 订阅多个，通配符*， PSUBSCRIBE new*
4 收取消息， PUBLISH new1 redisRec

## 其它

Set 可以实现交集、并集等操作，从而实现共同好友等功能。

ZSet 可以实现有序性操作，从而实现排行榜等功能。
  

---

# 六、Redis 与 Memcached

两者都是非关系型内存键值数据库，主要有以下不同：

## 数据类型

Memcached 仅支持字符串类型，而 Redis 支持五种不同的数据类型，可以更灵活地解决问题。

## 数据持久化

Redis 支持两种持久化策略：RDB 快照和 AOF 日志，而 Memcached 不支持持久化。

## 分布式

Memcached 不支持分布式，只能通过在客户端使用一致性哈希来实现分布式存储，这种方式在存储和查询时都需要先在客户端计算一次数据所在的节点。

Redis Cluster 实现了分布式的支持。

## 内存管理机制

- 在 Redis 中，并不是所有数据都一直存储在内存中，可以将一些很久没用的 value 交换到磁盘，而 Memcached 的数据则会一直在内存中。

- Memcached 将内存分割成特定长度的块来存储数据，以完全解决内存碎片的问题。但是这种方式会使得内存的利用率不高，例如块的大小为 128 bytes，只存储 100 bytes 的数据，那么剩下的 28 bytes 就浪费掉了。
  

---

# 七、Redis事务

一个事务包含了多个命令，服务器在执行事务期间，不会改去执行其它客户端的命令请求。

换句话说，可以一次执行多个命令，本质是一组命令的集合。一个事务中的所有命令都会序列化，按顺序地串行化执行而不会被其它命令插入，不许加塞。

**一个队列中，一次性、顺序性、排他性的执行一系列命令。**

事务中的多个命令被一次性发送给服务器，而不是一条一条发送，这种方式被称为流水线，它可以减少客户端与服务器之间的网络通信次数从而提升性能。

Redis 最简单的事务实现方式是使用 MULTI 和 EXEC 命令将事务操作包围起来。

## 常用命令  
  
**discard**:取消事务，放弃事务块中的所有命令  
  
**exec**:执行事务块中的所有命令  
  
**multi**:标记一个事务的开启  
  
**unwatch**:取消watch命令对所有key的监控  
  
**watch key [key ....]**:监控一个或多个key,如果事务执行之前，key被其他命令所改动，那么事务断事务不会执行  

## 常见场景
  

- 正常执行  
  
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/normal.png)  

- 放弃事务  
     
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/discard.png)  

- 全体连坐  
  
  
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/failAll.png)  

只要有 一个命令有语法错误，执行EXEC命令后Redis就会直接返回错误，连语法正确的命令也不会执行
  
- 冤头债主  
  
  
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/failPart.png)  

运行错误，运行错误指在命令执行时出现的错误，比如使用散列类型的命令操作集合类型的键，这种错误在实际执行之前Redis是无法发现的，所以在事务里这样的命令是会被Redis接受并执行的。如果事务里的 一条命令出现了运行错误，事务里其他的命令依然会继续执行（包括出错命令之后的命令）
  
- watch监控 
  

 **悲观锁(Pessimistic Lock)**：每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。

 **乐观锁(Optimistic Lock)**：每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，样可以提高吞吐量。
  
例子：  

初始化信用卡可用余额和欠额，无加塞篡改，先监控再开启multi，保证两笔金额变动在同一个事务内  
  
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/51.png)  
  
有加塞篡改，监控了key，如果key被修改了，后面一个事务的执行失效 
  
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/52.png)    
  
Unwatch 命令用于取消 WATCH 命令对所有 key 的监视。unwatch一定要在multi之前，在multi和exec之间unwatch是无效的,因为multi之后事务已经开启  
 
**小结**：Watch指令，类似乐观锁，事务提交时，如果Key的值已被别的客户端改变，比如某个list已被别的客户端push/pop过了，整个事务队列都不会被执行。
通过WATCH命令在事务执行之前监控了多个Keys，倘若在WATCH之后有任何Key的值发生了变化，EXEC命令执行的事务都将被放弃。

## 一般阶段  
  
- 开启：以MULTI开始一个事务
- 入队：将多个命令入队到事务中，接到这些命令并不会立即执行，而是放到等待执行的事务队列里面
- 执行：由EXEC命令触发事务
  
## 三大特性  

- 单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断
- 没有隔离级别的概念：队列中的命令没有提交之前都不会实际的被执行，因为事务提交前任何指令都不会被实际执行，也就不存在“事务内的查询要看到事务里的更新，在事务外查询不能看到”这个让人万分头痛的问题
- 不保证原子性：redis同一个事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚
  
## 事件

Redis 服务器是一个事件驱动程序。
  
  

---

# 八、持久化

Redis 是内存型数据库，为了保证数据在断电后不会丢失，需要将内存中的数据持久化到硬盘上。

## RDB 持久化

在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里。  
  
Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方
式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。  
  
**Fork** ：  
fork的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等）数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程  
  
**RDB 保存的是dump.rdb文件**   
  
**如何触发RDB快照**  
①达到配置文件中的条件，配置文件中默认的快照配置为:  
 save <seconds> <changes>  
 save 900 1  15分钟内有一次修改  
 save 120 10 5分钟内有一次修改  
 save 60 10000   1分钟内有一万次修改  
 如果配置成 save "" 表示不进行rdb持久化  

②命令save或者是bgsave  
Save：save时只管保存，其它不管，全部阻塞。  
BGSAVE：Redis会在后台异步进行快照操作，快照同时还可以响应客户端请求。  
执行flushall命令，也会产生dump.rdb文件，但里面是空的，无意义。  

**如何恢复**  
将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可  
CONFIG GET dir获取目录  
 

**优点**  
对数据完整性和一致性要求不高，适合大规模的数据恢复    
  
**缺点**  
在一定间隔时间做一次备份，所以如果redis意外down掉的话，最后一次持久化后的数据会丢失  
  
**如何停止**  
动态所有停止RDB保存规则的方法：redis-cli config set save ""    
  

## AOF 持久化

以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来(读操作不记录)，只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。Aof保存的是appendonly.aof文件。  

使用 AOF 持久化需要设置同步选项，从而确保写命令什么时候会同步到磁盘文件上。这是因为对文件进行写入并不会马上将内容同步到磁盘上，而是先存储到缓冲区，然后由操作系统决定什么时候同步到磁盘。有以下同步选项：

| 选项 | 同步频率 |
| :--: | :--: |
| always | 每个写命令都同步 |
| everysec | 每秒同步一次 |
| no | 让操作系统来决定何时同步 |

- always 选项会严重减低服务器的性能；
- everysec 选项比较合适，可以保证系统崩溃时只会丢失一秒左右的数据，并且 Redis 每秒执行一次同步对服务器性能几乎没有任何影响；
- no 选项并不能给服务器性能带来多大的提升，而且也会增加系统崩溃时数据丢失的数量。  
- 修复：redis-check-aof --fix进行修复
- 恢复：重启redis然后重新加载
- appendonly:默认为no 可改为yes启动AOF
- appendfilename:AOF文件的默认名

随着服务器写请求的增多，AOF 文件会越来越大。Redis 提供了一种将 AOF **重写**的特性，能够去除 AOF 文件中的冗余写命令。  
## 重写 ##
AOF采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制,当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集.可以使用命令bgrewriteaof。  

**原理**：AOF文件持续增长而过大时，会fork出一条新进程来将文件重写(也是先写临时文件最后再rename)，遍历新进程的内存中数据，每条记录有一条的Set语句。重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似。  
  
**触发机制**：Redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发  
  
**优点**
AOF文件保存的数据集要比RDB文件保存的数据集要完整  
  
**缺点**  
相同数据集的数据而言aof文件要远大于rdb文件，恢复速度慢于rdb，aof运行效率要慢于rdb,每秒同步策略效率较好，不同步效率和rdb相同  
  
**小结**  

- RDB持久化方式能够在指定的时间间隔能对你的数据进行快照存储   
  
- AOF持久化方式记录每次对服务器写的操作,当服务器重启的时候会重新执行这些命令来恢复原始的数据,AOF命令以redis协议追加保存每次写的操作到文件末尾。Redis还能对AOF文件进行后台重写,使得AOF文件的体积不至于过大  
  
- 只做缓存：如果你只希望你的数据在服务器运行的时候存在,你也可以不使用任何持久化方式  
  
- 同时开启两种持久化方式：在这种情况下,当redis重启的时候会优先载入AOF文件来恢复原始的数据，因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整  
  
- RDB的数据不实时，同时使用两者时服务器重启也只会找AOF文件。那要不要只使用AOF呢？本人建议不要，因为RDB更适合用于备份数据库(AOF在不断变化不好备份)，快速重启，而且不会有AOF可能潜在的bug，留着作为一个万一的手段  

---

# 九、复制
  
## 概念
通过使用 slaveof host port 命令来让一个服务器成为另一个服务器的从服务器。

一个从服务器只能有一个主服务器，并且不支持主主复制。  
  
**行话**：也就是我们所说的主从复制，主机数据更新后根据配置和策略，自动同步到备机的master/slaver机制，Master以写为主，Slave以读为主。  
  
## 作用效果
  
- 读写分离
- 容灾恢复  

## 连接过程

1. 主服务器创建快照文件，发送给从服务器，并在发送期间使用缓冲区记录执行的写命令。快照文件发送完毕之后，开始向从服务器发送存储在缓冲区中的写命令；

2. 从服务器丢弃所有旧数据，载入主服务器发来的快照文件，之后从服务器开始接受主服务器发来的写命令；

3. 主服务器每执行一次写命令，就向从服务器发送相同的写命令。
  
## 操作技巧  

- 配从(库)不配主(库)  

- 从库配置：slaveof 主库IP 主库端口  
（1）每次与master断开之后，都需要重新连接，除非你配置进redis.conf文件  
（2）info replication 

- 修改配置文件细节操作  
（1）拷贝多个redis.conf文件  
（2）开启daemonize yes  
（3）pid文件名字  
（4）指定端口  
（5）log文件名字  
（6）dump.rdb名字  

![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/ms1.png)  
   
- 常用三大招  
  
  
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/ms2.png)  
    
（1）**一主二仆**  
   
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/ms3.png)  
      
（2）**薪火相传**  
  
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/ms4.png)  
    
（3）**反客为主**  
  
  
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/ms5.png)  
    
  
## 复制原理  
  
- slave启动成功连接到master后会发送一个sync命令
- master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，master将传送整个数据文件到slave,以完成一次完全同步
- 全量复制：slave服务在接收到数据库文件数据后，将其存盘并加载到内存中
- 增量复制：master继续将新的所有收集到的修改命令依次传给slave,完成同步
- 只要是重新连接master,一次完全同步（全量复制)将被自动执行  
  
## 复制缺点  
  
延时  
    
## 哨兵模式  
    
Sentinel（哨兵）可以监听集群中的服务器，并在主服务器进入下线状态时，自动从从服务器中选举出新的主服务器。  
  
说白了，就是反客为主的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库。   
  
**一组sentinel能同时监控多个Master。**
   
![预览图](https://github.com/Zhangxuan-Xing/Review/blob/master/Picture/ms7.png)
   

---

# 十、键的过期时间

Redis 可以为每个键设置过期时间，当键过期时，会自动删除该键。

对于散列表这种容器，只能为整个键设置过期时间（整个散列表），而不能为键里面的单个元素设置过期时间。
  

---
  
# 十一、数据淘汰策略

可以设置内存最大使用量，当内存使用量超出时，会施行数据淘汰策略。

Reids 具体有 6 种淘汰策略：

| 策略 | 描述 |
| :--: | :--: |
| volatile-lru | 从已设置过期时间的数据集中挑选最近最少使用的数据淘汰 |
| volatile-ttl | 从已设置过期时间的数据集中挑选将要过期的数据淘汰 |
|volatile-random | 从已设置过期时间的数据集中任意选择数据淘汰 |
| allkeys-lru | 从所有数据集中挑选最近最少使用的数据淘汰 |
| allkeys-random | 从所有数据集中任意选择数据进行淘汰 |
| noeviction | 禁止驱逐数据 |

作为内存数据库，出于对性能和内存消耗的考虑，Redis 的淘汰算法实际实现上并非针对所有 key，而是抽样一小部分并且从中选出被淘汰的 key。

使用 Redis 缓存数据时，为了提高缓存命中率，需要保证缓存数据都是热点数据。可以将内存最大使用量设置为热点数据占用的内存量，然后启用 allkeys-lru 淘汰策略，将最近最少使用的数据淘汰。

Redis 4.0 引入了 volatile-lfu 和 allkeys-lfu 淘汰策略，LFU 策略通过统计访问频率，将访问频率最少的键值对淘汰。
  

---
  
# 十二、事件

Redis 服务器是一个事件驱动程序。

## 文件事件

服务器通过套接字与客户端或者其它服务器进行通信，文件事件就是对套接字操作的抽象。

Redis 基于 Reactor 模式开发了自己的网络事件处理器，使用 I/O 多路复用程序来同时监听多个套接字，并将到达的事件传送给文件事件分派器，分派器会根据套接字产生的事件类型调用相应的事件处理器。

## 时间事件

服务器有一些操作需要在给定的时间点执行，时间事件是对这类定时操作的抽象。

时间事件又分为：

- 定时事件：是让一段程序在指定的时间之内执行一次；
- 周期性事件：是让一段程序每隔指定时间就执行一次。

Redis 将所有时间事件都放在一个无序链表中，通过遍历整个链表查找出已到达的时间事件，并调用相应的事件处理器。

## 事件的调度与执行

服务器需要不断监听文件事件的套接字才能得到待处理的文件事件，但是不能一直监听，否则时间事件无法在规定的时间内执行，因此监听时间应该根据距离现在最近的时间事件来决定。

事件调度与执行由 aeProcessEvents 函数负责，伪代码如下：

```python
def aeProcessEvents():
    # 获取到达时间离当前时间最接近的时间事件
    time_event = aeSearchNearestTimer()
    # 计算最接近的时间事件距离到达还有多少毫秒
    remaind_ms = time_event.when - unix_ts_now()
    # 如果事件已到达，那么 remaind_ms 的值可能为负数，将它设为 0
    if remaind_ms < 0:
        remaind_ms = 0
    # 根据 remaind_ms 的值，创建 timeval
    timeval = create_timeval_with_ms(remaind_ms)
    # 阻塞并等待文件事件产生，最大阻塞时间由传入的 timeval 决定
    aeApiPoll(timeval)
    # 处理所有已产生的文件事件
    procesFileEvents()
    # 处理所有已到达的时间事件
    processTimeEvents()
```

将 aeProcessEvents 函数置于一个循环里面，加上初始化和清理函数，就构成了 Redis 服务器的主函数，伪代码如下：

```python
def main():
    # 初始化服务器
    init_server()
    # 一直处理事件，直到服务器关闭为止
    while server_is_not_shutdown():
        aeProcessEvents()
    # 服务器关闭，执行清理操作
    clean_server()
```



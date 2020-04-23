# Redis

## 1.关系型数据库CAP原理 

### CAP

C：Consistency （强一致性）

A：Availability（可用性）

P：Partition tolerance（分区容错性）

<span style="color:red">理论核心：</span>一个分布式系统不可能同时很好地满足一致性，可用性和分区容错性这三个需求，最多只能同时较好的满足两个。



三原则

+ CA： 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。（传统数据库）
+ CP： 满足一致性，分区容忍性的系统，通常性能不是特别高。 （Redis、Mongodb等）
+ AP：  满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。 （大多数网站架构）

### CAP 的三进二

CAP理论就是说在分布式存储系统中，最多只能实现上面的两点。

而由于当前的网络硬件肯定会出现延迟丢包等问题，所以 **分区容错性** 是我们必须需要实现的。

所以我们只能在一致性和可用性间进行权衡。



### BASE

BASE就是为了解决关系数据库强一致性引起的问题而引起的可用性降低而提出的解决方案。

基本可用 Basically Available

软状态 Soft state

最终一致 Eventually  consistent



思想：让系统放松对某一时刻数据一致性的要求来换取系统整体伸缩性和性能上改观。原因在于，大型系统往往由于地域分布和极高性能的要求，不可能采用分布式事务来完成这些指标，所以采用BASE。



## 2. 分布式和集群简介

### 分布式

不同的多台服务器上面部署不同的服务模块（工程）， 他们之间通过RPC/RMI之间通信和调用，对外提供服务和组内协作。

### 集群

不同的多台服务器上面部署相同的服务模块，通过分布式调度软件进行统一的调度，对外提供服务和访问



## 3. Redis入门

### 是什么

**RE**mote **DI**ctionary **S**erver（远程字典服务器）

高性能（Key/Value）的内存数据库，基于内存运行。

C语言编写的开源数据库

### 特点

+ 支持数据持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用
+ 不仅支持简单的key-value类型的数据，同时还提供list、set、zset、hash等数据结构的存储
+ 支持数据的备份，即主从模式的数据备份
+ 内部采用单线程机制进行工作
+ 数据间没有必然的关联关系
+ 高性能。



### 应用

+ 为热点数据加速查询，如热点商品、热点新闻、热点资讯、推广等高访问量信息。
+ 任务队列，如秒杀、抢购、购票排队等
+ 即时信息查询，如排行榜、各类网站访问统计、公交到站信息、在线人数信息、设备信号等
+ 时效性信息控制，如验证码控制、股票控制等
+ 分布式数据共享，如分布式集群架构中的session分离
+ 消息队列
+ 分布式锁

### 基本操作

#### 添加

> set key value

#### 取值

> get key

如果没有这个key，返回 null



## 4. Redis数据类型

### String

set，get，

mset（一次设置多个key-val）， mget（一次获取多个数据）

> mset key1 value1 key2 value2...
>
> mget key1 key2

strlen(获取数据字符个数：字符串长度)

> strlen key

append(追加str信息到原始数据尾部)

> append key str  //追加str字符串到key对应value的尾部

incr (增加1)

> incr key  //对key对应的value值加1

decr (减少1)

> decr key //对key对应的value值减少1

incrby （增加指定步长的值）

> incrby key len  //对key对应的value值增加 len

decrby （增加指定步长的值）

> incrby key len  //对key对应的value值减少 len

getrange（截取字符）

> getrange key1 L R  //截取key1对应的value [L,R]区间的字符串
>
> getrange key 0 -1  //获取全部字符串

 setrange（替换字符）

> setrange key x str  /将key对应的value，从x位置开始的字符串，替换为str

setex (set with expire) 设置过期时间

setnx (set if not exist)  不存在再设置  ，在分布式锁中会常常使用

>setex key second value //对key设置过期时间second(秒)，如果key存在，那么将会用value复写旧值。
>
>setnx key value  //当key不存在时，将value值赋给key



#对象

> set user:1 {name:wjb , age:1}  //传入json串，这里的key就是 user:1
>
> mset user:1:name wjb user:1:age 1 //set多值，这里的key就是 user:1:name 和 user:1:age



getset (先获取在设置)

> getset key value //如果key不存在返回null，如果存在则先获取原值，并设置新的值



**单数据操作与多数据操作的选择：**

打个比方：

set指令发送到返回结果有三个过程： set发送，redis处理，结果返回。 三个过程都有耗时。

单指令和多指令难以权衡，因为发送返回处理三个过程的耗时取决于你发送的数据。



### List

在redis内， list可以实现栈、队列、阻塞队列等。

```bash
#######################################
127.0.0.1:6379> lpush list one  //从左插入
(integer) 1
127.0.0.1:6379> lpush list two
(integer) 2
127.0.0.1:6379> lpush list three
(integer) 3
127.0.0.1:6379> lrange list 0 -1 //获取所有list值
1) "three"
2) "two"
3) "one"
127.0.0.1:6379> lrange list 0 1 //获取区间list值
1) "three"
2) "two"
127.0.0.1:6379> rpush list rone //从右插入
(integer) 4
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "two"
3) "one"
4) "rone"
###########################################
pop移除

127.0.0.1:6379> lpop list //从左移除
"three"
127.0.0.1:6379> rpop list //从右移除
"rone"
127.0.0.1:6379> lrange list 0 -1
1) "two"
2) "one"
###########################################
lindex //获取指定下标值

127.0.0.1:6379> lindex list 1 
"one"
127.0.0.1:6379> lindex list 0
"two"
###########################################
lset //设置指定下标的值，需要key存在

lset key index value
###########################################
llen //获取list长度
127.0.0.1:6379> llen list
(integer) 2
###########################################
lrem //移除 指定个数的值
lrem key count value
127.0.0.1:6379> lrem list 1 one
(integer) 1
127.0.0.1:6379> lrange list 0 -1
1) "two"

###########################################
ltrim //截断
ltrim list start,end ## list只剩下截取的值
###########################################
rpoplpush #移除列表最后一个元素，将它移动到新的列表中，若新列表不存在则创建
rpoplpush source destination

127.0.0.1:6379> lpush list one
(integer) 1
127.0.0.1:6379> lpush list two
(integer) 2
127.0.0.1:6379> lpush list three
(integer) 3
127.0.0.1:6379> rpoplpush list mylist
"one"
127.0.0.1:6379> lrange mylist 0 -1
1) "one"
###########################################
linsert #将某个具体的value插入道列表中某个元素的前面或后面

linsert key BEFORE|AFTER pivot value #pivot 插入的值
```



> 小结
>
> + 实际上是一个链表，left，right 都可以插入值
>
> + 如果key不存在，创建新的链表
> + 如果key存在，新增内容
> + 如果移除了所有值，空链表，也代表不存在
> + 在两边插入或者改动值，效率最高。 插入到中间元素，相对来说效率会低一些。
>
> 消息队列



### Set

set中的值是唯一的，无重复

```bash
127.0.0.1:6379> sadd set "hello" #插入
(integer) 1
127.0.0.1:6379> sadd set "wjb"
(integer) 1
127.0.0.1:6379> smembers set  #查看元素
1) "wjb"
2) "hello"
127.0.0.1:6379> sismember set hello  #判断值是否存在
(integer) 1
127.0.0.1:6379> sismember set hh
(integer) 0
127.0.0.1:6379> scard set  #查看元素个数
(integer) 2
127.0.0.1:6379> srem set hello  #移除元素
(integer) 1
127.0.0.1:6379> sadd set "A"
(integer) 1
127.0.0.1:6379> sadd set "B"
(integer) 1
127.0.0.1:6379> srandmember set  #随机取元素
"B"
127.0.0.1:6379> srandmember set
"A"
127.0.0.1:6379> smembers set
1) "wjb"
2) "A"
3) "B"
127.0.0.1:6379> spop set   #随机移除元素
"B"
127.0.0.1:6379> flushdb
OK
127.0.0.1:6379> sadd set1 "A"
(integer) 1
127.0.0.1:6379> sadd set1 "b"
(integer) 1
127.0.0.1:6379> sadd set1 "c"
(integer) 1
127.0.0.1:6379> sadd set2 "set2"
(integer) 1
127.0.0.1:6379> smove set1 set2 "c" #将set1中的元素移动到set2中
(integer) 1
127.0.0.1:6379> smembers set1
1) "b"
2) "A"
127.0.0.1:6379> smembers set2
1) "set2"
2) "c"

############################################

#微博等共同关注
127.0.0.1:6379> sadd set1 a
(integer) 1
127.0.0.1:6379> sadd set1 b
(integer) 1
127.0.0.1:6379> sadd set1 c
(integer) 1
127.0.0.1:6379> sadd set2 a
(integer) 1
127.0.0.1:6379>  sadd set2 d
(integer) 1
127.0.0.1:6379>  sadd set2 e
(integer) 1
127.0.0.1:6379> sduff set1 set2 
(error) ERR unknown command 'sduff'
127.0.0.1:6379> sdiff set1 set2   #差集
1) "b"
2) "c"
127.0.0.1:6379> sinter set1 set2  #交集
1) "a"
127.0.0.1:6379> sunion set1 set2  #并集
1) "b"
2) "a"
3) "c"
4) "e"
5) "d"

#A用户将所有关注的人放在一个集合，将他的粉丝也放在一个集合中
#可以实现 共同关注，共同爱好，二度好友，推荐好友
```



### Hash

Map 集合，存key-value

Redis中使用Hash，相当于Redis存了key-value，这个value存了个Map集合，又是一个key-value



```bash
127.0.0.1:6379> hset hash k1 wjb  #插入
(integer) 1
127.0.0.1:6379> hget hash k1  #获取
"wjb"
127.0.0.1:6379> hmset hash k2 a k3 b k4 c #插入多个
OK
127.0.0.1:6379> hmget hash k1 k2 k3 k4  #获取多个
1) "wjb"
2) "a"
3) "b"
4) "c"
127.0.0.1:6379> hgetall hash  #获取全部
1) "k1"
2) "wjb"
3) "k2"
4) "a"
5) "k3"
6) "b"
7) "k4"
8) "c"
127.0.0.1:6379> hdel hash k1  #删除key，value值也消失
(integer) 1
127.0.0.1:6379> hlen hash  #获取hash字段数量
(integer) 3
127.0.0.1:6379> hexists hash k1  #查看key是否存在
(integer) 0
127.0.0.1:6379> hexists hash k2
(integer) 1
127.0.0.1:6379> hkeys hash #获取所有键
1) "k2"
2) "k3"
3) "k4"
127.0.0.1:6379> hvals hash #获取所有值
1) "a"
2) "b"
3) "c"
127.0.0.1:6379> hset hash k5 5
(integer) 1
127.0.0.1:6379> hincrby hash k5 2 #增加
(integer) 7
```



hash变更的数据，尤其是用户信息之类的经常变动的信息，更适合对象的存储。

类似于hash< user,< name:xx>>



### Zset

有序集合

在set的基础上，增加了一个值 score 表示排序关键字

`zset k1 score1 v1`

```bash
127.0.0.1:6379> zadd zset 1 one  #添加
(integer) 1
127.0.0.1:6379> zadd zset 3 two 2 three
(integer) 2
127.0.0.1:6379> zrange zset 0 -1
1) "one"
2) "three"
3) "two"
127.0.0.1:6379> zadd zset 200 A 100 B 300 C
(integer) 3
127.0.0.1:6379> zrange zset 0 -1
1) "one"
2) "three"
3) "two"
4) "B"
5) "A"
6) "C"
127.0.0.1:6379> zrangebyscore zset -inf inf  #从小到大排序 score从负无穷到正无穷的范围
1) "one"
2) "three"
3) "two"
4) "B"
5) "A"
6) "C"
127.0.0.1:6379> zrangebyscore zset -inf 100  #从小到大排序 对score小于100的值排序
1) "one"
2) "three"
3) "two"
4) "B"
127.0.0.1:6379> zrevrangebyscore zset 100 -inf #从大到小排序 对score小于100的值降序排序
1) "B"
2) "two"
3) "three"
4) "one"
127.0.0.1:6379> zcount zset 100 300  #获取指定区间成员数量
(integer) 3
```

案例： set排序 存储班级成绩表，工资表排序等。

普通消息：1.重要消息 2.带权重消息 

排行榜应用，取Top N



## 5. Redis三种特殊数据类型

### geospatial 地理位置

朋友的位置，附近的人 ，打车距离计算

相关命令：

```bash
# geoadd 添加位置
# 规则：两级无法直接添加。
# 参数：  key 经度 纬度 名称
127.0.0.1:6379> geoadd china:city 116.40 39.90 beijing
(integer) 1
127.0.0.1:6379> geoadd china:city 121.47 31.23 shanghai
(integer) 1
127.0.0.1:6379> geoadd china:city 106.50 29.53 chongqing 114.05 22.52 shenzhen
(integer) 2
127.0.0.1:6379> geoadd china:city 120.16 30.24 hangzhou 108.96 34.26 xian
(integer) 2
```

```bash
# geopos 获取指定的城市的经度和纬度
127.0.0.1:6379> geopos china:city beijing
1) 1) "116.39999896287918"
   2) "39.900000091670925"
```

```bash
# geodist 两地间距离
127.0.0.1:6379> geodist china:city shanghai beijing
"1067378.7564"
127.0.0.1:6379> geodist china:city shanghai beijing km
"1067.3788"

单位：
m 米
km 千米
mi 英里
ft 英尺
```

```bash
# georadius 以给定的经纬度为中心，找出某一半径内的元素
127.0.0.1:6379> georadius china:city 110 30 500 km
1) "chongqing"
2) "xian"

127.0.0.1:6379> georadius china:city 110 30 500 km withcoord  #获取半径内元素的经纬度
1) 1) "chongqing"
   2) 1) "106.49999767541885"
      2) "29.529999579006592"
2) 1) "xian"
   2) 1) "108.96000176668167"
      2) "34.2599996441893"

127.0.0.1:6379> georadius china:city 110 30 500 km withdist  #获取半径内的元素到中心的距离
1) 1) "chongqing"
   2) "341.9374"
2) 1) "xian"
   2) "483.8340"
   
127.0.0.1:6379> GEORADIUSBYMEMBER china:city beijing 1000 km #获取到指定value半径内的元素
1) "beijing"
2) "xian"

127.0.0.1:6379> geohash china:city beijing chongqing  #获取value的hash值，将二维的经纬度转换为一维的hash字符串
1) "wx4fbxxfke0"
2) "wm5xzrybty0"

#可以通过比较hash值来判断两个地点是否在一定范围内
```

> GEO的底层实现原理就是 zset！我们可以使用zset命令来操作geo

附近的人？如何实现？

获得所有附近的人的地址，插入一个集合，通过半径来查询。





### Hyperloglog 基础统计

> 什么是基数？

A{1,3,5,7,8,7}

B{1,3,5,7,8}

基数（不重复的元素个数）= 5， 可以接受误差。



Redis Hyperloglog 基数统计的算法

有点：占用的内存是固定的， 2^64不同的元素，只需要消耗12kb内存。

**网页的UV （一个人访问一个网站多次，但是还是算作一个人）**

传统的方式使用set保存用户id，然后统计set的size

这个方式如果保存大量的用户id，就会比较麻烦。

我们的目的是为了计数，而不是为了保存用户id



```bash
127.0.0.1:6379> PFadd mykey a b c d e f g h i j #创建第一组元素
(integer) 1
127.0.0.1:6379> PFcount mykey  #统计基数
(integer) 10
127.0.0.1:6379> PFadd mykey2 i j z x c v b n m #创建第二组元素
(integer) 1
127.0.0.1:6379> PFcount mykey2
(integer) 9
127.0.0.1:6379> PFmerge mykey3 mykey mykey2  #合并两组，求并集
OK
127.0.0.1:6379> PFcount mykey3
(integer) 15
```



### Bitmaps

> 位存储

统计用户信息，活跃/不活跃。 登录/未登录 等只有两个状态的都可以使用Bitmaps。

Bitmaps 位图，是一种数据结构。都是操作二进制位来进行记录，就只有0和1两个状态。

```bash
# 使用bitmaps 来记录周一到周日的打卡。
127.0.0.1:6379> setbit sign 0 0
(integer) 0
127.0.0.1:6379> setbit sign 1 0
(integer) 0
127.0.0.1:6379> setbit sign 2 1
(integer) 0
127.0.0.1:6379> setbit sign 3 0
(integer) 0
127.0.0.1:6379> setbit sign 4 1
(integer) 0
127.0.0.1:6379> setbit sign 5 0
(integer) 0
127.0.0.1:6379> setbit sign 6 0
(integer) 0

#查看某一天是否打卡
127.0.0.1:6379> getbit sign 6
(integer) 0
127.0.0.1:6379> getbit sign 4
(integer) 1

#统计打卡天数（1的个数）
127.0.0.1:6379> bitcount sign
(integer) 2
```



## 6. Redis的事务

### 事务

ACID，<span style="color:red">Redis单条命令保证原子性，但是事务不保证原子性！</span>

Redis事务本质：一组命令的集合。一个事务中的所有命令都会被序列化，在事务执行过程中，会按照顺序执行。

一次性、顺序性、排他性。



<span style="color:red">Redis事务没有隔离级别的概念。</span>

所有的命令在事务中，并没有直接被执行，只有发起执行命令的时候才会执行。Exec



Redis的事务：

+ 开启事务（Multi）
+ 命令入队（....）
+ 执行事务（exec）

```bash
127.0.0.1:6379> multi  #开启事务
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> exec  #执行事务，显示结果
1) OK
2) OK
3) "v2"
4) OK 

#取消事务
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> set k4 v4
QUEUED
127.0.0.1:6379> discard  #取消事务，事务队列中的命令都不会被执行
OK
127.0.0.1:6379> get k4
(nil)
```

> 异常
>
> 编译型异常（代码有问题，命令有错），事务中所有的命令都不会被执行。
>
> Redis事务中，如果命令队列中出现编译型异常，那么事务执行exec也会报错。

> 运行时异常（如1除0），如果事务队列中存在语法性错误，那么执行命令时，其他命令可以正常执行。



## 7. Redis锁

### 悲观锁

很悲观，认为什么时候都会出问题，所以无论做什么都会加锁

### 乐观锁

很乐观，认为什么时候都不会出问题，所以不会上锁。更新数据时判断，在此期间是否有人修改过这个数据的版本号。

+ 获取version
+ 更新时比较version

> Redis 监视测试

```bash
127.0.0.1:6379> set money 100
OK
127.0.0.1:6379> set out 0
OK
127.0.0.1:6379> watch money  #监视money对象
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> decrby money 20
QUEUED
127.0.0.1:6379> incrby out 20
QUEUED
127.0.0.1:6379> exec
1) (integer) 80
2) (integer) 20

#正常执行
```



测试多线程修改值，使用watch可以当做redis的乐观锁操作。



```bash
127.0.0.1:6379> watch money
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> decrby money 10
QUEUED
127.0.0.1:6379> incrby out 10
QUEUED
127.0.0.1:6379> exec #执行前，另外一个线程修改了值，导致事务执行失败
(nil)

#监视失败
```

```bash
127.0.0.1:6379> unwatch # 当事务执行失败时，先解锁
OK
127.0.0.1:6379> watch money  #重新监视，获取最新值
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> decrby money 10
QUEUED
127.0.0.1:6379> incrby out 10
QUEUED
127.0.0.1:6379> exec  #重新执行
1) (integer) 990
2) (integer) 30
```



> 小结： Redis 的 Watch命令，其实就是乐观锁
>
> 因此Redis可以实现乐观锁



## 8. Jedis

我们使用 java来操作Redis

> 什么是Jedis？ Jedis是Redis官方推荐的java连接开发工具。使用java操作Redis中间件。如果你要使用java操作redis，需要对Jedis十分的熟悉。



### 操作

#### 导入依赖

```xml
<!--    jedis-->
        <!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.2.0</version>
        </dependency>

        <!--  fastjson      -->
        <!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.68</version>
        </dependency>
```



#### 连接redis数据库

```java
    public static void main(String []args){
        // 1. 连接服务器
        Jedis jedis = new Jedis("127.0.0.1",6379);

        // 2. jedis的所有命令就是之前学的set get 等等
        System.out.println(jedis.ping()); //连接成功输出 PONG
    }
```



#### 具体API

与4 5 节内容相同，不再赘述。

![Jedis](E:\myyyyyyyyyyyyyynote\Redis\Jedis.PNG)



#### 操作事务

```java
public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1",6379);

        JSONObject jsonObject = new JSONObject();
        jsonObject.put("hello","world");
        jsonObject.put("name","wjb");
        //开启事务
        Transaction multi = jedis.multi();
        String result = jsonObject.toJSONString();
        try{
            multi.set("user1", result);
            multi.set("user2",result);
            //int x = 1/0;
            multi.exec();  //执行事务
        }catch (Exception e){
            multi.discard();  //放弃事务
            e.printStackTrace();
        }finally {
            System.out.println(jedis.get("user1"));
            System.out.println(jedis.get("user2"));


            jedis.close();
        }

    }
```



## 9. Springboot整合Redis

Springboot操作数据： spring-data



说明：在Springboot2.x之后，原来使用的jedis被替换为了lettuce

jedis：采用的直连，多个线程操作的话，是不安全的，如果想要避免不安全的，使用jedis pool连接池，BIO。

lettuce：采用netty， 实例可以在多个线程中进行共享，不存在线程不安全的情况。可以减少线程数据，NIO。



> 源码分析

```java
@Bean 
@ConditionalOnMissingBean(name = {"redisTemplate"}) //可以自定义redisTemplate来替换默认的
public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
    //默认的RedisTemplate没有过多的配置， redis对象都需要序列化
    //两个泛型都是Object，后续使用需要强制转化<String,Object>
    RedisTemplate<Object, Object> template = new RedisTemplate();
    template.setConnectionFactory(redisConnectionFactory);
    return template;
}

@Bean
@ConditionalOnMissingBean //string是redis中最常用的数据类型，单独创建
public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
    StringRedisTemplate template = new StringRedisTemplate();
    template.setConnectionFactory(redisConnectionFactory);
    return template;
}
```



> 整合

1. 导入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
   </dependency>
   ```

2. 配置配置文件

```properties
#配置redis
spring.redis.host=127.0.0.1	
spring.redis.port=6379

```

3. API

   ```java
   /*
   redisTemplate  相当于jedis，用于操作指令
   opsForValue()  操作字符串
   opsForList()
   opsForSet()
   opsForHash()
   opsForZSet()
   opsForGeo()
   opsForHyperLogLog()
   */
   
   除了基本的操作，我们常用的方法都可以直接通过redisTemplate操作，比如事务，基本的CRUD
       
   /*
   获取redis的连接对象
   RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
           connection.flushDb();
           connection.flushAll();
   */
   ```

   

4. 自己编写RedisTemplate

   ```java
   @Configuration
   public class RedisConfig {
   
       @Bean
       public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
           RedisTemplate<String, Object> template = new RedisTemplate<>();
           template.setConnectionFactory(redisConnectionFactory);
           //json序列化配置
           Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
           ObjectMapper om = new ObjectMapper();
           om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
           om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
   
           //String序列化
           StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
   
           //配置具体的序列化方式
           //key设置为string序列化
           template.setKeySerializer(stringRedisSerializer);
           //Hash key设置为string序列化
           template.setHashKeySerializer(stringRedisSerializer);
           //value设置为jackson序列化
           template.setValueSerializer(jackson2JsonRedisSerializer);
           //Hash value设置为jackson序列化
           template.setHashValueSerializer(jackson2JsonRedisSerializer);
           template.afterPropertiesSet();
   
           return template;
       }
   }
   ```

   

## 10. Redis.conf 详解

> 单位

![RedisConf1](E:\myyyyyyyyyyyyyynote\Redis\RedisConf1.png)

redis可忽略 units的大小写



> 包含	

![RedisConf2](E:\myyyyyyyyyyyyyynote\Redis\RedisConf2.png)

可将多个配置文件包含进来。

 

> 网络

```bash
bind 127.0.0.1 # 绑定的ip
protected-mode yes  #保护模式 yes为开启 no为关闭
port 6379  #端口设置

```



> 通用 GENERAL

```bash
daemonize no #以守护进程方式运行，默认是no，需要手动开启yes
pidfile /var/run/redis.pid   #如果以后台方式运行，需要指定一个pid文件

#日志
# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably) 
# warning (only very important / critical messages are logged)
loglevel notice  #日志级别

logfile ""  #日志文件位置
databases 16  #数据库数量，默认16个

```



> 快照 SNAPSHOTTING

持久化，在规定的时间内，执行了多少次操作，则会持久化到文件 .rdb   .aof

redis 是内存数据库，如果没有持久化，那么数据断电即失。



```bash
# 如果900s内，有至少1个key进行了修改，我们就进行持久化操作
save 900 1
# 如果300s内，有至少10个key进行了修改，我们就进行持久化操作
save 300 10
# 如果60s内，有至少10000个key进行了修改，我们就进行持久化操作
save 60 10000

stop-writes-on-bgsave-error yes  # 持久化如果出错，是否还继续工作
rdbcompression yes    # 是否压缩rdb文件，压缩需要消耗一些cpu资源
rdbchecksum yes       # 保存rdb文件时，进行错误的检查校验
dir ./                # rdb文件的保存目录
```



> REPLICATION 主从复制



> 安全 SECURITY

```bash
requirepass  xxx  #设置密码， 默认为空， 也可通过命令行设置
```



> 限制 CLIENTS

```BASH
maxclients 10000   #设置能连接上redis的最大客户端数量
maxmemory <bytes>  #redis配置最大内存容量
maxmemory-policy noeviction   #内存到达上限之后的处理策略
    1、volatile-lru：只对设置了过期时间的key进行LRU（默认值） 
    2、allkeys-lru ： 删除lru算法的key   
    3、volatile-random：随机删除即将过期key   
    4、allkeys-random：随机删除  
    5、volatile-ttl ： 删除即将过期的   
    6、noeviction ： 永不过期，返回错误

```



> APPEND ONLY MODE  aof配置

```bash
appendonly no  #默认不开启aof模式，默认使用rdb方式持久化，在大部分情况下，rdb完全够用。 #持久化的文件
appendfilename "appendonly.aof"   #持久化的文件名 

# appendfsync always    #每次修改都会 sync， 消耗性能
appendfsync everysec    #每秒执行一次 sync， 可能会丢失这1s的数据
# appendfsync no        #不执行 sync， 这时操作系统自己同步数据，数据最快。
```



具体的配置，在redis持久化中再说。



## 11. Redis持久化（重点）

Redis 是内存数据库，如果不将内存中的数据库状态保存到磁盘，那么一旦服务器进程退出，服务器中的数据库状态也会消失。 所以 Redis 提供了持久化功能。



### RDB

> 什么是RDB? redis database

![RDB保存过程](E:\myyyyyyyyyyyyyynote\Redis\RDB保存过程.PNG)

在指定的时间间隔内，将内存中的数据集快照写入磁盘，它恢复时是将快照文件直接读到内存里。

Redis会单独创建（fork）一个子进程来进行持久化，会将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的。这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加地高效。RDB的缺点是最后一次持久化后的数据可能丢失（服务器宕机）。

redis中默认的就是RDB，一般情况下不需要修改这个配置。

**rdb保存的文件是 dump.rdb**



> 触发 redis 创建dump.rdb文件的机制

1、配置文件中，满足save的规则

2、执行flushall命令

3、退出redis

备份就自动生成



> 如何恢复rdb文件

只需要将rdb文件放在配置文件中设置的rdb文件目录，redis启动时会自动检查dump.rdb，恢复其中的数据。



#### 优点

+ 适合大规模的数据恢复
+ 对数据的完整性要求不高（因为可能服务器宕机导致小部分数据丢失）

#### 缺点

+ 需要一定的时间间隔进行操作。如果redis意外宕机，最后一次修改的数据就可能丢失。
+ fork进程时会消耗一定的内存空间



### AOF

> 什么是AOF？append only file

将我们的所有<span style="color:red">写命令</span>都记录下来，相当于一个history文件，恢复时就把这个文件中所有<span style="color:red">写命令</span>全部再执行一遍。

![AOF](E:\myyyyyyyyyyyyyynote\Redis\AOF.PNG)



以日志的形式来记录每个**写操作**，将Redis执行过的所有指令记录下来（读操作不记录）， 只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据。换言之， redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。



AOF保存的文件是**appendonly.aof**

只需将配置文件中的`appendonly no`改为yes，然后重启redis就可以使用AOF了



> 如果AOF出错怎么办

服务器可能在程序正在对 AOF 文件进行写入时停机， 如果停机造成了 AOF 文件出错（corrupt）， 那么 Redis 在重启时会拒绝载入这个 AOF 文件， 从而确保数据的一致性不会被破坏。

当发生这种情况时， 可以用以下方法来修复出错的 AOF 文件：

1. 为现有的 AOF 文件创建一个备份。
2. 使用 Redis 附带的 `redis-check-aof` 程序，对原来的 AOF 文件进行修复。

> ```
> $ redis-check-aof --fix
> ```

3. （可选）使用 `diff -u` 对比修复后的 AOF 文件和原始 AOF 文件的备份，查看两个文件之间的不同之处。
4.  重启 Redis 服务器，等待服务器载入修复后的 AOF 文件，并进行数据恢复。



#### 优点

```bash
# appendfsync always    #每次修改都会 sync， 消耗性能
appendfsync everysec    #每秒执行一次 sync， 可能会丢失这1s的数据
# appendfsync no        #不执行 sync， 这时操作系统自己同步数据，数据最快。
```

#### 缺点

+ 相对于数据文件来说，aof远远大于rdb，修复的速度也比rdb慢
+ aof的运行效率也要比rdb慢，所以redis默认的持久化配置是使用rdb





## 12. Redis发布订阅

略

使用场景：

1、实时消息系统。

2、实时聊天（聊天室）

3、订阅、关注功能

## 13. Redis主从复制

### 概念

主从复制，是指将一台redis服务器的数据，复制到其他的redis服务器。前者称为主节点（master/leader），后者称为从节点（slave/follower）；**数据的复制是单向的，只能由主节点到从节点**。 Master以写为主，Slave以读为主。



### 作用

1、数据备份：主从复制实现了数据的热备份，是持久化之外的一种数据备份方式。

2、故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复。实际上是一种服务的备份。

3、负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务，分担服务器负载。尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。

4、高可用基石：主从复制还是哨兵机制和集群能够实施的基础，因此说主从复制是redis高可用的基础。



**主从复制，读写分离。80%的情况下都是在进行读操作，减缓服务器的压力，架构中经常使用，最低配置1主2从。**



### 环境配置

只需配置从库，不用配置主库。

```bash
127.0.0.1:6379> info replication   #查看当前库信息
# Replication
role:master
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```



复制三个redis.conf，分别修改端口号、pidfile、logfile、dump.rdb名称，开启daemon

然后启动

![redis主从复制1](E:\myyyyyyyyyyyyyynote\Redis\redis主从复制1.PNG)



### 一主二从

<span style="color:red">默认情况下，每个redis节点都是主节点。</span>一般情况下我们只需要配置从节点。



一主（端口79）二从（端口80,81）

```bash
127.0.0.1:6380> SLAVEOF 127.0.0.1 6379  #将79端口设为主机
OK
127.0.0.1:6380> info replication
# Replication
role:slave      #当前角色为 从
master_host:127.0.0.1
master_port:6379
。。。。。。
```

81端口也是一样

```bash
127.0.0.1:6379> info replication  #查看主机信息
# Replication
role:master
connected_slaves:2   #两个从机
slave0:ip=127.0.0.1,port=6380,state=online,offset=196,lag=0   #从机信息
slave1:ip=127.0.0.1,port=6381,state=online,offset=196,lag=1

```



**命令行配置主从关系只是暂时的，实际上需要在配置文件中配置才是永久的。**



> 注意

主机可以执行写操作，从机不能写只能读。

```bash
127.0.0.1:6379> set k1 v1  #主机写
OK
 
#从机尝试读
127.0.0.1:6380> get k1
"v1"   #结果正常

#从机尝试写
127.0.0.1:6380> set k2 v2
(error) READONLY You can't write against a read only replica.  #报错


```





> 测试主机宕机

主机宕机，从机依旧连接到主机，但是没有写操作。此时主机回归，从机依旧可以直接获取到主机写的信息。

实际上，主机断开，需要在剩下的从机中选择一个当作主机，避免从机等待消耗资源。



> 测试从机宕机

从机宕机，如果主从关系不是永久的（即不是在配置文件中配置主从关系），那么这台从机重新连接后就会变成主机，那么就获取不到原主机写操作的值。如果对这台从机和原主机重新配置主从关系，那么从机还是能够获得原主机的所有数据。



> 复制原理

Slave启动成功连接到master后会发送一个sync同步命令

Master接到命令后，启动后台的存盘进程，同时收集所有接收到的用于修改数据集的命令，在后台进程执行完毕之后，**master将传送整个数据文件到slave，并完成一次完全同步。**

**全量复制：**在slave接收到数据库文件数据后，将其存盘并加载到内存中

**增量复制：**master继续将新的所有收集到的修改命令一次传给slave，完成同步



> 层层链路

![层层链路](E:\myyyyyyyyyyyyyynote\Redis\层层链路.PNG)



此时80从机虽然是81从机的主节点，但本质上80从机还是从节点。



如果79主机断开，那么80从机就可以使用`slaveof no one`指令让自己变成主机，其他节点就可以收到连接到这个最新的主节点。 如果79重新连接，此时并不会影响其他节点的主从关系！



### 哨兵模式

（自动选举主机）

手动切换主从机过于繁琐，因此Redis2.8开始提供了Sentinel（哨兵）架构来解决这个问题。

哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行。

**原理是：哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例。**



![哨兵](E:\myyyyyyyyyyyyyynote\Redis\哨兵.PNG)

哨兵作用

1、通过发送命令，让Redis服务器返回其运行状态

2、当哨兵监测到master宕机，会自动将slave切换成master，然后通过**发布订阅模式**通知其他的从服务器，修改配置文件，让它们切换主机。

为了防止哨兵挂掉，可以设置多个哨兵，多哨兵间互相监控。



多哨兵模式下，如果主机宕机，哨兵1先检测到宕机，系统并不会马上执行主机切换（故障转移）操作，因为此时仅仅是哨兵1主观认为主机不可用（**主观下线**），只有当后面的哨兵也检测到主机不可用，并且数量达到一定值时，哨兵之间就会进行一次投票。 投票由一个哨兵发起，进行failover故障转移操作。 切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从服务器实现切换主机，这个过程称为**客观下线**。



> 测试

我的目前状态是一主二从。

1、配置哨兵配置文件 sentinel.conf

```bash
# sentinel monitor 被监控的名称 host port 1
sentinel monitor myredis 127.0.0.1 6379 1
```

后面的这个数字1，代表主机挂了，slave投票看让谁接替称为主机，票数最多的，就会成为主机。



2、启动哨兵

```bash
wayjasy@wayjasy-virtual-machine:~/redis-5.0.8/src$ redis-sentinel ../myconfig/sentinel.conf 
13576:X 22 Apr 2020 22:25:52.377 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
13576:X 22 Apr 2020 22:25:52.377 # Redis version=5.0.8, bits=64, commit=00000000, modified=0, pid=13576, just started
13576:X 22 Apr 2020 22:25:52.377 # Configuration loaded
13576:X 22 Apr 2020 22:25:52.378 * Increased maximum number of open files to 10032 (it was originally set to 1024).
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 5.0.8 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in sentinel mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 26379
 |    `-._   `._    /     _.-'    |     PID: 13576
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

13576:X 22 Apr 2020 22:25:52.456 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
13576:X 22 Apr 2020 22:25:52.465 # Sentinel ID is debaca00730683e6abc58b232e4de78f88086b03
13576:X 22 Apr 2020 22:25:52.466 # +monitor master myredis 127.0.0.1 6379 quorum 1
13576:X 22 Apr 2020 22:25:52.551 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ myredis 127.0.0.1 6379
13576:X 22 Apr 2020 22:25:52.552 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ myredis 127.0.0.1 6379

```

3、主机宕机时，系统failover（故障转移）

![failover](E:\myyyyyyyyyyyyyynote\Redis\failover.png)



> 哨兵模式优点

+ 哨兵集群，基于主从复制模式，所有的主从复制优点，它全有。
+ 主从可以切换，故障可以转移，系统的可用性就会更好。
+ 哨兵模式就是主从模式的升级，手动到自动，更加健壮。

> 哨兵模式缺点

+ Redis 在线扩容不易，集群容量一旦到达上限，在线扩容就十分麻烦。
+ 实现哨兵模式的配置其实是很麻烦的，里面就很多选择。



### 写在最后

需要提醒的是，以上的主从复制以及哨兵模式，都是基于linux虚拟机实现的伪主从复制。实际上，他们的配置是很复杂的！！



## 14. Redis缓存穿透和雪崩 （重点）

服务的高可用问题

> 概念

用户想要查询一个数据，发现redis内存数据库中没有，也就是缓存没有命中，于是像持久层数据库查询。发现也没有，于是本次查询失败。当用户很多的时候，缓存都没有命中（秒杀场景），于是都去请求持久层数据库，这会给持久层数据库造成很大的压力，这时候就出现了**缓存穿透**。



> 解决方案

**布隆过滤器**

布隆过滤器是一种数据结构，对**所有可能查询的参数**（即相当于把所有可用的、有效的key）以hash形式存储，在控制层先进行校验，不符合则丢弃，从而避免了对底层存储系统的查询压力。

布隆过滤器的巨大作用就是判断 **一个元素是否存在与集合中**。（可能判错，但绝不会漏判）因此，Bloom Filter不适合那些“零错误”的应用场合。

在低错误率的场景下，布隆过滤器相较于其它查找算法，如Hash表、折半查找等，极大地节约了内存空间



**缓存空对象**

当底层数据库中也没有用户查询的数据时，那么将这个数据作为**空对象**缓存起来，并设置一个过期时间，之后再访问这个数据将会从缓存中获取，保护了底层数据库。



但是也有两个问题：

+ 如果缓存空值，意味着缓存需要更多的空间存储更多的键，而这些键对应的value是空，毫无意义还浪费空间。
+ 即使对空值设置了过期时间，还是会存在缓存层和存储层的数据会有一段时间窗口的不一致，这会影响数据的一致性。



### 缓存击穿

> 概念

有一个非常热门的key，有非常非常多的请求不断访问这个key，巨大的并发量集中对这一个点进行访问。由于我们对缓存中的key设置了过期时间，当这个key失效的瞬间，持续的大并发量就会瞬间穿破缓存，直接请求数据库，造成服务器宕机。



> 解决方案

**设置热点数据永不过期（不推荐）**

从缓存层面来看，没有设置过期时间，所以一个热点key的缓存可以持续抵挡大并发量。

**加互斥锁**

使用分布式锁，保证对于每个key同时只有一个线程去查询数据库，其他查询线程只能等待。因此高并发压力转移到分布式锁上，对分布式锁的考验很大。



### 缓存雪崩

> 概念

是指，在某个时间段，缓存集中过期失效。 （Redis宕机）

产生缓存雪崩的原因之一，比如双十一零点，很快迎来一波抢购，这波热点商品比较集中的放入缓存，假设设置过期时间一小时。到了凌晨一点时，大面积商品的缓存过期了，然而对这批商品的访问查询全都落在了数据库上，对于数据库来说，就产生了周期性的压力波峰。于是所有请求都会达到存储层，存储层的调用量就会保证，可能导致存储层挂掉。



双十一期间：阿里会停掉一些服务，保证主要的服务正常运行。



> 解决方案

**redis高可用**

搭建redis集群， 异地多活。

**限流降级**

在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。

**数据预热**

项目正式部署之前，先把可能的数据访问一遍，这样部分可能大量访问的数据就会加载到缓存中。在即将发生大并发访问前，手动触发加载缓存不同的key，设置不同的过期时间，让缓存失效的时间点尽量均衡，不要集中在一个点。这个方法就相当于常见的活动预告。
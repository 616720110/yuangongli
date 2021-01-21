### **Redis 开发与运维**

 ### <center>第一章</center>
> Redis :

1. <u>支持的数据结构：</u>
>1. Redis中的值可以是由string（字符串）、hash（哈希）、 list（列表）、set（集合）、zset（有序集合）、Bitmaps（位图）、 HyperLogLog、GEO

2. <u>redis 特性：</u>
>1. 速度快：10万/秒，Mermory storage, C language implement,single thread prevent muilt thread lock(单线程防止了多线程的竞争问题)
>2. 基于键值对的数据结构服务：java 的Map,python 的dict 
>3. 丰富的功能， expired/Lua/transaction/piepline
>4. 客户端语言多/简单稳定
>5. 持久化： RDB/AOF
>6. 主从复制
>7. 高可用，2.8后提供sentinel 哨兵
3. <u>Redis安装后的相关shell</u>
>1. redis-server :启动Redis
>2. redis-cli：客户端连接
>3. redis-benchmark Redis基准测试工具
>4. redis-check-aof：AOF持久化文件检测和修复工具
>5. redis-check-dunmp: RDB持久化检测和修复工具
>6. redis-sentinel: 启动哨兵，高可用
4. <u> redis启动可以修改变量</u>
> 1. redis-server --configKey configValue --key2 value2 :example redis-server --prot 6380
> 2. 可启动配置，/etc/redis/redis.conf
> 3. 通知redis服务  客户端发送 redis-cli shutdown
5. <u>Redis 服务关闭</u>
> 1. redis-cli shutdown: 断开与客户端的连接，持久化文件生产。（优雅关闭方式）
> 2. kill 方式。不会持久化，如为AOF方式，也将可能造成数据丢失
> 3. <u>redis-cli shutdown nosave | save 可选择不持久化</u>
6. <u> Redi 版本规则</u> 第二位奇数 非稳定版本。 偶数，稳定版本
> 1. Redis3.0是重要的里程碑，发布了Redis官方的分布式实现Redis Cluster

### <center>第二章 API的理解和使用</center>
理解redis机制，学会命令的通用性

1. <u>全局命令</u>
> 1. 查看所有键： key *
> 2. 总键数：dbsize
```  
    dbsize 在计算总数时不会遍历所有键，而是直接获取Redis内置的键总数。时间复杂度O(1)
    而keys命令会遍历所有键。时间复杂度O（n）
```
>3. 检查键是否存在: exists key
>4. 删除键： del key
>5. 键过期： expire key seconds / ttl key 返回剩余时间
>6. 键的数据结构类型：type key
2. <u> 数据结构和内部编码</u>:对外的五种数据结构，内部都有自己的实现编码，而且是多种实现，可以通过object encoding 命令查询内部编码

![image](https://github.com/616720110/yuangongli/blob/master/redisPricture/1610523854(1).jpg)

```
内外编码分层优势：可以改进内部编码，而对的数据结构和命令没有影响
```
2. <u>单线程架构</u>
>1. redis接收命令放入队列中，然后逐个执行。
> > 1. 存内存访问，非阻塞I/O,使用epoll作为I/O多路复用技术的实现且Redis自身的事件处理模型将epoll中的连接，读写，关闭都转换为实践。不在I/O上浪费过多的时间

2.1. <u>命令</u>
<center>String</center>

1. 批量设置获取 mget /mset   (multi)批量设置可以减少网络传输
2. 设置新值返回旧值 setget key
3. 获取部分字符串 getrange key start end
4. 内部编码
> string内部编码有3种：
>> int：8个字节得长整型  
>> embstr: 小于等于39个字符的字符串  
>> raw: 大于39个字节的字符串  
Redis会根据当前值的类型和长度决定使用哪种内部编码实现。
<center>哈希 (hash,字典，关联数组） 其对应类型关系为 key-field-value</center>

1. 计算key中field个数：hlen key
2. hmget/hmset 批量设置和获取：hmset key field1 vaule1 field2 value2
3. 获取所有的field: hkeys key  (hkeys 命令叫hfeilds 更为恰当)
4. 获取所有的vaule: hvals key  <font color="red"> O(n) n为field总数 </font>
> 在使用hgetall时，如果哈希元素个数比较多，会存在阻塞Redis可能，如果只需要获取部分field,可以使用hmget,如果一定要使用获取全部filed-vaule,可以使用hscan命令，改命令会渐进式遍历哈希类型。  

5. 内部编码
>hash 内部编码有两种
>> ziplist（压缩列表）当hash类型元素个数小于hash-max-ziplist-entries配置（default 512），同时所有值小于hash-max-ziplist-vaule配置（default64字节）。使用ziplist,其为更加紧凑的结构实现多个元素的连续存储，在节省内存方面比hashtable更加优秀。  
>> hashtable(哈希表)：当哈希无法满足ziplist的条件时。大数据和大量数据ziplist的读写效率会下降，故使用hashtable,其时间杂度为O(1)

<Center>List 列表</center>  

1. 删除指定元素:lrem key count vaule   
2. 修改指定元素：lset key index newVaule
3. 阻塞操作：blpop/brpop:blpop lsit 3,列表为null，等待3秒后是否有值（如果指定多个list，会遍历key，直到发现一个key里面有值） 如果多个客户端同时执行brpop最先执行的获取到弹出的值
4. 按照索引范围修剪列表：ltrim
5. 内部编码  

> ziplist： list-max-ziplist-entries(512),list-max-ziplist-vaule(64字节)  

> linkedlist:

> quicklist：结合ziplist和linkedlist的有点 (https://matt.sh/redis-quicklist)

lpush+lpop = 栈  
lpush+rpop = 队列
lpush+ ltrim = 有限集合
lpush+brpop = 消息队列 
 
 <center>集合。 唯一，无序。不能通过index 搜检</center>

1. 删除元素：srem key
2. 计算元素个数： scard key  O(1),其不会遍历集合，而是取内部变量值
3. 判断是否为集合中：sismember key
4. 随机从集合中返回指定个元素：sranmember key count
5. 获取所有元素：smembers key  smembers/lrange/hgettall 都属于比较重的命令，如果元素过多存在阻塞的可能性，这时候可以使用sscan来完成。
6. 集合间操作  

 > 求多个集合的交集： sinter key1 key2  
 >求多个集合的并集：suinon key1 key2  
 > 求多个集合的差集：sdiff key1 key2
 > 由于交集，并集，差集比较耗时，同时提供了将集合的结果保存， s[commond]store destination key [set1 set2]  

 7. 内部编码  
 > inset:整数集合，当元素都是整数且个数小于 set-max-intset-entries（512个）时  
 > hashtable:哈希表，当集合类型无法满足intset时。  
 8. 应用 
 > sadd=Tagging （标签）
 > spop/srandmember = Random item (生产随机数，比如抽奖)
 > sadd + sinter = Social Graph (社交需求)

<center>有序集合 zset</center>




<center>单个键管理</center>
type/del/object/exists/expire

1. 键重命名 rename key new key
2. 随机返回一个键： randomkey
3. 键值过期： expire/pexpireat : 内部皆为pexpireat 毫毛实现。
> 如果过期时间为负值，建会立即被删除。如del  
> persist 命令可以将键的过期时间清除。  
> <u><font color="red">对于字符串类型，执行set命令会去掉过期时间。</font></u>
>> ![image](https://github.com/616720110/yuangongli/blob/master/redisPricture/1611196685(1).jpg)
4. redis 不支持二级数据结构的过期功能

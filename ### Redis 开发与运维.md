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

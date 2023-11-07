[TOC]

# Redis

## NoSQL
Not-Only SQL (泛指非关系型的数据库)，作为关系型数据库的补充。

作用：应对基于海量用户、海量数据和高并发的数据处理问题。

## Redis 简介
Redis，高性能键值对（key-value）数据库。

特征：

- 数据间没有必然的关联关系
- 单线程+多路IO复用
- 高性能
- 多数据类型支持
- 持久化支持，可以数据灾难恢复

应用：

- 为热点数据加速查询（主要场景），热点商品、热点新闻等
- 任务队列，如秒杀、抢购、购票排队等
- 即使信息查询，如各位排行榜、各类网站访问统计、公交到站信息等
- 时效性信息控制，如验证码控制等
- 分布式的数据共享
- 消息队列
- 分布式锁

## Redis 安装及启动
1. 下载压缩包到`/opts`目录下  
    [redis官方下载](https://redis.io/download)
    
    ```
    cd /opt
    wget http://download.redis.io/releases/redis-6.2.1.tar.gz
    ```
    
2. 安装 gcc
    ```
    yum install -y gcc-c++
    ```
    
3. 解压 redis
    ```
    tar -zxvf redis-6.2.1.tar.gz
    ```
    
5. 编译 redis、安装，安装到目录`/usr/local/bin`
    ```
    cd /opt/redis-6.2.1
    make
    make install
    ```
    
7. 准备配置文件到`/root/myredis`
    ```
    cp /opt/redis-6.2.1/redis.conf /root/myredis/
    ```
    
8. 指定配置文件启动服务
    ```
    redis-server /root/myredis/redis.conf
    ```
    
9. 配置后台运行服务
    ```
    vim /root/myredis/redis.conf
    # 修改 daemonize no 改为 daemonize yes
    ```
    
10. 停止服务
    ```
    redis-cli shutdown
    ```

## 命令
[Redis 命令参考](http://redisdoc.com/)  

[Redis 命令手册](https://www.redis.net.cn/order)  

## 配置文件
### 常见配置
- `bind` 绑定ip，仅能通过绑定的ip连接redis
- `protect-mode`只允许本机连接
- `port` redis端口
- `timeout` 断开连接时间，0则关闭功能
- `daemonized` 是否以守护进程启动
- `save` 在指定时间内有指定次数的操作则保存db
- `requirepass` 指定连接密码
- `pidfile`进程号保存文件
- `loglevel`日志级别
- `logfile`日志文件
- `databases`数据库数量

## 五大数据类型

![202209081831975](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071611691.jpg)

### string

字符串类型是`Redis`最基础的数据类型。首先，所有键都是字符串类型，而且其他集中数据结构都是在字符串类型的基础上构建的。

字符串类型的值实际可以是字符串、数字、二进制，但值最大不能超过512MB。

#### 数据结构

字符串数据结构为简单动态字符串，是可以修改的字符串。采用预分配冗余空间的方式，减少内存频繁分配。

扩容都是加倍现有的空间，如果超过1M，扩容时一次只会多扩1M的空间。

- `int`：8个字节的长整型
- `embstr`：小于等于39个字节的字符串
- `raw`：大于39个字节的字符串



### list

用来存储多个有序的字符串， 一个列表最多可以存储2<sup>32</sup>-1个元素。可以充当栈和队列的角色。

特点：有序、可重复

#### 数据结构

- `ziplist` - 元素较少的情况下会使用一块连续的内存存储
- `quicklist` - 以`ziplist`为节点的`linkedlist`



### hash

哈希类型是指键值本身又是一个键值对结构

#### 数据结构

- `ziplist` - 当哈希类型元素个数小于hash-max-ziplist-entries配置（默认512个） 、 同时所有值都小于hash-max-ziplist-value配置（默认64字节） 时， Redis会使用ziplist作为哈希的内部实现， ziplist使用更加紧凑的结构实现多个元素的连续存储
- `hashtable` - 当哈希类型无法满足ziplist的条件时，Redis会使用hashtable作为哈希的内部实现  



### set

用来保存**无序**、**不重复**的多个字符串，最多可以存储2<sup>32</sup>-1个元素。支持多个集合取交集、 并集、 差集。

#### 数据结构

- `intset` - 当集合中的元素都是整数且元素个数小于set-maxintset-entries配置（默认512个）Redis会选用intset来作为集合的内部实现， 从而减少内存的使用
- `hashtable` - 当集合类型无法满足intset的条件时， Redis会使用`hashtable`作为集合的内部实现  



### zset

保留了集合不能有重复成员的特性，同时元素可以排序。

![image-20220908192044417](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071611471.png)

#### 数据结构

- `ziplist` - 当有序集合的元素个数小于zset-max-ziplistentries配置（默认128个） ， 同时每个元素的值都小于zset-max-ziplist-value配置（默认64字节） 时， Redis会用ziplist来作为有序集合的内部实现  
- `skiplist`-当ziplist条件不满足时， 有序集合会使用skiplist作为内部实现  



## 新数据类型

## 发布订阅

## 事务

## 持久化

### RDB

### AOF

## 主从复制

## 集群

## 缓存

### 缓存穿透

`key`对应的数据在数据源中并不存在，每次针对这个`key`的请求从缓存中获取不到，请求都会压到数据源，从而可能压垮数据源。

#### 解决方案

- 对空值缓存：如果一个查询返回数据为空，仍然把这个空结果进行缓存，设置的过期时间较短。存在问题：如果`key`是随机的，也没什么用
- 采用布隆过滤器：布隆过滤器可以用于检索一个元素是否在一个集合中（有一定误判率）。一个一定不存在的数据会被拦截，避免了对底层系统的查询压力。

### 缓存击穿

请求的`key`对应的是热点数据，存在于数据库中，但不存在于缓存中（通常是因为过期了），导致大量请求直接打在数据库上，对数据库造成了巨大压力。

#### 解决方案：

- 预先设置热门数据：在高峰访问前将其存入缓存并设置合理的过期时间，比如保证在秒杀结束前不过期
- 请求数据库写入缓存前，先获取互斥锁，保证只有一个请求会落到数据库上，减少数据库压力。

### 缓存雪崩

当某一个时刻出现大规模的缓存失效的情况，请求直接压倒数据源上，导致数据库压力骤增。

#### 解决方案

- 在原有的失效时间基础上增加随机值，比如1-5分钟，这样每一个缓存的过期时间的重复率就会降低
- 采用Redis集群，避免单机宕机导致压力来到数据库

## 分布式锁

# 参考资料

[B站尚硅谷大学 - Redis6](https://www.bilibili.com/video/BV1Rv41177Af)

[《Redis开发与运维》](https://m.douban.com/book/subject/26971561/)


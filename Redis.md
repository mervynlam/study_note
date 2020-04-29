[TOC]

# Redis

<!--开始学习时间20200324-->

## NoSQL
Not-Only SQL (泛指非关系型的数据库)，作为关系型数据库的补充。

作用：应对基于海量用户、海量数据和高并发的数据处理问题。

特征：

- 可扩容、可伸缩
- 大数据量下高性能
- 灵活的数据类型
- 高可用

常见NoSQL数据库：

- Redis
- memcache
- HBase
- MongoDB

## Redis 简介
Redis(REmote DIctionary Server)，高性能键值对（key-value）数据库。

特征：

- 数据间没有必然的关联关系
- 内部采用单线程机制
- 高性能
- 多数据类型支持
    + 字符串 `string` -> `Map<String, String>`
    + 列表 `list` -> `Map<String, List<Object>>`
    + 散列 `hash` -> `Map<String, Map<Object, Object>>`
    + 集合 `set` -> `Map<String, Set<Object>>`
    + 有序集合 `sorted_set`
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
1. 下载压缩包到`/usr/local`目录下  
    [redis官方下载](https://redis.io/download)
    ```
    cd /usr/local
    wget http://download.redis.io/releases/redis-5.0.8.tar.gz
    ```
2. 安装 gcc
    ```
    yum install -y gcc-c++
    ```
3. 解压 redis
    ```
    tar -zxvf redis-5.0.8.tar.gz
    ```
4. 编译依赖
    ```
    cd /usr/local/redis-5.0.8/deps
    make hiredis jemalloc linenoise lua
    ```
5. 编译 redis
    ```
    cd /usr/local/redis-5.0.8
    make
    ```
6. 安装到`/usr/local/redis`目录
    ```
    mkdir /usr/local/redis
    cd /usr/local/redis-5.0.8
    make install PREFIX=/usr/local/redis
    ```
7. 准备配置文件到`/root/myredis`
    ```
    cd /usr/local/redis-5.0.8
    cp redis.conf /root/myredis/
    ```
8. 指定配置文件启动服务
    ```
    cd /usr/local/redis/bin
    ./redis-server /root/myredis/redis.conf
    ```
9. 配置后台运行服务
    ```
    vim /root/myredis/redis.conf
    # 修改 daemonize no 改为 daemonize yes
    ```
10. 停止服务
    ```
    ./redis-cli shutdown
    ```

## 命令
[Redis 命令参考](http://redisdoc.com/)  
[Redis 命令手册](https://www.redis.net.cn/order)  

## 配置文件
### 常见配置
- `bind` 绑定ip，仅绑定的ip可以连接redis
- `port` redis端口
- `timeout` 断开连接时间，0则关闭功能
- `daemoniz` 是否以守护进程启动
- `save` 在指定时间内有指定次数的操作则保存db
- `requirepass` 指定连接密码
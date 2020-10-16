# Centos 7 安装 Redis

## 安装

1. 安装c编译环境

   ```bash
   yum -y install gcc gcc-c++
   ```

2. 下载`redis`压缩包

   [redis版本列表](http://download.redis.io/releases/)选择版本

   ```bash
   wget http://download.redis.io/releases/redis-3.2.10.tar.gz
   ```

3. 解压

   ```bash
   tar -zxvf redis-3.2.10.tar.gz
   mv redis-3.2.10 /usr/local/redis
   ```

4. 编译、安装

   ```bash
   cd /usr/local/redis
   make
   cd src
   make PREFIX=/usr/local/redis install
   ```

## 配置

修改`redis.conf`文件

**后台运行**

```
daemonize yes
```

**远程访问**

```conf
bind 0.0.0.0
```

## 启动

```bash
./bin/redis-server ./redis.conf
```




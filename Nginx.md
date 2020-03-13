# Nginx
## Nginx优势及特征
- 负载均衡
- 反向代理
- 动静分离
- 高可用

默认监听80端口，反向代理到各个tomcat服务器，实现负载均衡，高并发

## 安装
1. 下载 nginx 压缩包到`/usr/local`目录  
    [nginx官网下载](http://nginx.org/en/download.html)  
    ```
    cd /usr/local
    wget http://nginx.org/download/nginx-1.17.9.tar.gz
    ```
2. 安装 nginx 所需环境
    1. 安装 gcc  
        安装 nginx 需要先将官网下载的源码进行编译，编译依赖 gcc 环境。  
        ```
        yum install -y gcc-c++
        ```
    2. 安装 pcre  
        PCRE (Perl Compatible Regular Expressions) 是一个 Perl 库，包括 perl 兼容的正则表达式库。Nginx 的 http 模块使用 pcre 来解析正则表达式。  
        pcre-devel 是使用 pcre 开发的一个二次开发库。Nginx 也需要此库。
        ```
        yum install -y pcre pcre-devel
        ```
    3. 安装 zlib  
        zlib 库提供了很多压缩和解压缩算法，在 Nginx 的各种模块中需要使用 gzip 压缩。
        ```
        yum install -y zlib zlib-devel
        ```
    4. 安装 open ssl  
        nginx 不仅支持 http 协议，还支持 https （即在 ssl 协议上传输 http），如果使用了 https，需要安装 OpenSSL 库。
        ```
        yum install -y openssl openssl-devel
        ```
3. 解压 nginx 并使用默认配置
    ```
    tar -zxvf nginx-1.17.9.tar.gz
    cd nginx-1.17.9
    ./configure
    ```
4. 编译安装 nginx
    ```
    make
    make install
    ```

## 启动、停止 nginx  
```
#进入 nginx 目录
cd /usr/local/nginx
```
```
./nginx #启动
./nginx -s stop
./nginx -s quit #停止
./nginx -s reload #重启
ps aux|grep nginx #查看 nginx 进程
```
## 开机启动
```
vim /etc/rc.local
```

添加一句

```
/usr/local/nginx/sbin/nginx
```

## 修改配置
```
cd /usr/local/nginx/conf
vim nginx.conf
```

Nginx 配置文件主要部分：  

- `main` 部分设置的指令影响其他所有部分的设置  
- `server` 部分的指令主要用于制定虚拟主机域名、IP 和端口号  
    ```
    server {
        # 监听端口
        listen 80;

        # 域名可以有多个，用空格隔开
        server_name mervyn.com www.mervyn.com;

        # 入口文件
        index index.jsp index.html;
        root /usr/local/nginx/html
    }
    ```
- `upstream` 的指令用于设置一系列的后端服务器，设置反向代理及后端服务器的负载均衡  
    ```
    upstream web{
        # 负载均衡，weight为权重，权重越高被分配到的几率越高
        server 192.168.177.135:8080 weight=3;
        server 192.168.177.135:8081 weight=1;

        # max_fails：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream模块定义的错误
        # fail_timeout:max_fails次失败后，暂停的时间。
    }
    # 在需要使用负载均衡的server中增加
    # proxy_pass http://web/;
    ```

    ```
    # nginx的upstream目前支持4种方式的分配
    # 1、轮询（默认）
    # 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
    # 2、weight
    # 指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
    # 例如：
    # upstream bakend {
    #     server 192.168.0.14 weight=10;
    #     server 192.168.0.15 weight=10;
    # }
    # 2、ip_hash
    # 每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。
    # 例如：
    # upstream bakend {
    #     ip_hash;
    #     server 192.168.0.14:88;
    #     server 192.168.0.15:80;
    # }
    # 3、fair（第三方）
    # 按后端服务器的响应时间来分配请求，响应时间短的优先分配。
    # upstream backend {
    #     server server1;
    #     server server2;
    #     fair;
    # }
    # 4、url_hash（第三方）
    # 按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。
    # 例：在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法
    # upstream backend {
    #     server squid1:3128;
    #     server squid2:3128;
    #     hash $request_uri;
    #     hash_method crc32;
    # }
    ```
- `location` 部分用于匹配网页位置（比如，根目录“/”，“/images”，等等）
    ```
    # / 为路径，比如访问 www.mervyn.com 即进入 / 路径， www.mervyn.com/project 即进入 /project 路径
    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;

        #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # 负载均衡
        proxy_pass http://web/
    }

    # 静态文件分离
    location ~ .*\.(html|htm|jpg|png|bmp|js|css)$ {
        # 静态文件目录
        root /usr/local/static;
        # 缓存时间
        expires 30d;
    }
    ```

keepalived


## 参考资料
[Linux下nginx的安装以及环境配置 - 凉凉的西瓜](https://blog.csdn.net/qq_42815754/article/details/82980326)  
[nginx.conf 配置文件详解 - 掘金](https://juejin.im/post/5c1616186fb9a049a42ef21d)  
[反向代理为何叫反向代理 - 知乎](https://www.zhihu.com/question/24723688)  
[Nginx负载与动静分离实战 - 腾讯课堂](https://ke.qq.com/course/482713?taid=4269592629697945)  
[【Nginx】教学视频【雷哥】 - 腾讯课堂](https://ke.qq.com/course/469240?taid=4862306706467064)
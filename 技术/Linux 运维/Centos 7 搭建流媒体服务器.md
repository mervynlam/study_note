**下载nginx-rtmp-module模块**

```bash
wget https://github.com/arut/nginx-rtmp-module/archive/master.zip
unzip -o master.zip
```

**安装nginx**

```bash
yum install -y gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel
wget http://nginx.org/download/nginx-1.19.6.tar.gz
tar -zxvf nginx-1.19.6.tar.gz -C /usr/local/
cd /usr/local/nginx-1.19.6
# 配置nginx增加上面下载的模块
./configure --prefix=/usr/local/nginx --add-module=/root/nginx-rtmp-module
# 编译、安装
make
make install 
```

**修改nginx配置**

```conf
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

events {
    worker_connections  1024;
}

# rtmp配置
rtmp{
    server{
        listen 1935;
        application myapp{
            live on;
        }
        application hls{
            live on;
            hls on;
            hls_path /tmp/hls;
        }
    }
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }
    
        location /hls {
            # 跨域
            add_header Access-Control-Allow-Origin *;
            types{
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }
            root /tmp;
            add_header Cache-Control no-cache;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}

```

**安装ffmpeg**

```bash
#安装epel包
yum install -y epel-release 
#导入签名 
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7 
#导入签名 
rpm --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro 
#升级软件包 
rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-1.el7.nux.noarch.rpm 
#更新软件包 
yum update -y 
#安装ffmpeg 
yum install -y ffmpeg 
#检查版本 
ffmpeg -version
```

**启动nginx**

```bash
/usr/local/nginx/sbin/nginx
```

**测试**

```bash
ffmpeg -re -i testVideo.mp4 -vcodec copy -codec copy -f flv rtmp://192.168.247.130/hls/testVideo
```

**访问视频**

`http://192.168.247.130/hls/testVideo.m3u8`

# 参考资料

[centos7+nginx+rtmp+ffmpeg搭建流媒体服务器](https://www.cnblogs.com/alexliuzw/p/9792074.html)
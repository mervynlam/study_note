# Docker部署Nginx笔记

## 挂载宿主机的配置文件

配置数据卷`-v ./nginx.conf:/etc/nginx/nginx.conf:ro`

## SSL证书配置

需开放端口`443`，同时挂载宿主机中的`SSL`文件`-p 443:443 -v /ssl:/ssl:rw\`

配置文件`nginx.conf`

```conf
    server {
        listen       443 ssl;
        server_name  domain.com;

        ssl_certificate      /ssl/domain.pem;
        ssl_certificate_key  /ssl/domain.key;

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

        location / {
            proxy_pass http://IP:PORT;
        }
    }
```

## `docker-compose.yml`

```yml
version: '3'

services:
  my-nginx:
    container_name: "nginx"
    image: nginx:latest
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - /ssl:/ssl:rw\
```

## Nginx解析二级域名到不同端口上

```conf
    server {
        listen       80;
        server_name  aaa.domain.com;

        location / {
            proxy_pass http://IP:1234;
        }
    }
    server {
        listen       80;
        server_name  bbb.domain.com;

        location / {
            proxy_pass http://IP:5678;
        }
    }
```


# Centos7 安装 RabbitMQ

## 安装

1. 查看`RabbitMQ`版本对应`eralng`版本

   [RabbitMQ Erlang Version Requirements](https://www.rabbitmq.com/which-erlang.html)

2. 下载对应`erlang`版本

   [rabbitmq / erlang-rpm](https://github.com/rabbitmq/erlang-rpm)

   ```bash
   wget https://github.com/rabbitmq/erlang-rpm/releases/download/v23.1.1/erlang-23.1.1-1.el7.x86_64.rpm
   ```

3. 下载`RabbitMQ`

   [rabbitmq - releases](https://github.com/rabbitmq/rabbitmq-server/releases)

   ```bash
   wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.9/rabbitmq-server-3.8.9-1.el7.noarch.rpm
   ```

4. 安装`erlang`

   ```bash
   yum install -y erlang-23.1.1-1.el7.x86_64.rpm
   ```

5. 安装`RabbitMQ`

   ```bash
   # 先导入签名
   rpm --import https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.9/rabbitmq-server-3.8.9-1.el7.noarch.rpm.asc
   # rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
   # 再安装rabbitmq
   yum install rabbitmq-server-3.8.9-1.el7.noarch.rpm
   ```

## 启动

```bash
# 开机启动
systemctl enable rabbitmq-server.service
# 启动服务
systemctl start rabbitmq-server.service
```

## 参口资料

[RabbitMQ 官方安装文档](https://www.rabbitmq.com/install-rpm.html)
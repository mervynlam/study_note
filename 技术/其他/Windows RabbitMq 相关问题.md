1. 错误`Error: unable to perform an operation on node 'rabbit@DESKTOP-CGB1JQT'.`

   将`C:\Users\{用户名}\.erlang.cookie`

   复制到`C:\Windows\System32\config\systemprofile`目录

2. 安装插件

   ```
   rabbitmq-plugins enable rabbitmq_management
   rabbitmq-plugins enable rabbitmq_delayed_message_exchange
   # 会自动下载
   ```
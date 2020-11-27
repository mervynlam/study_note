1. 安装`ntp`、`ntpdate`

   ```bash
   yum install -y ntp ntpdatexx
   ```

2. 修改`ntp`配置文件

   ```bash
   vim /etc/ntp.conf
   ```

   ```conf
   # 注释这四行
   # server 0.centos.pool.ntp.org iburst
   # server 1.centos.pool.ntp.org iburst
   # server 2.centos.pool.ntp.org iburst
   # server 3.centos.pool.ntp.org iburst
   # 添加一行
   server ntp.aliyun.com iburst
   ```

3. 第一次同步时间先用`ntpdate`

   > ntpd有一个自我保护设置：如果本机与上源时间相差太大，ntpd不运行。所以新设置的时间服务器一定要先ntpdate从上源取得时间初值，然后启动ntpd服务。ntpd服务运行后，先是每64秒与上源服务器同步一次，根据每次同步时测得的误差值经复杂计算逐步调整自己的时间，随着误差减小，逐步增加同步的间隔。每次跳动，都会重复这个调整的过程。

   

   ```bash
   ntpdate ntp.aliyun.com
   ```

4. 启动`ntp`服务并设置开机启动

   ```bash
   systemctl start ntpd
   systemctl enable ntpd
   ```

   

## 参考资料

[CentOS7搭建NTP服务器及客户端同步时间](https://blog.csdn.net/hellboy0621/article/details/81903091)
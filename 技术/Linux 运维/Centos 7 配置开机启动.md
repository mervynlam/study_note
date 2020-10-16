# Centos7 配置开机启动

## 脚本

1. 编写启动脚本

   ```bash
   vim /opt/scripts/nacos-startup.sh
   ```

2. 赋予脚本执行权限

   ```bash
   chmod +x /opt/scripts/nacos-startup.sh
   ```

3. 将脚本添加至开机启动文件

   ```bash
   echo "/opt/scripts/nacos-startup.sh" >> /etc/rc.d/rc.local
   ```

4. 赋予启动文件执行权限

   ```bash
   chmod +x /etc/rc.d/rc.local
   ```

## 参考资料

[Centos7下添加开机自启动服务和脚本](https://blog.csdn.net/GMingZhou/article/details/78677953)

[CentOS 7添加开机启动服务/脚本](https://blog.csdn.net/wang123459/article/details/79063703)
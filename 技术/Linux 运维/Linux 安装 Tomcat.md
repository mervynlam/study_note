[tomcat下载地址](https://tomcat.apache.org/download-80.cgi)

![image-20200929094150755](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200929094221.png)

1. 下载 tomcat

   ```bash
   wget https://mirrors.bfsu.edu.cn/apache/tomcat/tomcat-8/v8.5.58/bin/apache-tomcat-8.5.58.tar.gz
   ```

2. 解压

   ```bash
   tar -zxvf apache-tomcat-8.5.58.tar.gz -C /mervyn
   ```

3. 配置参数

   ```bash
   cd apache-tomcat-8.5.58/bin
   vi ./catalina.sh
   ```

   ```bash
   JAVA_OPTS="-Xms1024m -Xmx4096m -Xss1024K -XX:PermSize=512m -XX:MaxPermSize=2048m -Djava.awt.headless=true"
   ```

4. 启动tomcat

   ```bash
   ./startup.sh
   ```
# 问题情景

Tomcat启动时间很长，甚至卡住不动。

```bash
10-Sep-2020 02:07:50.671 INFO [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory
# 卡在这里几分钟不动
```

# 问题解决

修改 `$JAVA_HOME/jre/lib/security/java.security` 文件

```bash
# 修改 securerandom.source=file:/dev/random 为：
securerandom.source=file:/dev/urandom
```

重启 tomcat ，问题解决。
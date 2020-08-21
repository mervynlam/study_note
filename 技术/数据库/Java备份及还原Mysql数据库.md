# Mysql备份与还原命令
- 备份命令：
    ```bash
    mysqldump -h127.0.0.1 -P3306 -uroot -p123456 mytable > mytable.sql
    ```
- 还原命令：
    ```bash
    mysql -h127.0.0.1 -P3306 -uroot -p123456 mytable < mytable.sql
    ```

`-h`主机
`-P`端口
`-u`用户名
`-p`密码
`mytable`需要备份的库
`mytable.sql`备份的文件名

# Java 实现
```java
//备份
public void backup() {
    String[] execCMD = {"mysqldump", "-h",+hostname, "-P"+ port, "-u"+user, "-p"+password, databaseName, ">", filename};
    try {
        Runtime.getRuntime().exec(execCMD);
    } catch(Exception e) {
        e.printStackTrace();
    }
}
//恢复
public void restore() {
    String[] execCMD = {"mysql", "-h",+hostname, "-P"+ port, "-u"+user, "-p"+password, databaseName, "<", filename};
    try {
        Runtime.getRuntime().exec(execCMD);
    } catch(Exception e) {
        e.printStackTrace();
    }
}
```
windows系统在exceCMD前面加"cmd","/c"
这种方法我备份出来的文件0字节。不清楚原因。解决方法见下文。
```java
//恢复
public void backup() {
    String[] execCMD = {"bash", "./backupSql.sh", hostname, 
port, user, password, databaseName, filename};
    try {
        Runtime.getRuntime().exec(execCMD);
    } catch(Exception e) {
        e.printStackTrace();
    }
}
public void restore() {
    String[] execCMD = {"bash", "./restoreSql.sh", hostname, 
port, user, password, databaseName, filename};
    try {
        Runtime.getRuntime().exec(execCMD);
    } catch(Exception e) {
        e.printStackTrace();
    }
}
```
脚本文件`backupSql.sh`
```bash
mysqldump -h"$1" -P"$2" -u"$3" -p"$4" "$5" > "$6"
```
脚本文件`restoreSql.sh`
```bash
mysql -h"$1" -P"$2" -u"$3" -p"$4" "$5" < "$6"
```
# 升级步骤

1. [下载Mysql5.7](https://dev.mysql.com/downloads/mysql/5.7.html#downloads)

2. 备份旧数据库

   ```bash
   mysqldump -uroot -p123456 --all-databases > all_db.sql
   ```

3. 停止服务

   ```bash
   net stop Mysql
   # Mysql 替换为电脑中的服务名
   # 等待出现 MySQL 服务已成功停止。
   ```

4. 删除服务

   在旧数据库`bin`目录下打开命令行

   ```bash
   mysqld --remove Mysql
   # Mysql 替换为电脑中的服务名
   # 等待出现 Service successfully removed.
   ```

5. 解压下载的`zip`包

6. 修改环境变量

   将5.6版本环境变量修改为5.7版本`bin`目录

7. 新建`my.ini`文件

   ```ini
   [mysql]
   # 设置mysql客户端默认字符集
   default-character-set=utf8 
   [mysqld]
   #设置3306端口
   port = 3306 
   # 设置mysql的安装目录
   basedir=D:\Program Files\MySQL\mysql-5.7.31-winx64
   # 设置mysql数据库的数据的存放目录
   datadir=D:\Program Files\MySQL\mysql-5.7.31-winx64\data
   # 允许最大连接数
   max_connections=200
   # 服务端使用的字符集默认为8比特编码的latin1字符集
   character-set-server=utf8
   # 创建新表时将使用的默认存储引擎
   default-storage-engine=INNODB
   sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 
   ```

8. 初始化数据库

   在5.7版本`bin`目录中打开命令行

   ```bash
   mysqld --initialize --console
   ```

9. 安装服务

   ```bash
   mysqld install
   ```

10. 启动服务

    ```bash
    net start mysql
    # MySQL 服务已经启动成功。
    ```

11. 修改`my.ini`文件，跳过密码检验

    ```ini
    # 添加一行
    skip-grant-tables
    ```

12. 重启服务

    ```bash
    net stop mysql
    # MySQL 服务已成功停止。
    net start mysql
    # MySQL 服务已经启动成功。
    ```

13. 进入`mysql`修改密码

    ```bash
    mysql
    ```

    ```mysql
    use mysql;
    update user set authentication_string=password("123456") where user="root";
    # 123456 替换成你的新密码
    flush privileges;
    quit;
    ```

14. 修改`my.ini`文件，去掉跳过密码检验

    ```ini
    # 去掉这行
    skip-grant-tables
    ```

15. 重启服务

    ```bash
    net stop mysql
    # MySQL 服务已成功停止。
    net start mysql
    # MySQL 服务已经启动成功。
    ```
    
16. 使用新密码登录`mysql`，开启远程登录

    ```bash
    mysql -uroot -p123456
    ```

    ```mysql
    alter user 'root'@'localhost' identified by '123456';
    # 123456 替换成你的新密码
    use mysql;
    update user set host='%' where user='root';
    flush privileges;
    quit;
    ```

17. 恢复数据

    ```bash
    mysql -uroot -p123456 < all_db.sql
    # 漫长的等待开始了
    ```

---

升级后导出数据库报错

```
1577 - Cannot proceed because system tables used by Event Scheduler were found damaged at server start
```

解决：

```bash
mysqlcheck -uroot -p123456 --all-databases --check-upgrade --auto-repair
#123456为数据库密码
mysql_upgrade -uroot -p123456
```

后重启mysql服务

```bash
net stop mysql
net start mysql
```

# 参考资料

[【MySQL】Windows下mysql5.6升级到5.7的方法](https://blog.csdn.net/cloverat/article/details/106264217)

[解决导出数据库报错](https://www.jianshu.com/p/f044e12a2f9f)
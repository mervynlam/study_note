1. 添加用户和用户组
	```
	groupadd mysql
	useradd -r -g mysql mysql
	```

2. 下载mysql
下载地址[https://dev.mysql.com/downloads/mysql/5.7.html#downloads](https://dev.mysql.com/downloads/mysql/5.7.html#downloads)
根据系统选择
将下载的压缩包推到`/usr/local`目录下
3. 解压
进入压缩包所在目录
	```
	cd /usr/local
	```
	解压
	```
	tar -zxvf mysql-5.7.16-linux-glibc2.5-x86_64.tar.gz
	```
	修改文件名为mysql方便后续操作
	```
	mv mysql-5.7.16-linux-glibc2.5-x86_64 mysql
	```

4. 创建data文件夹
	```
	cd mysql
	mkdir data
	```

5. 修改mysql文件夹权限
	```
	cd /usr/local/
	chown -R mysql mysql/
	chgrp -R mysql mysql/
	```

5. 配置mysql
	3. 创建mysql.sock文件并配置权限
		```
        cd /usr/local/mysql/
		mkdir tmp
		cd tmp
		touch mysql.sock
		chown -R mysql /usr/local/mysql/tmp
		chgrp -R mysql /usr/local/mysql/tmp
		chmod 755 /usr/local/mysql/tmp/mysql.sock
		ln -s /usr/local/mysql/tmp/mysql.sock /tmp/mysql.sock
		```
	4. 创建mysqld.log文件并配置权限
		```
        cd /usr/local/mysql/
		mkdir log
		cd log
		touch mysqld.log
		chown -R mysql /usr/local/mysql/log
		chgrp -R mysql /usr/local/mysql/log
		chmod 755 /usr/local/mysql/log/mysqld.log
		```
	5. 创建mysqld.pid文件并配置权限
		```
        cd /usr/local/mysql/tmp
		touch mysqld.pid
		chown -R mysql /usr/local/mysql/tmp/mysqld.pid
		chmod 755 /usr/local/mysql/tmp/mysqld.pid
		```
7. 初始化mysql
初始化mysql
	```
	cd /usr/local/mysql/
	bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
	```
此处会生成初始化密码，后续将修改
8. 配置mysql文件
删除原有my.cnf文件，并生成新的
```
rm -rf /etc/my.cnf
vi /etc/my.cnf
```

复制以下内容
```
[mysqld]
# 默认引擎
default-storage-engine=innodb
innodb_file_per_table
# 字符集
collation-server=utf8_general_ci
character_set_server=utf8
init_connect='SET NAMES utf8'
#路径
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
socket=/usr/local/mysql/tmp/mysql.sock
log-error=/usr/local/mysql/log/mysqld.log
pid-file=/usr/local/mysql/tmp/mysqld.pid
port = 3306
#表名不区分大小写
lower_case_table_names = 1
max_connections=5000
sql_mode=STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION,NO_ZERO_IN_DATE,NO_ZERO_DATE
```
9. 添加mysql服务

	```
	cd /usr/local/mysql/
	cp -a ./support-files/mysql.server  /etc/init.d/mysqld
	```
	
	```
	cd bin/
	./mysqld_safe --user=mysql &
	/etc/init.d/mysqld restart
	```
	
	服务开机自启
	
	```
	systemctl enable mysqld
	```

10. 加入全局变量

	```
	echo 'export PATH=/usr/local/mysql/bin:$PATH' >> /etc/profile
	source /etc/profile
	```

11. 修改密码
	1. 配置跳过密码认证 
	修改my.cnf文件
	
	    ```
	    vi /etc/my.cnf
	    ```

		添加一行
		
	    ```
	    skip-grant-tables
	    ```

		保存退出
		重启mysql
		
	    ```
	    service mysqld restart
	    ```

	2. 登录mysql修改密码
	
	    ```
	    ./mysql
	    mysql> use mysql;
	    mysql> update user set authentication_string=password("你的新密码") where user="root";
	    mysql> flush privileges;
	    mysql> quit
		```

    3. 去掉跳过密码验证
	修改my.cnf文件
	
	    ```
	    vi /etc/my.cnf
	    ```
	
		去掉
		
	    ```
	    skip-grant-tables
	    ```
	
		保存退出
		重启mysql
		
		```
		service mysqld restart
		```

12. 设置允许远程登陆mysql

	```
	./mysql -uroot -p新密码
	```
	```
	mysql> alter user 'root'@'localhost' identified by '修改的密码';
	mysql> flush privileges;
	mysql> use mysql
	mysql> update  user  set host='%' where user='root';
	mysql> flush privileges;
	mysql> quit
	```
	重启mysql
	
	```
	service mysqld restart
	```

**需关闭防火墙或开启3306端口方可远程连接**

13. 关闭防火墙

	```
	systemctl stop firewalld
	```

	禁止防火墙开机自启
	
	```
	systemctl disable firewalld
	```


---
参考博客：[https://blog.csdn.net/pdsu161530247/article/details/81585889](https://blog.csdn.net/pdsu161530247/article/details/81585889)
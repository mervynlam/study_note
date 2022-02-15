# Windows、Linux定时备份Mysql数据库

## 备份脚本

之前写过一篇[Mysql备份与还原命令](https://blog.csdn.net/Rec_Mervyn/article/details/108009712)，其中今天的备份脚本就是用的这篇文章提到的`mysqldump`。

`mysqldump`是`mysql`自带的转储数据库以进行备份的客户端工具。常用于备份数据库

```bash
# mysqldump -h主机 -P端口 -u用户 -p密码 [选项] > 转储目标文件
mysqldump -h127.0.0.1 -P3306 -uroot -p --all-databases > E:\all.sql
# 其中 --all-databases 为转储所有数据库
# 常用还有 --databases，指定一个或多个数据库
# --ignore-table，排除某张表
# 其他选项自行搜索学习
```

## 定时任务

### Linux

`Linux`使用`crontab`实现自动执行脚本。

1. 创建`backup.sh`文件，编写备份语句，赋予执行权限`chmod a+x backup.sh`
2. 设置`crontab`定时任务
   1. 打开配置文件`crontab -e`
   2. 加入定时任务`0 0 * * * /root/backup.sh`，示例为每天0时执行脚本
   3. 保存

### Windows

1. 创建`backup.bat`文件，编写备份语句。
2. 搜索任务计划程序并打开。
3. 创建基本任务
   1. 填写名称
   2. 触发器根据自己需要填写
   3. 选择备份脚本文件
   4. 保存

## 删除过时备份文件

每天备份数据库最终会出现备份文件过多，就备份文件无用、过时的问题。可以添加删除文件语句到定时任务脚本中。实现仅保留指定日期后的文件。

### Linux

```bash
find /root/backup -mtime +7 -name "all*.sql" -exec rm -Rf {} \;
# 查找/root/backup文件夹下，7天以前的，名字为"all*.sql"的文件，执行删除操作
# mtime是文件筛选的时间条件，+n为n天前，-n为n天内
```

结合备份语句

```bash
find /root/backup -mtime +7 -name "all*.sql" -exec rm -Rf {} \;
mysqldump -h127.0.0.1 -P3306 -uroot -p123456 --all-databases > /root/backup/all_$(date "+%Y%m%d%H%M%S").sql
# 数据库备份到指定目录/root/backup中，文件命名为all_备份时的日期时间.sql
# 删除7天前的指定文件
# 配合定时任务即可实现，每天自动备份数据库，且备份文件夹中仅保留最近7天的备份数据。
```

### Windows

```bash
forfiles /p "E:\backup" /m "all*.sql" /d -7 /c "cmd /c del /f /q /a @path"
# /p 指定文件路径
# /m 查找文件名
# /d 指定日期，-7为7天前
# /c 运行命令行
# @path 返回文件的完整路径
# 实现删除指定目录、名称，七天前的文件
```

结合备份语句

```bash
mysqldump -h127.0.0.1 -P3306 -uroot -p123456 --all-databases > /root/backup/all_$(date "+%Y%m%d%H%M%S").sql@echo off
::演示：删除指定路径下指定天数之前（以文件的最后修改日期为准）的文件。
::如果演示结果无误，把del前面的echo去掉，即可实现真正删除。
::本例需要Win7/Win2008/Win2012/Win2016自带的forfiles命令的支持

rem 指定待删除文件的存放路径
set SrcDir=E:\backup\
rem 指定天数
set DaysAgo=7
forfiles /p %SrcDir% /m "all*.sql" /d -%DaysAgo% /c "cmd /c echo del /f /q /a @path"
mysqldump -h127.0.0.1 -P3306 -uroot -p123456 --all-databases > E:\backup\all_%date:~0,4%%date:~5,2%%date:~8,2%%time:~0,2%%time:~3,2%%time:~6,2%.sql
# 数据库备份到指定目录E:\backup\中，文件命名为all_备份时的日期时间.sql
# 删除7天前的指定文件
# 配合定时任务即可实现，每天自动备份数据库，且备份文件夹中仅保留最近7天的备份数据。
```

## 参考资料

[批处理bat脚本删除指定天数日期之前的文件](https://blog.csdn.net/haisong/article/details/104483046)

[Linux 按时间批量删除文件（删除N天前文件）](https://blog.csdn.net/zouxucong/article/details/101292605)

[【批处理】打印当前日期和时间](https://blog.csdn.net/jiangwei0512/article/details/102770623)
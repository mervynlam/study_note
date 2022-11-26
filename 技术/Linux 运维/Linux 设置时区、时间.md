1. linux 时间
    ```bash
    # 查看时间
    date
    # 设置时间
    date -s "2020-07-09 17:20:00"
    ```
2. linux 时区
    1. CentOs7
        ```bash
        timedatectl set-timezone Asia/Shanghai
        ```
    2. 其他
        ```bash
        [root@linux-node ~]# tzselect
        [root@linux-node ~]# echo "ZONE=Asia/Shanghai" >> /etc/sysconfig/clock
        [root@linux-node ~]# rm -f /etc/localtime
        #链接到上海时区文件
        [root@linux-node ~]# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
        ```
3. tomcat 时间
    修改`bin/catalina.sh`
    ```bash
    export JAVA_OPTS="$JAVA_OPTS -Duser.timezone=Asia/hongkong"
    ```
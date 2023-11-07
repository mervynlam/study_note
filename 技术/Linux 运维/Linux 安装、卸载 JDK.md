# 在线安装

以下内容以centos7为例。
检查是否已安装jdk

```bash
java -version
```
若版本合适可不卸载，直接使用。

**卸载JDK**

查看已安装的JDK环境
```bash
yum list installed |grep java
```
![image-20200910140246956](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071636546.png)

卸载

```bash
yum -y remove java-1.7.0-openjdk*
```
```bash
yum -y remove tzdata-java.noarch
```

**安装JDK**

查看JDK软件包列表
```bash
yum search java | grep -i --color jdk
```
选择版本安装
```bash
yum install -y java-1.8.0-openjdk java-1.8.0-openjdk-devel
```
或者如下命令，安装jdk1.8.0的所有文件
```bash
yum install -y java-1.8.0-openjdk*
```
查看JDK是否安装成功
```bash
java -version
```

# 离线安装

[二进制文件下载](https://www.oracle.com/java/technologies/javase-downloads.html)

检查是否已安装jdk

```bash
java -version
```

查看java安装软件

```bash
rpm -qa|grep java
```

![image-20200910134824346](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071636205.png)

**卸载**

```bash
rpm -e --nodeps java-1.8.0-openjdk-headless-1.8.0.242.b08-0.el7_7.x86_64
rpm -e --nodeps java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64
rpm -e --nodeps java-1.8.0-openjdk-devel-1.8.0.242.b08-0.el7_7.x86_64
```

再次检查

```bash
rpm -qa|grep java
```

**安装**

```bash
# 创建目录
mkdir /usr/local/java
# 解压二进制文件
tar -zxvf /usr/local/java/jdk-8u221-linux-x64.tar.gz -C /usr/local/java
```

**环境变量配置**

```bash
vi /etc/profile
```

末尾添加

```bash
export JAVA_HOME=/usr/local/java/jdk1.8.0_221
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
export JAVA_PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin
export PATH=$PATH:${JAVA_PATH}
```

保存后刷新环境变量

```bash
source /etc/profile
```

验证安装是否完成

```java
java -version
```


[TOC]

# Nacos 

<!--开始学习时间20200428-->

## Nacos 简介

### 简介

Nacos是针对微服务架构中的服务发现、配置管理、服务治理的综合性解决方案。

### 特性

- 服务发现与服务健康检查
- 动态配置管理
- 动态DNS服务
- 服务和元数据管理

### 安装

1. 下载压缩包

   [官方git-releases](https://github.com/alibaba/nacos/releases)

2. 解压

3. 启动服务

   - Windows

     进入`bin`目录启动`startup.cmd`

   - Linux

     进入`bin`目录执行命令

     ```bash
     sh shtartup.sh -m standalone
     ```

4. 访问 http://localhost:8848/nacos/index.html

   默认用户名：nacos，默认密码：nacos
   
5. 外部mysql数据库支持

   1. 安装mysql数据库（5.6.5+）

   2. 创建数据库`nacos_config`，数据库初始化文件：`nacos/conf/nacos-mysql.sql`

   3. 修改`nacos/conf/application.properties`文件，增加mysql数据源配置

      ```properties
      spring.datasource.platform=mysql
      
      db.num=1
      db.url.0=jdbc:mysql://localhost:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
      db.user=root
      db.password=123456
      ```

## Nacos 配置管理

### Nacos 配置管理 - 基础应用

#### Nacos 配置管理模型

对于Nacos配置管理，通过Namespace、group、Data ID来定位到一个配置集。

- 配置集 (Data ID)

  一个配置文件通常就是一个配置集

- 配置项

  配置集中的包含的一个个配置内容就是配置项

- 配置分组 (Group)

  不同配置分组下可以有相同的配置集

- 命名空间 (Namespace)

  可用于进行不同环境的配置隔离。不同命名空间可以有相同的配置分组或配置集。

#### Nacos 配置管理 - Java实现

1. 新建maven项目。

2. 增加maven依赖

   ```xml
   <dependency>
       <groupId>com.alibaba.nacos</groupId>
       <artifactId>nacos-client</artifactId>
       <version>1.2.1</version>
   </dependency>
   ```

3. 创建类，根据信息获取配置

   ```java
   public static void main(String[] args) throws NacosException {
       //nacos url
       String serverAddr = "127.0.0.1:8848";
       //DataId
       String dataId = "db.properties";
       //group
       String group = "DEFAULT_GROUP";
       //namespace
       String namespace = "bfb75518-0f3d-47b1-b520-a27d21206013";
       //属性
       Properties properties = new Properties();
       properties.put("serverAddr", serverAddr);
       //指定命名空间，若不指定则默认public
       //properties.put("namespace", namespace);
       //获取配置
       ConfigService configService = NacosFactory.createConfigService(properties);
       String config = configService.getConfig(dataId, group, 5000);
       System.out.println(config);
   }
   ```
   
4. 监听配置修改

   ```java
       configService.addListener(dataId, group, new Listener() {
           public Executor getExecutor() {
               return null;
           }
   
           public void receiveConfigInfo(String s) {
               System.out.println(s);
           }
       });
   ```


### Nacos 配置管理 - 分布式系统应用

#### 单一配置文件实现

1. 创建父工程

   `pom.xml`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>com.mervyn</groupId>
       <artifactId>nacos_config</artifactId>
       <version>1.0-SNAPSHOT</version>
       <packaging>pom</packaging>
   
       <properties>
           <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
           <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
           <java.version>1.8</java.version>
       </properties>
   
       <dependencyManagement>
           <dependencies>
               <dependency>
                   <groupId>com.alibaba.cloud</groupId>
                   <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                   <version>2.1.0.RELEASE</version>
                   <type>pom</type>
                   <scope>import</scope>
               </dependency>
               <dependency>
                   <groupId>org.springframework.cloud</groupId>
                   <artifactId>spring-cloud-dependencies</artifactId>
                   <version>Greenwich.RELEASE</version>
                   <type>pom</type>
                   <scope>import</scope>
               </dependency>
               <dependency>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-dependencies</artifactId>
                   <version>2.1.3.RELEASE</version>
                   <type>pom</type>
                   <scope>import</scope>
               </dependency>
           </dependencies>
       </dependencyManagement>
   
       <build>
           <plugins>
               <plugin>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
               </plugin>
           </plugins>
       </build>
   </project>
   ```

2. 创建子工程 service1、service2

   `pom.xml`

   ```xml
   <dependencies>
       <dependency>
           <groupId>com.alibaba.cloud</groupId>
           <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-web</artifactId>
       </dependency>
   </dependencies>
   ```

3. `bootstrap.yml` 配置

   选择`bootstrap.yml`而非`application.yml`是因为`bootstrap.yml`启动优先级高于`application.yml`

   ```yaml
   server:
     port: 56010 #启动端口 命令行注入
   
   spring:
     application:
       name: service1
     cloud:
       nacos:
         config:
           server-addr: 127.0.0.1:8848 # 配置中心地址
           file‐extension: yaml # dataId = application.name + file-extension = service1.yaml 与nacos发布的配置相同
           namespace: bfb75518-0f3d-47b1-b520-a27d21206013 # 命名空间 - 测试环境
           group: TEST_GROUP # 测试组
   ```

4. 创建Springboot启动类

   ```java
   package com.mervyn.nacos.service1;
   
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.context.ConfigurableApplicationContext;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   @SpringBootApplication
   @RestController
   public class Service1Bootstrap {
       public static void main(String[] args) {
           SpringApplication.run(Service1Bootstrap.class, args);
       }
   
       //通过value注解读取配置信息，但无法动态更新配置的修改
       @Value("${common.name}")
       private String config1;
   
       //配置文件上下文，实现动态获取配置更新
       @Autowired
       private ConfigurableApplicationContext applicationContext;
   
       @GetMapping("/configs")
       public String getConfigs() {
           //读取配置信息
   //        return config1;
           return applicationContext.getEnvironment().getProperty("comman.name");
       }
   }
   ```

#### 自定义扩展 dataId

   应用于一个微服务不止一个配置文件。

   `bootstrap.yml`

   ```yaml
   # config external configuration
   # 1、Data Id 在默认的组 DEFAULT_GROUP,不支持配置的动态刷新
   ext-config[0]:
   data-id: ext-config-common01.properties
   
   # 2、Data Id 不在默认的组，不支持动态刷新
   ext-config[1]:
   data-id: ext-config-common02.properties
   group: GLOBALE_GROUP
   
   # 3、Data Id 既不在默认的组，也支持动态刷新
   ext-config[2]:
   data-id: ext-config-common03.properties
   group: REFRESH_GROUP
   refresh: true
   ```

#### 自定义共享 dataId

   该方法仅支持加载`DEFAULT_GROUP`中的配置，故不推荐使用

   `bootstrap.yml`

   ```yaml
   shared-dataids: ext-config-common01.properties,ext-config-common02.properties,ext-config-common03.properties
   refreshable-dataids: ext-config-common03.properties
   ```

#### 配置优先级

上述三种配置方式：

> A. `spring.cloud.nacos.config.shared-dataids`
> B. `spring.cloud.nacos.config.ext-config[n].data-id`
> C. 通过`application.name`和`file-extension`自动生成的dataId

优先级为 C > B > A。

B方式中多个配置优先级为：n越大，优先级越高。

### Nacos 集群部署

1. 安装3个以上 Nacos

2. 修改配置文件

   各 Nacos 配置文件`conf/application.properties`中`server.port`修改为不同端口

3. 修改集群配置

   各 Nacos 修改`cluster.conf.example`改为`cluster.conf`

   并修改内容为

   ```
   #ip:port
   127.0.0.1:8848
   127.0.0.1:8849
   127.0.0.1:8850
   ```

4. 分别执行各 Nacos 下的 `startup`

   ```
   startup -m cluster
   ```

5. 应用配置

   ```yaml
   spring:
     application:
       name: applicationName
     cloud:
       nacos:
         config:
           server-addr: 127.0.0.1:8848, 127.0.0.1:8849, 127.0.0.1:8850 # 配置集群地址
   ```

## Nacos 服务发现

### Nacos 服务发现 - 快速入门

1. 创建父工程

   `pom.xml`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>com.mervyn</groupId>
       <artifactId>nacos_discovery</artifactId>
       <version>1.0-SNAPSHOT</version>
       <packaging>pom</packaging>
   
       <properties>
           <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
           <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
           <java.version>1.8</java.version>
       </properties>
   
       <dependencyManagement>
           <dependencies>
               <dependency>
                   <groupId>com.alibaba.cloud</groupId>
                   <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                   <version>2.1.0.RELEASE</version>
                   <type>pom</type>
                   <scope>import</scope>
               </dependency>
               <dependency>
                   <groupId>org.springframework.cloud</groupId>
                   <artifactId>spring-cloud-dependencies</artifactId>
                   <version>Greenwich.RELEASE</version>
                   <type>pom</type>
                   <scope>import</scope>
               </dependency>
               <dependency>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-dependencies</artifactId>
                   <version>2.1.3.RELEASE</version>
                   <type>pom</type>
                   <scope>import</scope>
               </dependency>
           </dependencies>
       </dependencyManagement>
   
       <build>
           <plugins>
               <plugin>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
               </plugin>
           </plugins>
       </build>
   </project>
   ```

2. 创建子工程 - 服务生产者

   1. `pom.xml`

       ```xml
       <dependencies>
           <dependency>
               <groupId>com.alibaba.cloud</groupId>
               <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
           </dependency>

           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>

           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-openfeign</artifactId>
           </dependency>
       </dependencies>
       ```
   
   2. `application.yml`
   
       ```yaml
       server:
         port: 56010
       spring:
         application:
           name: quickstart-provider # 不支持_下划线
         cloud:
           nacos:
             discovery:
               server-addr: 127.0.0.1:8848
       logging:
         level:
           root: info
           org.springframework: info
       ```
   
   3. 创建`controller`提供接口
   
       ```java
       @RestController
       public class ProviderController {
       
           private static final Logger LOG = LoggerFactory.getLogger(ProviderController.class);
       
           @GetMapping("/service")
           public String service() {
               LOG.info("provider service");
               return "provider service";
           }
       }
       ```
   
   4. 创建启动程序
   
       ```java
       @SpringBootApplication
       //开启feign远程链接
       @EnableFeignClients
       //是一个注册发现客户端
       @EnableDiscoveryClient
       public class NacosProviderApp {
       
           public static void main(String[] args) {
               SpringApplication.run(NacosProviderApp.class, args);
           }
       
       }
       ```
   
3. 创建子工程 - 服务消费者

   1. `pom.xml` 同上

   2. `application.yml`

      ```yaml
      server:
        port: 56020
      spring:
        application:
          name: quickstart-consumer
        cloud:
          nacos:
            discovery:
              server-addr: 127.0.0.1:8848
      ```

   3. 创建 服务生产者的客户端 接口

      ```java
      @FeignClient(name = "quickstart-provider")
      public interface ProviderClient {
      
          @GetMapping("/service")
          public String service();
      
      }
      ```

   4. 创建 服务消费者 功能实现

      ```java
      @RestController
      public class ConsumerController {
      
          @Autowired
          private ProviderClient providerClient;
      
          @GetMapping("service")
          public String service() {
              String providerResult = providerClient.service();
              return "consumer invoke | " + providerResult;
          }
      
      }
      ```

   5. 创建 服务消费者 启动类

      ```java
      @SpringBootApplication
      @EnableDiscoveryClient
      @EnableFeignClients
      public class NacosConsumerApp {
      
          public static void main(String[] args) {
              SpringApplication.run(NacosConsumerApp.class, args);
          }
      
      }
      ```

      

## 参考资料

[Spring Cloud Alibaba Nacos配置中心与服务发现 - 黑马程序员](https://www.bilibili.com/video/BV1VJ411X7xX)


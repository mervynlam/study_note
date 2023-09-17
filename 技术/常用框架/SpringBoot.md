# SpringBoot

## 快速开始一个SpringBoot项目

1. 创建项目

   ```xml
   <!--    所有springboot项目都必须继承自 spring-boot-starter-parent -->
   <parent>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-parent</artifactId>
       <version>3.0.5</version>
   </parent>
   ```

2. 导入场景启动器

   ```xml
       <dependencies>
   <!--        web开发的场景启动器 -->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
       </dependencies>
   ```

3. 主程序

   ```java
   @SpringBootApplication //这是一个SpringBoot应用
   public class MainApplication {
       public static void main(String[] args) {
           SpringApplication.run(MainApplication.class,args);
       }
   }
   ```

4. 编写业务

   ```java
   @RestController
   public class HelloController {
       @GetMapping("/hello")
       public String hello(){
           return "Hello,Spring Boot 3!";
       }
   }
   ```

5. 打包

   ```xml
   <!--    SpringBoot应用打包插件-->
   <build>
       <plugins>
           <plugin>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-maven-plugin</artifactId>
           </plugin>
       </plugins>
   </build>
   ```

## 依赖管理机制

每个`boot`项目都有一个父项目`spring-boot-starter-parent`，`parent`项目的父项目是`spring-boot-dependencies`，在该项目中把所有常见的依赖版本都声明了。

开发什么场景，就导入相应的场景启动器。场景启动器中包含了该场景需要的所有核心依赖。

## @SpringBootApplication注解

可以看作是`@Configuration`、`@ComponentScan`和`@EnableAutoConfiguration`注解的集合。

- `@ComponentScan`扫描被`@Component`注解的类，默认扫描启动类所在包下的所有的类。
- `@Configuration`允许在上下文中注册额外的`bean`或导入其他配置类
- `@EnableAutoConfiguration`启用`SpringBoot`自动配置机制

### 自动配置

`@EnableAutoConfiguration`注解通过`@Import`导入了`AutoConfigurationImportSelector`类。`AutoConfigurationImportSelector`类中的`getCandidateConfigurations`方法将所有自动配置类的信息以`List`的形式返回。这些配置信息会被`Spring`容器管理。

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    List<String> configurations = ImportCandidates.load(AutoConfiguration.class, this.getBeanClassLoader()).getCandidates();
    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports. If you are using a custom packaging, make sure that file is correct.");
    return configurations;
}
```

**自动配置类**位于`spring-boot-autoconfigure`包中，每一个场景启动器`spring-boot-starter-*`都有`spring-boot-starter`依赖，而`spring-boot-starter`中有`spring-boot-autoconfigure`依赖，保证了每个场景启动器都可以引入自动配置类。

但自动配置类并不是全部开启的，而是通过配置类中的**条件注解**来判断是否开启。

#### 完整流程

![image](https://raw.githubusercontent.com/mervynlam/Pictures/master/202309111901989.png)

1. 导入场景启动器
   1. 场景启动器导入了相关场景的所有核心依赖
   2. 每个场景启动器都引入了一个`spring-boot-starter`，核心场景启动器
   3. `spring-boot-starter`引入了`spring-boot-autoconfigure`包
   4. `spring-boot-autoconfigure`包中囊括了所有场景的所有配置
   5. 通过条件注解来使相应的配置生效
2. 主程序`@SpringBootApplication`
   1. 由`@Configuration`、`@ComponentScan`和`@EnableAutoConfiguration`组成
   2. `ComponentScan`默认只扫描主程序所在包下的所有类，自动配置类需要通过`@EnableAutoConfiguration`启动
      1. `@EnableAutoConfiguration`启动自动配置
         1. `@Import(AutoConfigurationImportSelector.class)`批量导入所有位于`spring-boot-autoconfigure`下 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`的配置类
         2. 根据条件注解按需生效
3. `xxxAutoConfiguration`自动配置类
   1. 通过相关属性类给容器导入组件
   2. 通过`@EnableConfigurationProperties(xxxProperties.class)`把配置文件和属性类字段一一匹配
4. 写业务

#### 核心流程

1. 导入`starter`，就会导入`autoconfigure`包
2. `autoconfigure`包中有一个文件`META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`，里面包含所有启动要加载的自动配置类
3. 通过`@EnableAutoConfiguration`注解，会将上面所有的自动配置类导入，根据条件注解按需加载。
4. `xxxAutoConfiguration`给容器导入一些组件，组建是从`xxxProperties`属性类中提取属性值。
5. `xxxProperties`属性类通过`@EnableConfigurationProperties()`注解进行属性绑定。

# 参考资料

[《Java 面试指北》](https://javaguide.cn/zhuanlan/java-mian-shi-zhi-bei.html)

[尚硅谷SpringBoot3](https://www.bilibili.com/video/BV1Es4y1q7Bf)
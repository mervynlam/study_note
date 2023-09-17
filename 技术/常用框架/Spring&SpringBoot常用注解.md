#### `@SpringBootApplication`

可以看作是`@Configuration`、`@ComponentScan`和`@EnableAutoConfiguration`的集合

##### `@Configuration`

允许在 Spring 上下文中注册额外的 `bean` 或导入其他配置类

##### `@ComponentScan`

扫描被`@Component`(`@Service`、`@Controller`、`@Repository`)注解的`Bean`，默认扫描主程序所在包下的所有类

##### `@EnableAutoConfiguration`

启用`SpringBoot`的自动配置机制

#### Spring Bean 相关

##### `@Autowired`

自动装配`Bean`，`Spring`容器会在需要的地方将被管理的`Bean`注入。

##### `@Component`

通用的注解，可以标注任意类为`Bean`

##### `@Repository`

作用于`Dao`层

##### `@Service`

作用于`Service`层

##### `@Controller`

作用于`Controller`层

##### `@RestController`

是`@Controller`和`ResponseBody`的合集，表示该是控制器`Bean`并且将函数返回值直接填入`HTTP`响应体中

##### `@Scope`

生命`Bean`的作用域

- `singleton`唯一 bean 实例，Spring 中的 bean 默认都是单例的
- `prototype`每次请求都会创建一个新的 bean 实例。
- `request`每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP request 内有效。
- `session`每一个 HTTP Session 会产生一个新的 bean，该 bean 仅在当前 HTTP session 内有效。

##### `@Configuration`

用来生命配置类，在上下文中可以注册额外的`bean`和导入其他配置类

#### 处理常见的 HTTP 请求类型

##### `GetMapping`、`PutMapping`、`PostMapping`、`DeleteMapping`、`PatchMapping`

**GET**：请求从服务器获取特定资源。

**POST**：在服务器上创建一个新的资源。

**PUT**：更新服务器上的资源（客户端提供更新后的整个资源）。

**DELETE**：从服务器删除特定的资源。

**PATCH**：更新服务器上的资源（客户端提供更改的属性，可以看做作是部分更新）

#### 前后端传值

##### `@PathVariable` 、 `@RequestParam`

`@PathVariable` 用于获取路径参数， `@RequestParam`用于获取查询参数

```java
@GetMapping("/klass/{klassId}/students")
public User getStudents(@PathVariable("klassId") Long  klassId,
                       @RequestParam String name){}
```

##### `@RequestBody`

用于读取 Request 请求（可能是 POST,PUT,DELETE,GET 请求）的 body 部分并且**Content-Type 为 application/json** 格式的数据，接收到数据之后会自动将数据绑定到 Java 对象上去

**一个请求方法只可以有一个`@RequestBody`，但是可以有多个`@RequestParam`和`@PathVariable`**

#### 读取配置信息

##### `@Value`

获取单个配置到指定属性

##### `@ConfigurationProperties(prefix="xxx")`

将指定前缀的配置与类中的同名字段一一绑定。

通常需要加`@Component`注解声明为`SpringBean`才可使用，或者使用`@EnableConfigurationProperties`注解

`@EnableConfigurationProperties(xxxPreperties.class)`

用来把配置文件中配的指定前缀的属性值封装到 `xxxProperties`**属性类**中，同时将该类注册到`Spring`容器中

#### 字段校验

`@NotEmpty` 被注释的字符串的不能为 null 也不能为空

`@NotBlank` 被注释的字符串非 null，并且必须包含一个非空白字符

`@Null` 被注释的元素必须为 null

`@NotNull` 被注释的元素必须不为 null

`@AssertTrue` 被注释的元素必须为 true

`@AssertFalse` 被注释的元素必须为 false

`@Pattern(regex=,flag=)`被注释的元素必须符合指定的正则表达式

`@Email` 被注释的元素必须是 Email 格式。

`@Min(value)`被注释的元素必须是一个数字，其值必须大于等于指定的最小值

`@Max(value)`被注释的元素必须是一个数字，其值必须小于等于指定的最大值

`@DecimalMin(value)`被注释的元素必须是一个数字，其值必须大于等于指定的最小值

`@DecimalMax(value)` 被注释的元素必须是一个数字，其值必须小于等于指定的最大值

`@Size(max=, min=)`被注释的元素的大小必须在指定的范围内

`@Digits(integer, fraction)`被注释的元素必须是一个数字，其值必须在可接受的范围内

`@Past`被注释的元素必须是一个过去的日期

`@Future` 被注释的元素必须是一个将来的日期

#### `@Transactional`

开启事务，可以作用在类上，也可以作用在方法上。如果作用在类上，类中所有的方法都将开启事务。有些只读类可以通过`@Transactional(readOnly=true)`来标注。

#### 参考资料

[JavaGuide - Spring&SpringBoot常用注解总结](https://javaguide.cn/system-design/framework/spring/spring-common-annotations.html)
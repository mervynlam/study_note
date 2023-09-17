# `Spring`

## `IoC`（`Inversion of Control`:控制反转）

`IoC`思想是将原本在程序中手动创建对象的控制权，交由`Spring`框架来管理.

- 控制：指对象创建（实例化、管理）的权利
- 反转：控制权交给外部环境（`Spring`框架、`IoC`容器）

### `SpringBean`

被`IoC`容器所管理的对象称为`SpringBean`。

通过注解：

- `@Component`：可以标注任意类为`Spring`组件。
- `@Repository`：对应持久层`Dao`
- `@Service`：对应服务层
- `@Controller`：对应控制层

### `@Component` 和`@Bean`注解

- `@Component`注解作用于类，`@Bean`作用于方法
- `@Component` 通常是通过类路径扫描来自动侦测及装配，而`@Bean`通常是我们在标注该注解的方法中产生这个`bean`，告诉`Spring`这个类的实例

### Bean注入的注解

- `@Autowired`
  - `org.springframework.bean.factory`
  - `byType` -> `byName`
  - 优先根据接口类型去匹配并注入`Bean`，当接口存在多个实现类的话，这种方式就无法正确注入对象了，因为这个时候`Spring`同时找到多个满足的条件的选择，默认情况下他不知道应该选择哪一个。这种情况下注入方式会变成`byName`。这个名称是变量名匹配类名。但建议使用`@Qualifier`注解来显示指定名称。
- `@Resource`
  - 属于`JDK`提供的注解
  - `byName` -> `byType`
  - `@Resource(name = xxxServiceImpl)` 等价于 `@Autowire`+`@Qualifier`

### Bean作用域

- `singleton`：单例，`IoC`容器中只有唯一的实例
- `prototype`：每次获取都会创建一个新的实例
- `request`：每一次`HTTP`请求都会产生一个新的实例，仅在当前`HTTP request`中有效
- `session`：每一次新`session`都会产生一个新的实例，仅在当前`HTTP session`中有效
- `application/global-session`：每一次Web 应用都会产生一个新的实例，仅在当前应用启动中有效
- `websocket`：每一次 `WebSocket` 会话产生一个新的 `bean`。

### Bean线程安全问题

- `prototype`作用域下，每次获取都会创建一个新的实例，不存在资源竞争的问题，所以不存在线程安全问题。
- `singleton`作用域下，`IoC`容器只有唯一的实例，可能会存在资源竞争问题。如果这个`Bean`是有状态的话，存在线程安全问题

## `Aop`（`Aspect-Oriented Programming`：面向切面编程）

将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

`Spring AOP` 就是基于**动态代理**的，如果要代理的对象，实现了某个接口，那么 `Spring AOP` 会使用 **`JDK Proxy`**，去创建代理对象，而对于没有实现接口的对象，就无法使用 `JDK Proxy` 去进行代理了，这时候 `Spring AOP` 会使用 **`Cglib`** 生成一个被代理对象的子类来作为代理。

| 术语              |                             含义                             |
| :---------------- | :----------------------------------------------------------: |
| 目标(Target)      |                         被通知的对象                         |
| 代理(Proxy)       |             向目标对象应用通知之后创建的代理对象             |
| 连接点(JoinPoint) |         目标对象的所属类中，定义的所有方法均为连接点         |
| 切入点(Pointcut)  | 被切面拦截 / 增强的连接点（切入点一定是连接点，连接点不一定是切入点） |
| 通知(Advice)      | 增强的逻辑 / 代码，也即拦截到目标对象的连接点之后要做的事情  |
| 切面(Aspect)      |                切入点(Pointcut)+通知(Advice)                 |
| Weaving(织入)     |       将通知应用到目标对象，进而生成代理对象的过程动作       |

### AspectJ 定义的通知类型有哪些？

- **Before**（前置通知）：目标对象的方法调用之前触发
- **After** （后置通知）：目标对象的方法调用之后触发
- **AfterReturning**（返回通知）：目标对象的方法调用完成，在返回结果值之后触发
- **AfterThrowing**（异常通知）：目标对象的方法运行中抛出 / 触发异常后触发。AfterReturning 和 AfterThrowing 两者互斥。如果方法调用成功无异常，则会有返回值；如果方法抛出了异常，则不会有返回值。
- **Around** （环绕通知）：编程式控制目标对象的方法调用。环绕通知是所有通知类型中可操作范围最大的一种，因为它可以直接拿到目标对象，以及要执行的方法，所以环绕通知可以任意的在目标对象的方法调用前后搞事，甚至不调用目标对象的方法

多个切面顺序通过`@Order`注解控制

## MVC

### Spring MVC 的核心组件有哪些？

记住了下面这些组件，也就记住了 SpringMVC 的工作原理。

- **`DispatcherServlet`**：**核心的中央处理器**，负责接收请求、分发，并给予客户端响应。
- **`HandlerMapping`**：**处理器映射器**，根据 uri 去匹配查找能处理的 `Handler` ，并会将请求涉及到的拦截器和 `Handler` 一起封装。
- **`HandlerAdapter`**：**处理器适配器**，根据 `HandlerMapping` 找到的 `Handler` ，适配执行对应的 `Handler`；
- **`Handler`**：**请求处理器**，处理实际请求的处理器。
- **`ViewResolver`**：**视图解析器**，根据 `Handler` 返回的逻辑视图 / 视图，解析并渲染真正的视图，并传递给 `DispatcherServlet` 响应客户端

![img](https://raw.githubusercontent.com/mervynlam/Pictures/master/202309111751080.png)

**流程说明（重要）：**

1. 客户端（浏览器）发送请求， `DispatcherServlet`拦截请求。
2. `DispatcherServlet` 根据请求信息调用 `HandlerMapping` 。`HandlerMapping` 根据 uri 去匹配查找能处理的 `Handler`（也就是我们平常说的 `Controller` 控制器） ，并会将请求涉及到的拦截器和 `Handler` 一起封装。
3. `DispatcherServlet` 调用 `HandlerAdapter`适配器执行 `Handler` 。
4. `Handler` 完成对用户请求的处理后，会返回一个 `ModelAndView` 对象给`DispatcherServlet`，`ModelAndView` 顾名思义，包含了数据模型以及相应的视图的信息。`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
5. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
6. `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
7. 把 `View` 返回给请求者（浏览器）

## 设计模式

### 工厂模式

通过`Beanfactory`或`ApplicationContext`创建`bean`对象。

### 单例模式

Spring 中 bean 的默认作用域就是 singleton(单例)的。

### 代理模式

`AOP`是基于动态代理

### 适配器模式

`MVC`中`HandlerAdapter`

## 事务

### Spring 管理事务的方式有几种？

- **编程式事务**：通过 `TransactionTemplate`或者 `TransactionManager` 手动管理事务
- **声明式事务**：基于注解： 实际是通过 AOP 实现

### 事务传播行为

- `REQUIRED`：`@Transactional`默认使用的转播行为。如果当前存在事务，则加入事务；如果当前没有事务，则创建一个新事务。
- `REQUIRES_NEW`：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- `NESTED`：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行
- `MANDATORY`：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
- `SUPPORTS`：不会发生回滚。如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- `NOT_SUPPORTS`：不会发生回滚。以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- `NEVER`：不会发生回滚。以非事务方式运行，如果当前存在事务，则抛出异常。

### 事务的隔离级别

- `READ_UNCOMMITTED`：读未提交，可能会发生脏读、幻读、不可重复读
- `READ_COMMITTED`：避免脏读，但可能会发生幻读、不可重复读
- `REPEATABLE_READ`：避免脏读、不可重复读，幻读仍有可能发生
- `SERIALIZABLE`：所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰。但是这将严重影响程序的性能。

### `@Transactional(rollbackFor=Exception.class)`注解

默认只在遇到`RuntimeException`时才会回滚，加上该注解后，可以让事务在遇到非运行时异常时也回滚。

# 参考资料

[JavaGuide - Spring常见面试题总结](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html)

[尚硅谷新版SSM框架全套视频教程](https://www.bilibili.com/video/BV1AP411s7D7)

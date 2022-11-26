# 循环依赖
## 什么是循环依赖
从字面上来理解就是A依赖B的同时B也依赖了A，就像下面这样
![807a0e9cf320c831f9cff20554bafd78.png](https://raw.githubusercontent.com/mervynlam/Pictures/master/202211262309533.png)

体现到代码层次就是这个样子
```java
@Component
public class A {
    // A中注入了B
    @Autowired
    private B b;
}
@Component
public class B {
    // B中也注入了A 
    @Autowired
    private A a;
}
```
或者是自己依赖自己
```java
@Component
public class A {
    @Autowired
    private A a;
}
```
## 什么情况下循环依赖可以被处理
Spring 解决循环依赖的前置条件：
1. 出现循环依赖的Bean是单例的。
2. 依赖注入的方式不能全是构造器注入的方式。

# 参考资料

https://wiki.jikexueyuan.com/project/spring/overview.html

[面试必杀技：讲一讲Spring中的循环依赖](https://mp.weixin.qq.com/s?__biz=MzAwNDA2OTM1Ng==&mid=2453145072&idx=1&sn=5d46d4294e48526a5a7496e9980d4319&chksm=8cfd2773bb8aae65c9687ba3b8d73b9df4e5f8ce90e76c2598d58dd7b2186486a9fa88e35abc&scene=126&sessionid=1596502525&key=38f9a66cdb126ea01857d9bff4e1b7c2c18f70429cc9ab42124205ee69454c7610c1c725a0041c3d96918ba15c1fbc9a50c59be9b5c390116d071cfdb050930bcdf122a35df761b6c6cd8463b369790d&ascene=1&uin=OTgxMDI0NzIw&devicetype=Windows+10+x64&version=62090529&lang=zh_CN&exportkey=A6XUfdfbz%2BbxWjX1ltKlIyU%3D&pass_ticket=7HRfTeYriXa1Lz5Fb9SN2l1xQtTKR%2B%2Bs121ab2F4%2BqGnf2NDIsptweV%2FBOFdxHJf)

https://mp.weixin.qq.com/s/5mwkgJB7GyLdKDgzijyvXw
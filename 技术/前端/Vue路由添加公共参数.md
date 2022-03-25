# Vue路由添加公共参数

如[Vue-Router](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html)介绍所说，`router.beforeEach`是路由跳转前的钩子，需要增加公共参数可以在这里处理。

>  In that case, **you must call `next` exactly once** in any given pass through a navigation guard.

`router.beforeEach`中第三个参数`next`，必须在每一种给定的导航中严格执行一次。

执行分两种情况

- `next()`，这种情况不再执行前置钩子
- `next('/')`，这种情况会执行前置钩子

所以钩子中必须包含`next()`，否则将进入死循环。

**添加公共参数**

```js
router.beforeEach(to, from, next){
    //to是即将跳转的路由
    //from是跳转前的路由
    //公共参数以commonId为例
    if (!to.query.commonId) {
        //如果目标路由没有公共参数，就获取公共参数并添加
        let commonId = from.query.commonId 
        if (commonId) {
            //如果公共参数不为空，则塞进目标路由中
            to.query.commonId = commonId
            next({
                to: to.name,
                query: to.query
            })
        } else {
            next()
        }
   	} else {
        next()
    }
}
```


# 使用`beforeRouteUpdate`和`beforeRouteLeave`解决路由前置钩子需要与实例通信的问题

## 前景提要

近期项目有个需求：为了增加用户粘性，增加菜单要可配置成需要登陆后才允许访问的功能。

最初的想法：在路由前置钩子中，判断目标路由是否需要登陆，不需要就直接跳转，否则弹出登录对话框。

然后问题来了：当初写登录的时候，没有写成一个组件，仅仅是常驻组件`header`中的一个对话框，通过一个布尔`loginFlag`判断是否打开对话框。然而，在全局前置钩子中无法访问组件实例`this`，无法通过事件总线与对话框取得通信，也无法通过`route`跳转到登录页。

## 解决过程

经过查找资料、与朋友探讨和学习，了解到[“组件内守卫”](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html#%E5%8F%AF%E7%94%A8%E7%9A%84%E9%85%8D%E7%BD%AE-api)，

```js
  beforeRouteEnter(to, from) {
    // 在渲染该组件的对应路由被验证前调用
    // 不能获取组件实例 `this` ！
    // 因为当守卫执行时，组件实例还没被创建！
  },
  beforeRouteUpdate(to, from) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 `/users/:id`，在 `/users/1` 和 `/users/2` 之间跳转的时候，
    // 由于会渲染同样的 `UserDetails` 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 因为在这种情况发生的时候，组件已经挂载好了，导航守卫可以访问组件实例 `this`
  },
  beforeRouteLeave(to, from) {
    // 在导航离开渲染该组件的对应路由时调用
    // 与 `beforeRouteUpdate` 一样，它可以访问组件实例 `this`
  },
```

由于项目的登录对话框是写在`header`中的，看起来很符合`beforeRouteUpdate`的使用场景，无论路由如何改变，`header`都是复用的，并且还可以访问组件实例`this`，岂不是可以直接操作`loginFlag`来打开登录对话框了。

```js
beforeRouteUpdate(to, from, next) {
    let token = getToken()
    if (to.meta.needLogin && !token) {
        //打开登录对话框
        this.loginFlag = true
    } else {
        next()
    }
},
```

## 可能存在的问题

`beforeRouteUpdate`和`beforeRouteLeave`两个钩子放在子组件中可能不生效。需要写在父组件中才生效。

## 参考资料

[组件内守卫](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html#%E7%BB%84%E4%BB%B6%E5%86%85%E7%9A%84%E5%AE%88%E5%8D%AB)
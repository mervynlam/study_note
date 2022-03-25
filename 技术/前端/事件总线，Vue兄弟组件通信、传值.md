# 事件总线，Vue兄弟组件通信、传值

## 父子组件

在`Vue`中，父子组件通信、传值是比较方便的。

通过`$emit`和`@事件名`即可完成通信

```vue
<!-- 子组件 -->
<template>
	<button @click="sendToParent">
        向父组件传值
    </button>
</template>
<script>
export default{
    name: 'son',
    data(){
        return {
            msg:'子组件信息'
        }
    },
    methods: {
        sendMsg() {
            this.$emit("sendMsg", this.msg)	//子组件传值
        }
    }
}
</script>
```

```vue
<!-- 父组件 -->
<template>
	<son @sendMsg="getMsg"></son>
</template>
<script>
import son from "./son"
export default{
    name: 'parent',
    components:{
        son
    }
    data(){
        return {
        }
    },
    methods: {
        getMsg(msg) {
			alert(msg)//父组件获取到值
        }
    }
}
</script>
```

## 兄弟组件

但是在兄弟组件中，没有这种直接引用关系。通信就需要借助“事件总线”来完成。

在`main.js`中创建一个的“事件总线”

```javascript
Vue.prototype.$eventBus = new Vue()
```

此时在任一组件中，可以直接`this.$eventBus`引用事件总线。

创建后事件总线后，发送和接收事件的方法就类似父子组件了。

```vue
<!-- a组件发送消息 -->
<template>
	<button @click="sendMsg">
		发送信息
    </button>
</template>
<script>
export default{
    name: 'a',
    data(){
        return {
            msg:'a组件信息'
        }
    },
    methods: {
        sendMsg() {
            this.$eventBus.$emit("sendMsg",this.msg)
        }
    }
}
</script>
```

```vue
<!-- b组件接收消息 -->
<template>
	<!-- 页面内容 -->
</template>
<script>
export default{
    name: 'b',
    data(){
        return {
        }
    },
    mounted() {
        //接收信息
        this.$eventBus.$on("sendMsg", (msg) => {
            alert(msg)
        })
    }
}
</script>
```


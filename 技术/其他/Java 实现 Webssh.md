基于[NoCortY](https://github.com/NoCortY)的项目[WebSSH](https://github.com/NoCortY/WebSSH)修改：

1. 升级xterm到4.x
2. 自适应宽高
3. 心跳检查
4. 断开自动重连

NoCortY的博客：[使用纯Java实现一个WebSSH项目](https://blog.objectspace.cn/2020/03/10/使用纯Java实现一个WebSSH项目/)

篇幅限制，这里只放出效果及关键代码。完整代码见 [mervynlam/Webssh-Java - Github](https://github.com/mervynlam/Webssh-Java)

## 效果

### 自适应宽高

![autoFit](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200925100444.gif)

### 心跳检查

![heartbeat](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200925101005.gif)

### 断开自动重连

![reconnect](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200925101148.gif)

## 关键代码

### 自适应宽高

**前端**

```javascript
//terminal 大小改变
function resizeTerminal() {
    //默认字体大小的宽高和行列计算比例
    var c = parseInt($("#outerDiv").width() / 9);
    var r = parseInt($("#outerDiv").height() / 17);
    //前端调用xterm的resize方法
    term.resize(c, r);
    //调整terminal的大小后，需要把行列传给后台
    client.send({
        "operate": "command"
        , "command": ''
        , "cols": c
        , "rows": r
    });
};
```

```javascript
WSSHClient.prototype.send = function (data) {
    this._connection.send(JSON.stringify(data));
};
```

**后台**

```java
//实体类添加行列宽高字段
private int cols = 80;
private int rows = 24;
private int width = 640;
private int height = 480;
```

```java
//实现类添加调整channel行列宽高的代码
channel.setPtySize(webSSHData.getCols(),webSSHData.getRows(),webSSHData.getWidth(),webSSHData.getHeight());
```

前后端都需要调整，否则会出现输入很长的命令会把前面的文字覆盖的问题。

### 心跳检查

使用websocket的过程中，有时会出现断开连接但没有触发`onclose`事件。这就需要做心跳检查，客户端和服务端相互告知还“活着”。

**实现逻辑**

1. 客户端定时向服务端发送数据，告知服务端自己还活着。
2. 服务端收到数据，如果连接没有断开，则返回一条信息给客户端，告诉客户端自己还活着。
3. 如果客户端在一定时间内没收到服务端返回的“活着”的信息，则判为已断开连接，触发`onclose`事件。

**实现代码**

客户端

```javascript
WSSHClient.prototype.connect = function (options) {
    var endpoint = this._generateEndpoint();

    if (window.WebSocket) {
        //如果支持websocket
        this._connection = new WebSocket(endpoint);
    }else {
        //否则报错
        options.onError('WebSocket Not Supported');
        return;
    }

    this._connection.onopen = function () {
        options.onConnect();
        //开始连接，启动心跳检查
        heartCheck.start();
    };

    this._connection.onmessage = function (evt) {
        var data = evt.data.toString();
        //如果是返回心跳，不执行onData();方法
        if (data !== "Heartbeat healthy") {
            options.onData(data);
        }
        //收到消息，重置心跳检查
        heartCheck.start();
    };

    this._connection.onclose = function (evt) {
        options.onClose();
    };
};

WSSHClient.prototype.send = function (data) {
    this._connection.send(JSON.stringify(data));
};

WSSHClient.prototype.sendInitData = function (options) {
    //连接参数
    this._connection.send(JSON.stringify(options));
}

//关闭连接
WSSHClient.prototype.close = function () {
    this._connection.close();
}

var client = new WSSHClient();

//心跳检查
var heartCheck = {
    checkTimeout: 5000,//心跳检查时间
    closeTimeout: 2000,//无心跳超时时间
    checkTimeoutObj: null,//心跳检查定时器
    closeTimeoutObj: null,//无心跳关闭定时器
    start: function () {
        //清除定时器
        clearTimeout(this.checkTimeoutObj);
        clearTimeout(this.closeTimeoutObj);

        // console.log("检查心跳");
        var _this = this;

        this.checkTimeoutObj = setTimeout(function () {
            //向服务端发送心跳包
            client.send({operate: "heartbeat"});
            _this.closeTimeoutObj = setTimeout(function () {
                //超时未收到服务端返回的心跳消息，则断开连接
                console.log("无心跳，关闭连接");
                client.close();
            }, _this.closeTimeout);
        }, this.checkTimeout);
    }
}
```

服务端

```java
//处于连接状态则发送健康数据，不能为空，空则断开连接。
if (channel.isConnected())
    sendMessage(session, "Heartbeat healthy".getBytes());
```

### 断开后自动重连

断开连接会触发`onclose`时间，所以我们可以在`onclose`中实现重连。

```javascript
WSSHClient.prototype.connect = function (options) {
	/*
	...
	重复代码省略
	*/
    this._connection.onmessage = function (evt) {
        var data = evt.data.toString();
        //如果是返回心跳，不执行onData();方法
        if (data !== "Heartbeat healthy") {
            options.onData(data);
        } else {
            //心跳健康，重置重连次数
            reconnectTimes = 0;
        }
        //收到消息，重置心跳检查
        heartCheck.start();
    };

    this._connection.onclose = function (evt) {
        options.onClose();
        //关闭后重连
        reconnect(options);
    };
};


//重新连接
var lockReconnect = false;//重连锁，避免重复连接
var reconnectTimes = 0;//重连次数
var maxReconnectTimes = 6;//最大允许重连次数
function reconnect(options) {
    if (lockReconnect)
        return;

    // console.log("重新连接");

    //超过次数不重启
    if (reconnectTimes >= maxReconnectTimes) {
        alert("超过" + maxReconnectTimes + "次重连失败。");
        return;
    }

    setTimeout(function() {
        //重新连接
        client.connect(options);
        lockReconnect = false;
    }, 500);
}
```

以上就是自适应、心跳、重连的关键代码。

总结：

1. 自适应宽高需要前后端同时调整。
2. 心跳检查逻辑：
   1. 客户端发送心跳包到服务端。
   2. 服务端接收到后返回健康消息给客户端。
   3. 客户端如果在时间范围没收到消息则视为断开连接。
3. 断开重连关键：
   1. `onclose`中实现重连。
   2. 如果需要限制重连次数，则客户端在收到服务端发来的健康信息时，需要重置当前已重连次数。

## 参考资料

[理解WebSocket心跳及重连机制（五）](https://www.cnblogs.com/tugenhua0707/p/8648044.html)
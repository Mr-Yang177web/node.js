现在，我们有了WebSocket，它是HTML5的新API。WebSocket连接本质上就是建立一个TCP连接，WebSocket会通过HTTP请求建立，建立后的WebSocket会在客户端和服务端建立一个持久的连接，直到有一方主动关闭该连接。

socket.io提供了基于事件的实时双向(工)通讯，它同时提供了服务端和客户端的API

运用场景
1、实时分析：将数据推送到客户端，客户端表现为实时计数器、图表、日志客户。
2、实时通讯：聊天应用
3、二进制流传输：socket.io支持任何形式的二进制文件传输，例如图片、视频、音频等。
4、文档合并：允许多个用户同时编辑一个文档，并能够看到每个用户做出的修改

第一步: 下载依赖
```js
npm install -D koa koa-static socket.io
```

第二步:创建应用程序及挂载静态资源处理中间件

```js
const Koa = require('koa');
const koaStatic = require('koa-static');
const app = new Koa();

app.use(koaStatic('./static'));

```

第三步:将应用程序服务于socket.io连接
>app.callback(): 返回适用于 http.createServer() 方法的回调函数来处理请求
>http.Server: http.Server 类
```js
const server = require('http').Server(app.callback());
server.listen(3000, () => {console.log(1)})

// 连接
const io = require('socket.io')(server);

```

第四步：io监听连接连接状态
监听连接，返回的回调函数中接收socket的实例
```js
// 监听连接
io.on('connect', (socket) => {
    console.log('连接');
    // 监听信号（xinhao）
    socket.on('xinhao', (msg) => {
        console.log(msg);
        // 回复信号
        socket.emit('xinhao2', '第二个信号');
    });
});
```

客户端
```js
let socket = io();
// 发送信号： 信号标识符   信息
socket.emit('xinhao','这是第一个信号');
// 监听信号，xinhao2, 回调函数里：接收: 信息
socket.on('xinhao2', (msg) => {
    let p = document.createElement('p');
    p.innerHTML = msg;
    content.appendChild(p);
});
```



实例：
1、客户端
```js
let socket = io();
    socket.on('szc', (msg) => {
        console.log(msg);
    });

    socket.emit('jjj', '你好');

    socket.on('allperson', (obj) => {
        let li = document.createElement('li');

        li.innerHTML = obj.name + ':' + obj.msg;

        ul.appendChild(li);
    })


    btn.onclick = function () {
        socket.emit('all', {
            name: '访客' + Math.floor(Math.random() * 1000),
            msg: txt.value
        });
        txt.value = '';
    }
```

2. 服务端
```js
const Koa = require('koa');

const koaStatic = require('koa-static');

const app = new Koa();

app.use(koaStatic('./static'));

// app.listen(3000)

// koa的应用给了http的服务
const server = require('http').Server(app.callback());
server.listen(3000);

// 将服务于io联系在一起
const io = require('socket.io')(server);

// 监听连接
io.on('connection', function (socket) {

    // 发送消息
    // 第一个参数是：信号
    // 第二个参数: 内容
    socket.emit('szc', '您好');

    socket.on('jjj', (msg) => {
        console.log(msg);
    });


    socket.on('all', (obj) => {
        socket.broadcast.emit('allperson', obj);
    });
});
```
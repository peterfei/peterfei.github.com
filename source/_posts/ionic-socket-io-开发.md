title: ionic socket.io 开发
date: 2015-07-09 00:48:44
tags:
- ionic&socket.io
- ionic
---
*这两天一直在做*ionic*开发，期间有个诉求是要完成ionic与socket.io对接，中间绕了不少弯子，故留下笔记。*

*先叙述下基本的诉求：*

* Nodejs 做为服务端，主要做服务端消息的监听与发送
* 客户端App 为服务端发送消息
* 主要协议TCP

服务端代码如下：
```javascript
var http= require('http'), io= require('socket.io');
var app = http.createServer(), io = io.listen(app);


app.listen(10010);

io.sockets.on('connection', function (socket) {
  socket.emit('news', { hello: 'world' });//监听，一旦客户端连接上，即发送数据，第一个参数'new'为数据名，第二个参数既为数据
  socket.on('my new event', function (data) {//捕获客户端发送名为'my other event'的数据
    console.log(data.my);
  });

  socket.emit('other', { hello: 'other world' });//发送另一个数据
  socket.on('event1', function (data) {//捕获另外一个数据
    console.log(data);
  });
});

```

*Ionic 部分：*

* 生成新项目：`ionic start chat-app blank`
* 引入angular-socket-io 库:` bower install angular-socket-io`
* ![folder](http://pic.yupoo.com/peterfei/EMOPKcas/Veffo.png)
* 注入socket.io在项目里：`var app=angular.module('ionic-socketio-chat-client', ['ionic','btford.socket-io'])`
* 在index.html导入socket.io，**注：socket.io/socket.io.js是服务端动态生成**
	
	 	<script src="http://192.168.0.101:10010/socket.io/socket.io.js"></script>
		<script src="lib/angular-socket-io/socket.js"></script>
	
	```javascript
		var socket = io.connect('http://192.168.0.101:10010');
        // io.set('origins', '*:*');
        socket.on('news', function(data) {
            console.log(data.hello);
        });
        socket.on('other', function(data) { //接收另一个名为'other'数据，  
            console.log(data.hello);
            socket.emit('event1', {
                my: 'other data'
            });
        });
	
	```
	
如此，客户端调用成功。
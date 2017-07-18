# socker.io

## 开始使用：聊天应用程序
在本指南中，我们将创建一个基本的聊天应用程序。它几乎不需要Node.JS或Socket.IO的基本的先验知识，因此它是所有知识层面的用户的理想选择。

## 介绍

使用流行的Web应用程序堆栈（如LAMP（PHP））编写聊天应用程序传统上是非常困难的。它涉及轮询服务器进行更改，跟踪时间戳，并且比它应该慢得多。

传统上，套接字是大多数实时聊天系统被构建的解决方案，在客户端和服务器之间提供双向通信通道。

这意味着服务器可以将消息推送给客户端。每当你写一个聊天消息时，这个想法是服务器将获取它并将其推送到所有其他连接的客户端。

## 网络框架
第一个目标是设置一个简单的HTML网页，提供表单和消息列表。为此，我们将使用Node.JS Web框架express。确保Node.js的安装。

首先我们来创建一个*package.json*描述我们项目的清单文件。我建议你将它放在一个专用的空目录中。

```
{
  "name": "socket-chat-example",
  "version": "0.0.1",
  "description": "my first socket.io app",
  "dependencies": {}
}
```
    
现在，为了轻松地填充dependencies我们需要的东西，我们将使用`npm install --save`：

`npm install --save express@4.15.2`

现在安装了express，我们可以创建一个*index.js*将设置我们的应用程序的文件。

```javascript
var app = require('express')();
var http = require('http').Server(app);

app.get('/', function(req, res){
  res.send('<h1>Hello world</h1>');
});

http.listen(3000, function(){
  console.log('listening on *:3000');
});
```

这转化为以下内容：

Express初始化app为可以提供给HTTP服务器的函数处理程序（如第2行所示）。
我们定义一个路由处理程序/，当我们在我们的网站回到家时被调用。
我们使http服务器在端口3000上侦听。

如果您将浏览器指向[http://localhost:3000](http://localhost:3000)

## 提供HTML

到目前为止，*index.js*我们正在调用`res.send`并传递一个HTML字符串。

如果我们把整个应用程序的HTML放在那里，我们的代码会变得很混乱。相反，我们将创建一个*index.html*文件并提供服务。

让我们重构我们的路由处理程序来sendFile改用：

```javascipt
app.get('/', function(req, res){
res.sendFile(__dirname + '/index.html');
});
```

并填充*index.html*以下内容：

```html
<!doctype html>
<html>
  <head>
    <title>Socket.IO chat</title>
    <style>
      * { margin: 0; padding: 0; box-sizing: border-box; }
      body { font: 13px Helvetica, Arial; }
      form { background: #000; padding: 3px; position: fixed; bottom: 0; width: 100%; }
      form input { border: 0; padding: 10px; width: 90%; margin-right: .5%; }
      form button { width: 9%; background: rgb(130, 224, 255); border: none; padding: 10px; }
      #messages { list-style-type: none; margin: 0; padding: 0; }
      #messages li { padding: 5px 10px; }
      #messages li:nth-child(odd) { background: #eee; }
    </style>
  </head>
  <body>
    <ul id="messages"></ul>
    <form action="">
      <input id="m" autocomplete="off" /><button>Send</button>
    </form>
  </body>
</html>
```

## 集成Socket.IO

Socket.IO由两部分组成：

- 与Node.JS HTTP Server集成（或挂接）的服务器：`socket.io`
- 在浏览器端加载的客户端库： socket.io-client
在开发期间，socket.io为我们自动为我们提供服务，所以现在我们只需要安装一个模块：

`npm install --save socket.io`

这将安装模块并添加依赖关系*package.json*。现在我们来编辑*index.js*来添加它：

```javascript
var app = require('express')();
var http = require('http').Server(app);
var io = require('socket.io')(http);

app.get('/', function(req, res){
  res.sendFile(__dirname + '/index.html');
});

io.on('connection', function(socket){
  console.log('a user connected');
});

http.listen(3000, function(){
  console.log('listening on *:3000');
});
```

请注意，socket.io通过传递http（HTTP服务器）对象来初始化一个新的实例。然后收听有关connection入站套接字的事件，并将其登录到控制台。

现在在*index.html*添加以下代码片段之前`</body>`：

```html
<script src="/socket.io/socket.io.js"></script>
<script>
  var socket = io();
</script>
```

这就是加载它所需要的`socket.io-client`，它暴露了一个io全局，然后连接。

请注意，当我调用 io() 时，我没有指定任何URL，因为它默认尝试连接到提供该页面的主机。

如果您现在重新加载服务器和网站，您应该看到控制台打印`"a user connected"`。

尝试打开几个选项卡，您会看到几个消息：

每个套接字还触发一个特殊disconnect事件：

```javascript
io.on('connection', function(socket){
  console.log('a user connected');
  socket.on('disconnect', function(){
    console.log('user disconnected');
  });
});
```

## Emitting events

Socket.IO背后的主要思想是您可以发送和接收任何您想要的事件，以及任何您想要的数据。

任何可以编码为JSON的对象都可以进行编码，二进制数据也被支持。

这样当用户键入消息时，服务器将其当作`chat message`事件。

现在*index.html*的script部分现在看起来如下：

```html
<script src="/socket.io/socket.io.js"></script>
<script src="https://code.jquery.com/jquery-1.11.1.js"></script>
<script>
  $(function () {
    var socket = io();
    $('form').submit(function(){
      socket.emit('chat message', $('#m').val());
      $('#m').val('');
      return false;
    });
  });
</script>
```

在*index.js*我们打印出的`chat message`事件：

```javascript
io.on('connection', function(socket){
  socket.on('chat message', function(msg){
    console.log('message: ' + msg);
  });
});
```

## Broadcasting
下一个目标是让我们从服务器发送到其他用户的事件。

为了向大家发送一个事件，Socket.IO给了我们`io.emit`：

`io.emit('some event', { for: 'everyone' });`

如果你想发送消息给所有人，除了某个套接字，我们有broadcast标志：

```javascript
io.on('connection', function(socket){
  socket.broadcast.emit('hi');
});
```

在这种情况下，为了简单起见，我们会将消息发送给所有人，包括发件人。

```javascript
io.on('connection', function(socket){
  socket.on('chat message', function(msg){
    io.emit('chat message', msg);
  });
});
```

而在客户端，当我们捕获一个`chat message`事件时，我们将其包含在页面中。客户端所有的JavaScript代码如下：

```html
<script>
  $(function () {
    var socket = io();
    $('form').submit(function(){
      socket.emit('chat message', $('#m').val());
      $('#m').val('');
      return false;
    });
    socket.on('chat message', function(msg){
      $('#messages').append($('<li>').text(msg));
    });
  });
</script>
```

这就完成了我们的聊天应用程序，大约20行代码！
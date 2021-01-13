## WebSocket 详解

### 一. WebSocket的通信原理和机制

#### 1. 客户端：申请协议升级
websocket本身虽然也是一种新的应用层协议，但是它也不能够脱离http而单独存在。具体来讲，我们在客户端构建一个websocket实例，并且为它绑定一个需要连接到的服务器地址，当客户端连接服务端的时候，会向服务端发送一个类似下面的http报文：

```sh
GET /webfin/websocket/ HTTP/1.1
Host: localhost
Connection: Upgrade # 表示要升级协议
Upgrade: websocket # 表示要升级到 websocket 协议
# Sec-WebSocket-Key 的值是随机生成的 Base64 编码的字符串，用于安全校验
Sec-WebSocket-Key: xqBt3ImNzJbYqRINxEFlkg==
Origin: http://localhost:8080
Sec-WebSocket-Version: 13 # 表示 websocket 的版本
```

这是一个 http/get 请求报文，该报文中有一个 upgrade 首部，它的作用是告诉服务端需要将通信协议切换到 websocket，如果服务端支持 websocket 协议，那么它就会将自己的通信协议切换到 websocket，同时发给客户端类似于以下的一个响应报文头：

#### 2. 服务端：响应协议升级

```sh
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
# Sec-WebSocket-Accept 的值是服务端采用与客户端一致的密钥计算出来后返回客户端的
Sec-WebSocket-Accept: K7DJLdLooIwIG/MOpvWFB3y3FE8=

# 备注：每个 header 都以 \r\n 结尾，并且最后一行加上一个额外的空行 \r\n
```

返回的状态码为`101`，表示同意客户端协议转换请求，并将它转换为 websocket 协议。以上过程都是利用 http 通信完成的，称之为 websocket 协议握手(websocket Protocol handshake)，进过这握手之后，客户端和服务端就建立了 websocket 连接，以后的通信走的都是 websocket 协议了。所以总结为 websocket 握手需要借助于 http 协议，建立连接后通信过程使用 websocket 协议。同时需要了解的是，该 websocket 连接还是基于我们刚才发起 http 连接的那个 TCP 连接。

一旦建立连接之后，就可以进行数据传输了，websocket 提供两种数据传输：文本数据和二进制数据。

#### 3. 数据帧格式

详解有点绕，反正记不住，用到再查！

```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-------+-+-------------+-------------------------------+
|F|R|R|R| opcode|M| Payload len |    Extended payload length    |
|I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
|N|V|V|V|       |S|             |   (if payload len==126/127)   |
| |1|2|3|       |K|             |                               |
+-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
|     Extended payload length continued, if payload len == 127  |
+ - - - - - - - - - - - - - - - +-------------------------------+
|                               |Masking-key, if MASK set to 1  |
+-------------------------------+-------------------------------+
| Masking-key (continued)       |          Payload Data         |
+-------------------------------- - - - - - - - - - - - - - - - +
:                     Payload Data continued ...                :
+ - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
|                     Payload Data continued ...                |
+---------------------------------------------------------------+
```

#### 4. 连接保持 + 心跳

有些场景，客户端、服务端虽然长时间没有数据往来，但仍需要保持连接。这个时候，可以采用心跳来确认连接是否断开。
* 发送方 -> 接收方：ping
* 接收方 -> 发送方：pong

`ping`、`pong`的操作，对应的是WebSocket的两个控制帧，`opcode`分别是`0x9`、`0xA`

#### 5. 纯原生实现
**客户端代码：**
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>document</title>
</head>

<body>
</body>
<script>
    let ws = new WebSocket('ws://localhost:12010')
    ws.onopen = function (e) {
        ws.send('你好 PercyMo！');
    }
    ws.onerror = function (e) {
    }
    ws.onmessage = function (e) {
        console.log('onmessage', e);
    }
    ws.onclose = function (e) {
    }
</script>

</html>
```

**服务端代码：**
```js
var http = require('http');
var crypto = require('crypto');

var server = http.createServer(function (req, res) {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('hello world\n');
});
server.listen(12010);

server.on('upgrade', function (req, socket, head) {
    var GUID = '258EAFA5-E914-47DA-95CA-C5AB0DC85B11';
    var key = req.headers['sec-websocket-key'];
    // 获取客户端返回的 key 与 GUID 进行 sha1 编码后获取 base64 格式
    key = crypto.createHash('sha1').update(key + GUID).digest('base64');

    // 返回 101 协议切换响应
    var headers = [
        'HTTP/1.1 101 Switching Protocols',
        'Upgrade: websocket',
        'Connection: Upgrade',
        'Sec-WebSocket-Accept: ' + key
    ];
    socket.setNoDelay(true);
    socket.write(headers.concat('', '').join('\r\n'));

    // 数据传输
    socket.on('data', data => {
        // data: <Buffer 81 90 a2 d8 42 4e 46 65 e2 ab 07 65 62 1e cd ab 3b 03 cd 37 fe cf>
        console.log(decodeWsFrame(data))
        setTimeout(() => {
            // 编码数据帧后，发送消息到客户端
            socket.write(encodeData(decodeWsFrame(data)));
        }, 3000);
    })
});

// 解码ws数据帧
function decodeWsFrame(e) {
    var i = 0, j, s, frame = {
        // 解析前两个字节的基本数据
        FIN: e[i] >> 7,
        Opcode: e[i++] & 15,
        Mask: e[i] >> 7,
        PayloadLength: e[i++] & 0x7F
    }
    // 处理特殊长度126和127
    if (frame.PayloadLength == 126) {
        frame.length = (e[i++] << 8) + e[i++]
    }
    if (frame.PayloadLength == 127) {
        i += 4,
            // 长度一般用四字节的整型，前四个字节通常为长整形留空的
            frame.length = (e[i++] << 24) + (e[i++] << 16) + (e[i++] << 8) + e[i++]
    }
    // 判断是否使用掩码
    if (frame.Mask) {
        // 获取掩码实体
        frame.MaskingKey = [e[i++], e[i++], e[i++], e[i++]]
        // 对数据和掩码做异或运算
        for (j = 0, s = []; j < frame.PayloadLength; j++) {
            s.push(e[i + j] ^ frame.MaskingKey[j % 4])
        }
    } else {
        s = e.slice(i, frame.PayloadLength) // 否则直接使用数据
    }
    // 数组转换成缓冲区来使用
    s = new Buffer(s)
    // 如果有必要则把缓冲区转换成字符串来使用
    if (frame.Opcode == 1) {
        s = s.toString()
    }
    // 设置上数据部分
    frame.PayloadData = s
    // 返回数据帧
    return frame
}

// 编码ws数据帧
function encodeData(e) {
    var s = [], o = new Buffer(e.PayloadData), l = o.length;
    // 输入第一个字节
    s.push((e.FIN << 7) + e.Opcode);
    // 输入第二个字节，判断它的长度并放入相应的后续长度消息
    // 永远不使用掩码
    if (l < 126) s.push(l);
    else if (l < 0x10000) s.push(126, (l & 0xFF00) >> 2, l & 0xFF);
    else s.push(
        127, 0, 0, 0, 0, //8字节数据，前4字节一般没用留空
        (l & 0xFF000000) >> 6, (l & 0xFF0000) >> 4, (l & 0xFF00) >> 2, l & 0xFF
    );
    //返回头部分和数据部分的合并缓冲区
    return Buffer.concat([new Buffer(s), o]);
}
```

参考： 
[* WebSocket详解系列（IM开发者社区）](http://www.52im.net/forum.php?mod=viewthread&tid=338)

[WebSocket协议：5分钟从入门到精通](https://www.cnblogs.com/chyingp/p/websocket-deep-in.html)

### 二. 基于WebSocket的实现方案

#### 1. 使用ws模块搭建的简单服务器
```html
<!-- 客户端代码 -->
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>document</title>
</head>

<body>
</body>
<script>
    let ws = new WebSocket('ws://localhost:8080')
    ws.onopen = function (e) {
        ws.send('hello PercyMo!')
    }
    ws.onerror = function (e) {
    }
    ws.onmessage = function (e) {
        console.log('onmessage', e);
    }
    ws.onclose = function (e) {
    }
</script>

</html>
```

```js
// 服务端代码
var WebSocketServer = require('ws').Server;
var wss = new WebSocketServer({ port: 8080 });

wss.on('connection', function connection(ws) {
    ws.on('message', function incoming(message) {
        console.log('received: %s', message);
    });

    ws.send('something');
});
```
        
#### 2. Socket.IO
Socket.IO 是一个封装了 Websocket、基于 Node 的 JavaScript 框架，包含 client 的 JavaScript 和 server 的 Node。其屏蔽了所有底层细节，让顶层调用非常简单。

另外，Socket.IO 还有一个非常重要的好处。其不仅支持 WebSocket，还支持许多种轮询机制以及其他实时通信方式，并封装了通用的接口。这些方式包含 Adobe Flash Socket、Ajax 长轮询、Ajax multipart streaming 、持久 Iframe、JSONP 轮询等。换句话说，当 Socket.IO 检测到当前环境不支持 WebSocket 时，能够自动地选择最佳的方式来实现网络的实时通信。

```html
<!-- 客户端实现 index.html -->
<!doctype html>
<html>

<head>
    <title>Socket.IO chat</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box;}
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
    <script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/2.0.3/socket.io.js"></script>
    <script src="http://code.jquery.com/jquery-1.11.1.js"></script>
    <script>
        var socket = io();
        $('form').submit(function () {
            // io.emit提供给我们可以发送给所有人的事件io.emit('some event', { for: 'everyone' });
            socket.emit('chat message', $('#m').val());
            $('#m').val('');
            return false;
        });
        socket.on('chat message', function (msg) {
            $('#messages').append($('<li>').text(msg));
        });
    </script>
</body>

</html>
```

```js
// 服务端实现 index.js
var app = require('express')();
var http = require('http').Server(app);
var io = require('socket.io')(http);

app.get('/', function (req, res) {
    res.sendfile(__dirname + '/index.html');
});

io.on('connection', function (socket) {
    console.log('a user connected', socket.id);
    // 监听客户端的消息
    socket.on('chat message', function (msg) {
        // 用于将消息发送给每个人，包括发送者
        io.emit('chat message', msg);
    });
    socket.on('disconnect', function () {
        console.log('user disconnected');
    });
});

http.listen(3000);
```
        
### 三. 利用 socket 实现消息实时推送思路
1. 在 Node 服务器建立一个用户信息和 socket id 的映射表，因为同一用户可能打开了多个页面，所以他的 socket id 可能存在多个值。当用户建立连接时，往其中添加值；用户断开连接后，删除相应值。

    ```js
    socket.on('user_login', function(info) {
        const { tokenId, userId, socketId } = info;
        addSocketId(users, { tokenId, socketId, userId });
    });
    ```
2. 根据 tokenId 找出与该用户对应的 socket id。根据 id 来向特定用户推送消息。

    ```js
    // 只向 id = socketId 的这一连接发送消息 
    io.sockets.to(socketId).emit('receive_message', {
        entityType,
        data
    });
    ```
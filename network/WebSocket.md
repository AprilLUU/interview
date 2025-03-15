## WebSocket

WebSocket建立在http之上，经过http一次握手之后，协议切换到WebSocket

- 客户端发送http切换协议报文给服务端
- 服务端返回101响应报文，允许切换协议
- 双方切换协议进行通信

#### 帧格式

- Fin：标识消息结束
- Opcode： 表示帧格式，文本或二进制
- Mask：掩码加密，客户端到服务端必须掩码加密
- Payload Length：数据长度
- Payload Data

#### 心跳机制

在onopen事件监听器中启动定时器，定时向服务端发送空数据包，确保通信双方仍存在

```javascript
// 客户端心跳
const heartbeatInterval = 30000; // 30秒
let heartbeatTimer;

socket.onopen = () => {
  heartbeatTimer = setInterval(() => {
    if (socket.readyState === WebSocket.OPEN) {
      socket.send('ping');
    }
  }, heartbeatInterval);
};

socket.onclose = () => {
  clearInterval(heartbeatTimer);
};
```

#### 断线重连

在onclose事件监听器中尝试重新连接

```javascript
let retries = 0;
const maxRetries = 5;

function connect() {
  const socket = new WebSocket('wss://example.com/socket');
  // ...绑定事件
  socket.onclose = () => {
    if (retries < maxRetries) {
      setTimeout(() => {
        connect();
        retries++;
      }, 1000 * Math.pow(2, retries)); // 指数退避
    }
  };
}
```


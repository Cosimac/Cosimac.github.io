---
title: 服务端消息推送方式
date: 2022-08-14 14:17:54
tags: web
---

## 服务端向web前端推送消息常用方式

### 问题出现场景
+ 面试官向你提问，说说如何实现服务端向web前端推送数据？😂
+ 实际开发中当前端页面在挂起状态，这时候服务端推送了一条消息需要前端展示（单向推送）
+ web页面实时聊天功能（双向推送）
  
### 解决方案
主要写了5种方案，分别介绍下实现以及运用场景
> 1. 轮询（短轮询）
这是最简单直观的方法，就是每隔一段时间发起一个请求到后端询问是否有新信息。至于为什么又叫短轮询，其是相对于后续要说的长轮询来对比的。

前端写法只要设置一个setTimeout来定时请求就行
```javascript
// 缓存前端已经获取的最新id
let id = 0;
function poll() {
  $.ajax({
    url: '/api/polling',
    data: { id },
  }).done(res => {
    id += res.length;
  }).always(() => {
    // 10s后再次请求
    setTimeout(poll, 10000);
  });
}
poll();
```
后端写法，根据前端给到的id，看看有没有新消息，有就返回，没有就返回空
```javascript
const id = parseInt(query.id || '0', 10) || 0;
res.writeHead(200, { 'Content-Type': 'application/json;' });
res.end(JSON.stringify(datas.slice(id)));
```
这种方式不需要后端实现消息推送改造，只需要前端定时请求后端接口其实时性与请求频率成正相关，但是当请求频率上来了，性能浪费也就越高，毕竟可能大部分请求都是无意义的。

> 2. 长轮询
这里的长轮询相对前面的轮询来说，算是一种优化。具体就是前端发起请求到后端，后端不直接返回，而是等待有新信息时再返回。所以这样发起的一个请求，可能需要很长的时间才能等到返回，故而叫做长轮询。

其前端代码基本和短轮询一致，只不过把请求的超时时间设置较长（比如1分钟），然后无论请求成功或失败，马上再次发起请求即可。
```javascript
let id = 0
function poll() {
  $.ajax({
    url: '/api/long-polling',
    data: { id },
    timeout: 60000, // 请求的超时时间设置较长
  }).done((res) => {
    id += res.length
  }).always(() => {
    setTimeout(poll, 1000)
  })
}
poll()
```
后端写法
```javascript
const id = parseInt(query.id || '0', 10) || 0;
const cbk = 'long-polling';
delete callbacks[cbk];
const data = datas.slice(id);
res.writeHead(200, { 'Content-Type': 'application/json' });
// 发起请求时，正好有新消息就返回
if (data.length) {
 return res.end(JSON.stringify(data));
}
// 注册新消息回调
callbacks[cbk] = (d) => {
 res.end(d);
};
```
这样，相对于短轮询，少了很多无意义的请求，而且消息的实时性也非常好。缺点就是当服务端有异常时，会导致长轮询短时间内不断发起请求，可能让服务端承受更大的压力，所以两次长轮询之间最好有一定间隔，或者异常检测机制。

> 3. SSE（Server-sent events）
前面提到的轮询、长轮询都是一问一答式的，一次请求，无法推送多次消息到前端。而SSE就厉害了，一次请求，N次推送。
其原理可以理解为下载一个巨大的文件，文件的内容分块传给前端，每块就是一次消息推送。

前端写法，有专门的EventSource来接收，使用起来很方便
```javascript
let id = 0
const es = new EventSource('/api/sse')
es.onmessage = (e) => {
  try {
    const res = JSON.parse(e.data)
  } catch (e) {
    console.log(e)
  }
}
es.onopen = () => {
  pushStatus(s, '等待消息中', 'success')
}
es.onerror = () => {
  pushStatus(s, '连接错误', 'danger')
}
```
后端写法，核心在于Content-Type: text/event-stream，这要让前端知道这是SSE，还有就是传输信息的格式比较特别一点，详细的可以看 MDN（ https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events ）
```javascript
res.writeHead(200, {
  // 这个是核心
  'Content-Type': 'text/event-stream',
  'Connection': 'keep-alive',
});
// 把缓存的信息推送给前端
res.write(`data: ${JSON.stringify(datas)}\n\n`);
// 注册新消息回调
callbacks[cbk] = (d) => {
  res.write(`data: ${d}\n\n`);
};
```
SSE还支持自动重连！服务器短时间异常，恢复之后，无需额外代码，SSE就自动重连上了。

> 4. WebSocket
WebSocket可以一次连接，双向推送，而SSE只能从服务端推送到前端。从这个角度来看，用WebSocket来单做服务端推送，有点大材小用了。

前端写法与SSE类似，十分简单，只不过请求链接为ws://或者wss://开头（相当于http://和https://）
```javascript
const ws = new WebSocket('ws://localhost:3000/ws');
ws.onmessage = e => {
  try {
    const c = JSON.parse(e.data);
  } catch (err) {
    console.log(err);
  }
};
```
如果要用原生Node.js来写WebSocket服务，就会麻烦一些了，一般情况都会使用类似socket.io之类的三方库来降低实现成本。这边也就在网上摘抄了一段代码来简单实现一下
后端写法
```javascript
server.on('upgrade', (req, socket) => {
  const acceptKey = req.headers['sec-websocket-key'];
  const hash = generateAcceptValue(acceptKey);
  // 生成响应头信息
  const responseHeaders = [ 'HTTP/1.1 101 Web Socket Protocol Handshake', 'Upgrade: WebSocket', 'Connection: Upgrade', `Sec-WebSocket-Accept: ${hash}` ];
  // 告知前端这是WebSocket协议
  socket.write(responseHeaders.join('\r\n') + '\r\n\r\n');
  // 发送数据
  socket.write(constructReply(datas));
  callbacks[cbk] = (d) => {
    socket.write(constructReply(d));
  }
});
```
Websocket没有像SSE一样有自动重连，这块需要自行实现。一般网页实时聊天之类需要双向推送的，都会使用WebSocket来实现。

> 5. iFrame
原理类似使用iFrame加载一个巨大的网页，利用浏览器会一边加载一边解析执行返回的HTML，通过分次返回Script标签来实现消息推送。其实现类似SSE

前端代码很简单，要注册一个回调给iframe使用
```javascript
// 注册给iframe使用的方法
window.change = function(data) {};
$('body').append('<iframe src="/api/iframe"></iframe>');
```
后端也很简单，有消息的时候返回script标签即可
```javascript
// 有推送消息返回缓存信息
res.write(`<script>window.parent.change(${JSON.stringify(datas)});</script>`);
// 注册消息回调
callbacks[cbk] = (d) => {
  res.write(`<script>window.parent.change(${d});</script>`);
};
```
iFrame这种方式缺点没有判断加载异常的情况

### 总结
| 方案 | 实时性 | 单次连接 | 自动重连	| 断线检测 | 双向推送	| 
| ---- | :---: | :---: |:---: | :---: |:---: |
| 短轮询 |  ❌  | ❌ | ➖ |  ✅   | ❌ |
| 长轮询 |  ✅  | ❌ | ➖ |  ✅   | ❌ |
| SSE |  ✅  | ✅ | ✅ |  ✅   | ❌ |
| WebSocket |  ✅  | ✅ | ❌ |  ✅   | ✅ |
| iFrame |  ✅  | ✅ | ❌ |  ❌   | ❌ |

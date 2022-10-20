# Node.js 创建 webSever


### 创建 webSever

Node.js 创建 Http 服务器可以利用 http 包的 createServer 方法创建一个简单的 web 服务器

```js
const http = require('http');

const server = http.createServer((req, res) => {
  if (req.url === '/ping') {
    res.end('pong');
  }
  res.end('hello web service');
});

server.listen(3000, () => {
  console.log('http start, listen on port 3000');
});
```


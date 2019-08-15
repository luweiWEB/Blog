Fiddler是一个非常强大的代理工具，可以让你的前端开发调试更加方便。下面介绍在微信开发调试方面的应用。

微信网页开发中，由于有js接口安全域名和授权域名等的限制，导致部分功能需要部署到线上才能测试。通过代理可以实现本地调试网站的所有功能。

### 配置代理规则

全站转发可以这样设置：`Tools` -> `HOSTS`
![HOST配置](https://s2.ax1x.com/2019/07/27/eMSuVJ.png)

图片中表示`your.domain.com`的请求全部转发到`127.0.0.1:8000`。第二个参数的限制是：不能加协议、路径或参数。

如果你的网站域名和接口域名是同一个，那就不能使用全站转发了，需要html、css、js、websocket请求转发到本地，接口调用请求则直接发送到远程服务器。

可以使用自定义规则实现
![自定义规则](https://s2.ax1x.com/2019/07/27/eMPCLT.png)

上面图片中的正则表达式和目标地址如下：

```
regex:^http://your.domain.com(?!/api|/swagger|/webjars|/configuration/ui)(.*)


http://localhost:8000$1
```

本条规则表示：将`your.domain.com`下的`http`请求转发到`localhost:8000`，其中`/api`、`/swagger`、`/webjars`、`configuration/ui`开头的路径不转发。

目标地址表达式中的`$1`代表原始请求URL域名后面的字符串，包括`path`和`search`。

### 设置代理服务器

打开微信开发者工具，`设置` -> `代理设置` -> 选择`手动设置代理`。
`Fiddler`默认运行在`127.0.0.1:8888`。

### 在真机上测试

在真机上测试本地微信网页项目的步骤如下：

1. 手机和电脑连接同一个局域网。
2. 设置`Fiddler`允许远程连接，`Tools` -> `Options` -> `Connections` -> 勾选`Allow remote computers to connect`。

![allow-remote](https://s2.ax1x.com/2019/07/27/eMEDRH.png)

3. 查看电脑在局域网中的IP地址，命令行输入`ipconfig`(windows)。

![getIp](https://s2.ax1x.com/2019/07/27/eMead0.png)

4. 手机网络配置代理服务器。

![phone-config](https://s2.ax1x.com/2019/07/27/eMe6y9.png)

到这里，本篇文章的主要内容就结束了，如果你想了解更多关于Fiddler和代理工具的使用，可以参考我同事的文章[代理工具Fiddler -调试与替换接口状态](https://www.luweitech.cn/article_detail?id=17)，
[代理工具做微信项目的调试配置](https://www.luweitech.cn/article_detail?id=18)。
如果你想了解使用nodejs如何实现上述以及更多自定义的功能，敬请往下阅读。

### nodejs实现代理服务器

下文中，client表示客户端（浏览器），proxy表示代理服务器，server表示目标服务器

#### HTTP

实现HTTP代理服务器是非常简单的，因为HTTP为明文传输，所以proxy只需要解析client的HTTP报文，再向server发送相同的请求，server响应时，proxy将HTTP响应状态，响应首部字段和响应主体返回给client即可。

这里可以使用nodejs的`http`模块实现

```js
const http = require('http');
const { URL } = require('url');

let server = http.createServer((req, res) => {
  let {
    pathname,
    search,
    hostname,
    port
  } = new URL(req.url);

  console.log('http ', req.url);

  // 后端api调用请求直接发送给远程服务器，除此之外的HTTP请求发送给本地运行的端口
  if (!pathname.startsWith('/api')) {
    hostname = 'localhost';
    port = 8000; // 项目运行的端口
  }

  req.pipe(http.request({
    hostname,
    port,
    path: `${pathname}${search}`,
    method: req.method,
    headers: req.headers
  }, (response) => {
    res.writeHead(response.statusCode, response.statusMessage, response.headers);
    response.pipe(res);
  }));
});

server.listen(8888);

```

#### HTTPS

只有HTTP代理服务器是不够的，因为不管是微信开发工具，还是浏览器，都有可能发送HTTPS请求。比如微信开发者工具的登录和域名校验就是使用的HTTPS与微信服务器通信的，如果不代理这部分流量是无法正常运行微信开发者工具的。

HTTPS因为报文加密的原因，proxy不能解析报文后伪装client发送请求。最常见的解决方案是web隧道，proxy建立和client、server的TCP连接，之后盲转发两端的数据。

建立web隧道的方式之一是使用HTTP的CONNECT方法，实际上客户端（浏览器）设置了代理服务器后，client发出的HTTPS请求是不同的，它首先会使用CONNECT方法发送HTTP请求，请求proxy建立连接，这是proxy能代理HTTPS的关键。

client请求proxy与server建立TCP连接的HTTP报文如下：

```
CONNECT your.domain:443 HTTP/1.1
Host: your.domain:443
Connection: keep-alive
User-Agent: M....

```

proxy在与server建立TCP连接后，按照约定，需要`200 Connection Established`响应，响应首部字段可自定义：

```
HTTP/1.1 200 Connection Established
Connection: close

```

所以http服务器就可以代理https请求。实际上，按照上面的原理http服务器能够代理很多其他协议的流量。

https代理服务器需要使用http和net模块，对上面的http代理的代码扩展即可

```js
server.on('connect', (req, clientSocket) => {
  let {
    port,
    hostname
  } = new URL(`http://${req.url}`);

  console.log('https', req.url);

  let serverSocket = net.connect(port || 80, hostname, () => {
    clientSocket.write(
      `HTTP/1.1 200 Connection Established
Connection: close

`
    );

    clientSocket.pipe(serverSocket);
    serverSocket.pipe(clientSocket);
  });
});
```

从实现方式可以看出来，这种代理服务器是无法正常获取和更改通信双方的数据的。如果要实现能监听和更改通信数据的HTTPS代理服务器，可以使用自签名证书实现，这里不做探究。

### websocket

本地开发的项目往往有热更新功能，而热更新的通信依靠websocket，所以websocket代理也是必不可少的。websocket的连接也是用HTTP建立起来的。
如果根据我们之前了解的websocket知识，client会向服务器发送协议升级请求（请求报文中包含特殊的请求首部字段），服务器响应`101 Switching Protocols`，之后的数据则转为websocket协议发送。根据以上流程，同样只需要对http服务器代码扩充即可，我们很容易写出如下代码：

```js
server.on('upgrade', (req, clientSocket) => {
  let {
    pathname,
    search,
    hostname,
    port
  } = new URL(req.url);

  console.log('websocket', req.url);

  let request = http.request({
    pathname: 'localhost',
    port: 8000, // 项目运行的端口
    method: req.method,
    headers: req.headers
  });

  req.pipe(request);

  request.on('upgrade', (response, serverSocket) => {
    let resStr = 'HTTP/1.1 101 Switching Protocols\r\n';

    for (let [key, value] of Object.entries(response.headers)) {
      resStr += `${key}: ${value}\r\n`;
    }

    resStr += '\r\n';

    clientSocket.write(resStr);
    clientSocket.pipe(serverSocket);
    serverSocket.pipe(clientSocket);
  });
});

server.listen(8888)
```

上面的写法实际上是反向代理服务器的写法。即，浏览器直接建立到`ws://localhost:8888`的请求，该代理服务器是能够将请求转发到`8000`端口的，但当浏览器设置了代理服务器后，发送websocket请求和没设置前是不同的，它同样会先向proxy请求建立连接，所以代理websocket请求和代理https请求代码是一样的，我们在`connect`事件中做好区分即可。

```js
server.on('connect', (req, clientSocket) => {
  let {
    port,
    hostname
  } = new URL(`http://${req.url}`);

  console.log('https', req.url);

  // websocket请求发送到本地8000端口
  if (req.url === 'your.domain.com:80') {
    port = 8000; // 项目运行的端口
    hostname = 'localhost';
  }

  let serverSocket = net.connect(port || 80, hostname, () => {
    clientSocket.write(
      `HTTP/1.1 200 Connection Established
Connection: close

`
    );

    console.log(req.url, 'connect');

    clientSocket.pipe(serverSocket);
    serverSocket.pipe(clientSocket);
  });
});
```

经过以上三步，一个带有完整功能的代理服务器就写好了，想要实现自定义功能，无非是在请求报文识别和server响应报文篡改上做文章，尽情发挥你的创造力吧。

原文地址: [https://www.talkmoney.cn/blog/articleId367]()



作者简介：叶茂，芦苇科技web前端开发工程师，代表作品：口红挑战网红小游戏、服务端渲染官网、微信小程序粒子系统。擅长网站建设、公众号开发、微信小程序开发、小游戏、公众号开发，专注于前端领域框架、交互设计、图像绘制、数据分析等研究。 一起并肩作战： [yemao@talkmoney.cn](mailto:yemao@talkmoney.cn) 访问 [www.talkmoney.cn](http://www.talkmoney.cn/) 了解更多
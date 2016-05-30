#Linux安装NodeJs并配合nginx实现反向代理

## NodeJs

### 是什么

> Node.js是一个Javascript运行环境(runtime)。实际上它是对Google V8引擎进行了封装。V8引 擎执行Javascript的速度非常快，性能非常好。

> Node.js对一些特殊用例进行了优化，提供了替代的API，使得V8在非浏览器环境下运行得更好。


### 本地安装(OS X)

#### 版本选择

- V4.4.4,长期支持版本,成熟可靠
- V6.2.0 稳定版本,最新特性

这里我还是倾向于使用最新的版本~

#### 下载安装包

`https://nodejs.org/dist/v6.2.0/node-v6.2.0.pkg`

双击安装安装包

下一步下一步,就安装完成了。

简单执行

`node -v`

`v6.2.0`

### 本地运行(OS X)

#### 创建demo文件

```
const http = require('http');
const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World\n');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```


写入到文件`example.js`

#### 执行文件 

`node example.js`

这时命令行输出`Server running at http://127.0.0.1:3000/`

同时在浏览器输入`http://127.0.0.1:3000/`,页面输出`Hello World`

关闭终端,页面不再可用。

#### Express框架

我们这里采用Express框架进行网站项目demo的搭建。

`npm install express`

```
node_modules
```

创建demo.js文件
```
var express = require('express');
app = express(); 
app.use(express.static(__dirname + '/public'));  
app.listen(8081)
```

在同级文件夹创建`public`文件夹,里面放入静态文件`1.jpg`
 
在浏览器输入`http://127.0.0.1:8081/1.jpg`

查看Response Headers,`X-Powered-By:Express`

### 服务器安装(CentOS 7)

#### 安装node

```
curl --silent --location https://rpm.nodesource.com/setup | bash -
yum -y install nodejs
yum install npm

```

#### 关于Node的版本

我上一步通过node安装的版本号是`v0.10.42`,一开始以为错了,经过查资料发现,目前node共维护了4个版本

- v0.10.42 (LTS)
- v0.12.10 (LTS)
- 4.4.5 LTS
- 6.2

呵呵哒,真乱。


#### 编写demo实例

这部分的流程跟上面的一致。

#### 安装forever 并运行

`npm install forever -g`

`forever start app.js `

#### 配置Nginx

`cd /usr/local/nginx/conf/vhost/`

`vi demonode.coderfix.cn.conf`

```
server {
listen 80;
server_name demonode.coderfix.cn;
    location / {
    proxy_pass http://127.0.0.1:8899;
    }
}
```


Nginx解析域名,转发给本地的nodejs的8899端口~

 
#### 配置域名解析并访问

`http://demonode.coderfix.cn/`

这样就完成了nodejs和nginx的部署。

### 可能出现的问题

#### Nodejs服务多开导致报错

```
events.js:72
    throw er; // Unhandled 'error' event
          ^
Error: listen EADDRINUSE
    at errnoException (net.js:884:11)
    at Server._listen2 (net.js:1022:14)
    at listen (net.js:1044:10)
    at Server.listen (net.js:1110:5)
    at Object.<anonymous> (folderName/app.js:33:24)
    at Module._compile (module.js:456:26)
    at Object.Module._extensions..js (module.js:474:10)
    at Module.load (module.js:356:32)
    at Function.Module._load (module.js:312:12)
    at Function.Module.runMain (module.js:497:10)
```

关掉之前启动的进程,再开就好了。

```
ps aux | grep node

kill -9 ****
```
 

## 参考资料

- http://nodejs.cn/doc/node/index.html
- http://nodejs.cn/download/
- http://www.csdn.net/article/2013-08-28/2816731-absolute-beginners-guide-to-nodejs
- https://cnodejs.org/topic/5021c2cff767cc9a51e684e3
- https://nodejs.org/en/blog/release/v0.10.42/
- http://www.nodejs.net/a/20141030/122537.html
- http://stackoverflow.com/questions/16827987/expressjs-throw-er-unhandled-error-event

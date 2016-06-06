#Yii2实现RESTful架构配置最佳实践

## 为什么要用RESTful API 

> 在服务器端，应用程序状态和功能可以分为各种资源。资源是一个有趣的概念实体，它向客户端公开。资源的例子有：应用程序对象、数据库记录、算法等等。每个资源都使用 URI (Universal Resource Identifier) 得到一个唯一的地址。所有资源都共享统一的接口，以便在客户端和服务器之间传输状态。使用的是标准的 HTTP 方法，比如 GET、PUT、POST 和 DELETE。Hypermedia 是应用程序状态的引擎，资源表示通过超链接互联。


无状态,分层,可扩展

## 基于Yii2的RESTful API 的实现

### 版本控制

Mdedia Type， 即在HTTP请求的header中使用Media Type标记该请求想获取的资源， 同样的可以不设置或设置通用的Media Type表示最新版本的API。 

```
===>  
GET /customer/123 HTTP/1.1  
Accept: application/vnd.xianlinbox.customer-v3+json  
  
<===  
HTTP/1.1 200 OK  
Content-Type: application/vnd.xianlinbox.customer-v3+json  
  
{"customer":  
  {"name":"Xianlinbox"}  
} 

```

API（面向资源开发）-> Server端(面向数据开发)





## 参考资料

- http://www.yiichina.com/doc/guide/2.0/rest-quick-start
- http://ningandjiao.iteye.com/blog/1990004
- http://www.cnblogs.com/yuzhongwusan/p/3152526.html
https://zhuanlan.zhihu.com/p/20034107

# 使用Git的Webhooks进行服务器自动部署代码


## 前言

之前一直想做自动化运维相关的东西,最近有时间,开始整理相关资料,增强服务器运维的技术点.

## 官方文档说明

本文并不是翻译官方文档,而是用自己的理解描述出对应的含义,算是一个说明.

### 说明

#### Webhooks

- Events
- Payloads
- Ping Event
- Service Hooks

Webhooks允许我们建立关于项目的在GitHub上的事件集成.当事件触发的时候,会通过HTTP POST的方式,向我们的服务器发送请求.我们可以用这个服务器去*更新外部跟踪*,*触发CI build*,*更新备份镜像*,*部署到生产服务器*.

每个Webhook都可以安装在一个项目或者组织上,安装完毕之后,就可以随着订阅的时间进行触发.

每个项目的每个事件最多可以安装20个Webhook.


##### 事件

你可以选择你要监听的对应的事件.你可以使用*通配符*去匹配事件.还可以限制对应的HTTP请求访问你的服务器.默认是通过`POST`事件,你可以在任何时候通过改变API或者UI去改变事件.

事件的指定的颗粒度很精细,你可以指定一个issue的打开,关闭,标签等.

支持的事件在下面的表

| 名称  | 描述 |
|---|---|
| *  |  触发的任何情况,通配符  |
|  commit_comment  |  提交完成了一个 Commit |
|  create | 创建了一个Branch或者Tag |
|delete |  删除了一个Branch或者Tag|
|deployment | 一个Repository通过API有个一个新的部署 |
|deployment_status	  | 一个Repository的部署的状态通过API改变 |
|fork |一个Repository被fork了 |
|gollum	 |Wiki页面被更新 |
|issue_comment | 一个issue的评论被创建,修改,删除 |
|issues | 一个issues的全部状态 |
| member	 | 一个用户添加了一个collaborator到项目中  |
| membership | 一个用户添加或者移除了一个从一个组织,区别去上一个的是,上一个针对于项目,这个针对于组织. | 
|page_build |  GitHub page 创建或者失败 |
|public | 私有项目转换成公开项目 |
| pull_request_review_comment  |  一个pull_request的评论的状态变化  |
|pull_request  |  一个pull_request的所有状态变化  |
| push | push一次代码,操作对象包括修改分支,标签.调用API的也会被算进去.默认的事件. |
|repository | 一个项目被创建,删除,转换公开,转换私有. |
| release  | 在一个项目上创建一个release|
| status | 一个项目的status通过API进行改变 |
|team_add | 项目添加或者修改一个team|
| watch |一个项目被用户star |

###### 通配符事件

我们也支持了通配符事件来匹配时间,用法和一般的事件没有区别.

##### Payloads

每个事件都会有一个十分详细的Payloads.

> NOTE:Payloads上限是5MB,产生过大的Payloads不会被失效.推荐监控Payloads的大小来确保交付.

###### 交付的headers

触发后的HTTP请求会发送特殊的header给我们设定的URL.

|  Header | 描述 |
|---|---|
| X-GitHub-Event  |  被触发的事件的名称  |
| X-Hub-Signature | HMAC十六进制的签名 |
|X-GitHub-Delivery  | 这次返回的唯一的ID|

还有`User-Agent`也会有前缀`GitHub-Hookshot/`

###### 举例

```json

POST /payload HTTP/1.1

Host: localhost:4567
X-Github-Delivery: 72d3162e-cc78-11e3-81ab-4c9367dc0958
User-Agent: GitHub-Hookshot/044aadd
Content-Type: application/json
Content-Length: 6615
X-GitHub-Event: issues

{
  "action": "opened",
  "issue": {
    "url": "https://api.github.com/repos/octocat/Hello-World/issues/1347",
    "number": 1347,
    ...
  },
  "repository" : {
    "id": 1296269,
    "full_name": "octocat/Hello-World",
    "owner": {
      "login": "octocat",
      "id": 1,
      ...
    },
    ...
  },
  "sender": {
    "login": "octocat",
    "id": 1,
    ...
  }
}

```


##### Ping 事件

当你创建一个新的webhook,会被发送一个请求,用来测试是不是有效,还可以被重复发送.

###### Ping Event Payload

|  Key  |  Value  |
|---|--- |
|  zen | 随机字符串  |
| hook_id | 唯一ID |
| hook  | webhook配置项 |

##### Service Hooks

除了webhooks,还可以集成开源社区的其他的服务,安装方式和webhooks很像.

注意下面几点

- 只能被安装在项目,不能对组织
- 一个项目只对应一个服务端
- 每个服务只支持特定的事件，这取决于服务的实现
- 每一个服务都有自己独特的配置选项


看到一个完整的可用服务列表，它们支持的事件，和配置选项，看看https://api.github.com/hooks。为所有服务挂钩的文档可以在GitHub服务库文件目录中找到。


注意：如果你正在建设一个新的整合，你应该把它作为webhook。我们建议创建一个OAuth应用自动安装和管理您的用户webhooks。我们将不再接受新的服务GitHub服务库。


#### 创建Webhooks

现在我们了解webhooks的基础知识，让我们通过建立我们自己的动力一体化进程webhook。在本教程中，我们将创建一个版本库webhook负责列出我们的仓库是多么的流行，基于问题数量接收每一天。


创建一个webhook是一个两步的过程。首先你需要设置你的行为通过GitHub webhook什么事件要听。在那之后，您将设置您的服务器来接收和管理负载。

##### 建立一个webhook

在一个项目,进入设置界面,选择*Webhooks & services*,点击*Add webhook*. 

选择配置项,配置参数,就可以使用了.


##### 有效的URL

填写GitHub服务器在事件触发的时候访问的URL.

如果是本地测试,可以采用 [这个地址](https://developer.github.com/webhooks/configuring/)进行本地测试,这里使用的是`ngrok`来进行本地服务器映射外网来达到调试的目的.


##### Content Type

Webhooks能够返回两种格式的内容

- `application/json` 
- `application/x-www-form-urlencoded`

一般我们都选择`application/json`.

##### 事件

事件是webhooks的核心,项目webhooks的被触发后会通过你的URL传递过去.

最后,点击*Add webhook*,就完成了整个创建流程.

#### 配置你的服务端


##### 安装ngrok

下载地址  https://ngrok.com/download

```shell
./ngrok -help
./ngrok http 8081
```
 
 就会出现
 
```
 ngrok by @inconshreveable                                                                    (Ctrl+C to quit)
                                                                                                              
 Tunnel Status                 online                                                                         
 Version                       2.0.25/2.1.1                                                                   
 Region                        United States (us)                                                             
 Web Interface                 http://127.0.0.1:4040                                                          
 Forwarding                    http://6bba728e.ngrok.io -> localhost:8081                                       
 Forwarding                    https://6bba728e.ngrok.io -> localhost:8081                                      
                                                                                                              
 Connections                   ttl     opn     rt1     rt5     p50     p90                                    
                               0       0       0.00    0.00    0.00    0.00                                   
```
 
 
 在本地浏览器输入 `http://127.0.0.1:4040/` ,就会进入本地测试的管理界面. :)
 
 
```
 No requests to display yet
 To get started, make a request to one of your tunnel URLs
 http://5d3951bf.ngrok.io
 https://5d3951bf.ngrok.io
```

再去浏览器输入可以对外访问的地址 `http://5d3951bf.ngrok.io`
 
这样就完成了本地服务器可以进行外部访问的映射.

这时给出的返回是

```
Failed to complete tunnel connection
The connection to http://5d3951bf.ngrok.io was successfully tunneled to your ngrok client, but the client failed to establish a connection to the local address localhost:8081.
Make sure that a web service is running on localhost:80 and that it is a valid address.
The error encountered was: dial tcp 127.0.0.1:8081: connection refused
```

表示已经连上了本地测试,但是本地的80端口没有给出返回,需要我们进行本地环境的代码编写.


#####  写server端处理代码

首先解决gem在国内不能用的情况

```shell
gem sources --remove https://rubygems.org/
gem sources -a https://ruby.taobao.org/
gem sources -l
```

安装`Sinatra`,进行本地的测试代码

```shell
gem install sinatra
```

编写对应的代码,GitHub给出了相关的demo代码,熟悉阶段可以直接用.

```
vi demo.rb

require 'sinatra'
require 'json'

post '/payload' do
  push = JSON.parse(request.body.read)
  puts "I got some JSON: #{push.inspect}"
end
```


创建本地服务,这里注意的是,默认的端口是4567,我们需要改成和上面配置一样的**8081**

```
ruby demo.rb -p 8081
```


经过这个操作,我们就可以在本地进行测试了.

注意的是,这部分的配置测试环节并不是强制的.

我们可以把sinatra换成本地的apache+php或者别的,或者,我们可以直接在自己的服务器上进行测试,只要保证可以看到对应的返回方便测试就是可以的.

接下来我们就可以进行测试了.


#### 测试Webhooks

##### 监听地址

我们回到项目的配置地址,填写的URL为`http://5d3951bf.ngrok.io/payload`. 

就可以测试了

[Recent Deliveries](https://developer.github.com/assets/images/webhooks_recent_deliveries.png)



##### 检查结果

我们可以通过点击每条结果上前面的icon,查看对应的详细数据,展示结果很直观,不用多解释.


###### Request

![Request](https://developer.github.com/assets/images/payload_request_tab.png)


###### Response

![Request](https://developer.github.com/assets/images/payload_response_tab.png)


#### 保护你的webhooks

由于被动触发的webhooks是对外公开的,所以我们需要保证每次请求都是合法的.IP白名单是一个手段,但是采用密钥的方式更好~

##### 设置加密token

设置项目setting,在创建webhooks的时候,填写一个随机的字符串作为Secret.

![](https://developer.github.com/assets/images/webhook_secret_token.png)

推荐使用`ruby -rsecurerandom -e 'puts SecureRandom.hex(20)'`进行加密字符串的生成.

生成结果类似`b93cc391fd5773057397ada7070e91add983cbb9`.

接下来在服务器采用定义环境变量的方式存储token,**千万不要写在代码中!**

```
export SECRET_TOKEN=your_token
```

##### 服务端校验



## 参考资料

- https://developer.github.com/webhooks/
- https://developer.github.com/webhooks/configuring/
- https://ngrok.com/
- https://ngrok.com/docs/2#expose
- http://www.sinatrarb.com/
- http://www.sinatrarb.com/intro-zh.html
- http://www.haorooms.com/post/gem_not_use
# 使用Git的Webhooks进行服务器自动部署代码


## 前言

之前一直想做自动化运维相关的东西,最近有时间,开始整理相关资料,增强服务器运维的技术点.

## 官方文档说明

本文并不是翻译官方文档,而是用自己的理解描述出对应的含义,算是一个说明.

### 概述

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





## 参考资料

- https://developer.github.com/webhooks/
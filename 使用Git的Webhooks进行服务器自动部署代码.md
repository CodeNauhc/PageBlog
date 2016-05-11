# 使用Git的Webhooks进行服务器自动部署代码


## 前言

之前一直想做自动化运维相关的东西,最近有时间,开始整理相关资料,增强服务器运维的技术点.

## 官方文档

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



## 参考资料

- https://developer.github.com/webhooks/
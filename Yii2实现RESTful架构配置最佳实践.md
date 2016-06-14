#Yii2实现RESTful架构配置最佳实践

## 为什么要用RESTful API 

> 在服务器端，应用程序状态和功能可以分为各种资源。资源是一个有趣的概念实体，它向客户端公开。资源的例子有：应用程序对象、数据库记录、算法等等。每个资源都使用 URI (Universal Resource Identifier) 得到一个唯一的地址。所有资源都共享统一的接口，以便在客户端和服务器之间传输状态。使用的是标准的 HTTP 方法，比如 GET、PUT、POST 和 DELETE。Hypermedia 是应用程序状态的引擎，资源表示通过超链接互联。


无状态,分层,可扩展

## 基于Yii2的RESTful API 的实现

### 不用自带的REST实现方式

首先,Yii2自带了实现RESTful api的方式,但是,官方的例子过于简单,把一个资源限定成了数据库中的一个表,这显然是和REST中定义的资源不相符的,并且实际的业务需求过于复杂,不能通过一个表进行数据的操作。

所以,我们采用了另一种方式,通过使用Yii2的不同组件,完成RESTful API的构建

### 路由

REST要求定义资源,采用不同HTTP方式进行访问。

这里用到了框架内部的路由

在项目的配置文件`/config/web.php`,中,对不同资源进行路由设置,从而达到同一个URL用不同的访问方式来处理不同业务的目的。

```php
        //url配置对应规则
        'urlManager' => [
            'enablePrettyUrl' => true,
            'showScriptName' => false,
            'rules' => [
                //配置规则,实现RESTful的方式
                'POST accounts'=>'account/login', // 登录
                'GET accounts'=>'account/get-avatar', //获取头像
                "GET accounts/status"=>'account/status', //查询登录状态

                //获取用户组信息
                "GET groups"=>"account/profile",

                //用户申请加入用户组的接口
                "GET users/ask"=>"user/apply", //申请列表
                "PUT users/<uid:\d+>"=>"user/pass", //通过
                "PATCH users/<uid:\d+>"=>"user/nopass", //不通过
                "GET users"=>"user/list",  //通过的用户列表
                "DELETE users/<uid:\d+>"=>"user/remove", //移除现有的用户
                "GET users/<uid:\d+>"=>"user/info", //网红的具体信息
                "GET users/serach"=>"user/serach", //搜索当前用户的用户


                //消息中心相关
                "GET  notices"=>"notice/list", //消息列表
                "PUT notices/<nid:\d+>"=>"notice/mark", //消息标记为已读
                "GET notices/status"=>"notice/status", //获取消息的概要
            ],
```

这部分的设计,参考官方文档对于RESTful实现的部分

> GET /users: 逐页列出所有用户
  HEAD /users: 显示用户列表的概要信息
  POST /users: 创建一个新用户
  GET /users/123: 返回用户 123 的详细信息
  HEAD /users/123: 显示用户 123 的概述信息
  PATCH /users/123 and PUT /users/123: 更新用户123
  DELETE /users/123: 删除用户123
  OPTIONS /users: 显示关于末端 /users 支持的动词
  OPTIONS /users/123: 显示有关末端 /users/123 支持的动词


### 接收数据

我们需要获取客户端传递过来的`GET`,`POST`,`Header`的数据,框架自带了`Yii::$app->request`,可以直接得到客户端传过来的数据。

参考`\yii\web\Request|\yii\console\Request $request The request component. This property is read-only.`

### 处理数据

这部分和一般的项目一样,都用`/models`中的数据表对应的Model文件,继承了`\yii\db\ActiveRecord`的类,实现对数据表的CURD操作。

参考URL: http://www.yiichina.com/doc/guide/2.0/db-active-record

### 响应数据

我们在处理数据之后,需要返回给客户端对应的数据,在REST的设计规则里是这样处理的

- Body只返回主要的数据,比如用户列表,用户的详细数据
- Header返回其他的信息,包括页码信息,身份校验信息
- 完全的使用HTTP状态码作为资源被请求状态的返回,比如404,403

HTTP状态码的说明如下

|Code	|HTTP Operation|	Body Contents	|Description|
|:------------- |-------------| -----| -----|
|200	|GET,PUT|	资源	|操作成功|
|201	|POST	|资源,元数据	|对象创建成功|
|202	|POST,PUT,DELETE,PATCH|	N/A	|请求已经被接受|
|204	|DELETE,PUT,PATCH	|N/A|	操作已经执行成功，但是没有返回数据|
|301	|GET	|link	|资源已被移除|
|303	|GET	|link	|重定向|
|304	|GET	|N/A	|资源没有被修改|
|400	|GET,PSOT,PUT,DELETE,PATCH	|错误提示(消息)	|参数列表错误(缺少，格式不匹配)|
|401	|GET,PSOT,PUT,DELETE,PATCH	|错误提示(消息)	|未授权|
|403	|GET,PSOT,PUT,DELETE,PATCH	|错误提示(消息)	|访问受限，授权过期|
|404	|GET,PSOT,PUT,DELETE,PATCH	|错误提示(消息)	|资源，服务未找到|
|405	|GET,PSOT,PUT,DELETE,PATCH	|错误提示(消息)	|不允许的http方法|
|409	|GET,PSOT,PUT,DELETE,PATCH	|错误提示(消息)	|资源冲突，或者资源被锁定|
|415	|GET,PSOT,PUT,DELETE,PATCH	|错误提示(消息)	|不支持的数据(媒体)类型|
|429	|GET,PSOT,PUT,DELETE,PATCH	|错误提示(消息)	|请求过多被限制|
|500	|GET,PSOT,PUT,DELETE,PATCH	|错误提示(消息)	|系统内部错误|
|501	|GET,PSOT,PUT,DELETE,PATCH	|错误提示(消息)	|接口未实现|

格式化数据,我们的返回数据的格式为JSON

```php
$response= Yii::$app->response;
$response->format = \yii\web\Response::FORMAT_JSON; //返回json
```

### 错误处理

通过不同的错误码,返回不同的HTTP状态码。

由于的非常频繁的调用,所以对数据进行了封装。

错误处理类`Error.php`

```php

/**
 * 错误处理类
 * 
 * ------------------------------------
 * 
 * 错误处理扩展 seaslog
 * 
 * http://www.oschina.net/news/50333/seaslog-0-21
 * 
 * http://neeke.github.io/SeasLog/
 *
 * ------------------------------------
 * 
 * @author Calvin 
 * @version 1.0
 * @copyright (c) 2016,
 */

namespace app\components\error;

use Yii;

class Error {

    /**
     * 公共的错误返回处理,通过传入参数,返回对应的错误代码,这里也定义了错误返回的格式,以及对日志的记录
     *
     * @param $error_code
     * @return array
     */
    public static function errorJson($error_code) {
        $requests = Yii::$app->request; //返回值
        $post = json_encode($requests->post());
        $get = json_encode($requests->get());
        $headers = json_encode($requests->getHeaders()->toArray());

        //带入记录错误代码的文件
        $error_file = require_once \Yii::$app->basePath . "/components/error/ErrorCode.php";
        //获取http状态码,以及文字说明
        $error_info = $error_file["$error_code"];


        $http_code = $error_info['http_code'];
        $error_text = $error_info['remark'];

        $error_body = [ //设置返回的格式
            'request' => $requests->getUrl(),
            'method'=>$requests->getMethod(),
            'error_code' => $error_code,
            'error' =>$error_text,
        ];

        $response = Yii::$app->response;
        $response->statusCode=$http_code;
        $response->format = \yii\web\Response::FORMAT_JSON;
        $headers = Yii::$app->response->headers;
        $headers->add('X-Halo-Result', 0);
//        $response->data = $error_body;


//        $userIP = $requests->userIP;
        //写入日志
//        \SeasLog::error('error === {error} && error_text === {error_text}  && userIP === {userIP} && post === {post}  && get === {get} && header === {header} ', [
//            '{error}' => $error,
//            '{error_text}' => $error_file["$error"],
//            '{post}' => $post,
//            '{get}' => $get,
//            '{userIP}' => $userIP,
//            '{header}' => $headers,
//                ], $request);
//        $appLog = \Alibaba::AppLog();
//        $appLog->debug("debug-log-emssage");
//        $appLog->info("info-log-emssage");
//        $appLog->warn("warn-log-emssage");
//        $appLog->error("error-log-emssage");

        return $error_body;
    }

}


```

错误代码文件`ErrorCode.php`

```php
/**
 * 增加对不同类型的HTTP状态码的返回
 *
 * 错误码,HTTP状态码,返回值
 */
return [
    '1001'=>[
        'http_code'=>403,
        'remark'=>"Token验证失败",
    ],
    '1002'=>[
        'http_code'=>400,
        'remark'=>"参数错误",
    ],
    '1003'=>[
        'http_code'=>400,
        'remark'=>"参数不全",
    ],
    '2001'=>[
        'http_code'=>404,
        'remark'=>"用户不存在",
    ],

];
```

调用错误数据返回
```php
return Error::errorJson(1002);
```

## 文档

有句话说得好,程序员不愿意写文档,但是看到没有文档的项目又会抓狂。。。。

### 概括结构

一个合格的API文档应该包含下面几项

- 概括说明
- 加密协议
- 数据类型
- 错误处理
- 接口文档
- 参考资料

### 接口文档

接口文档告诉客户端,调用什么数据,怎么掉,异常了咋办。。。

- 简单说明
- 访问地址
- 请求方式
- 返回结果
- 返回结果字段说明
- 错误代码
- 更新记录

## 总结

RESTful API的好处在于更简洁的规范了数据请求的方式,通过资源来设计数据接口,方便客户端的调用,减少沟通成本。

不过协议毕竟只是个建议,我们可以根据自己项目的实际情况,有选择的满足协议的需求,更好的为自己的项目服务。

:)

## 参考资料

- http://www.yiichina.com/doc/guide/2.0/rest-quick-start
- http://ningandjiao.iteye.com/blog/1990004
- http://www.cnblogs.com/yuzhongwusan/p/3152526.html
- https://zhuanlan.zhihu.com/p/20034107
- http://my.oschina.net/freegeek/blog/303639

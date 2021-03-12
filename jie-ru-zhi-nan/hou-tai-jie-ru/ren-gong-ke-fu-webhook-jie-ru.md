# 人工客服webhook接入

## 场景描述&数据流

![chatbot webhook.png](https://i.loli.net/2021/03/12/KADJrhvsPN54zly.png)

## 接入api

### webhook

接入端需要准备以下信息给服务端

* 密钥：作为服务器调用接入端的回调接口，数据签名使用
* 以下的webhook（真实的webhook链接以接入端的为准，下面的链接只是示例）
  * 获取客服列表：[https://webhook.com/api/get\_agent\_info](https://webhook.com/api/get_agent_info)
  * 新建会话：[https://webhook.com/api/channel\_built](https://webhook.com/api/channel_built)
  * 通知通道被关闭：[https://webhook.com/api/channel\_finish](https://webhook.com/api/channel_finish)
  * 用户发送消息：[https://webhook.com/api/user\_send\_msg](https://webhook.com/api/user_send_msg)
  * 客服转接结果：[https://webhook.com/api/forward\_result](https://webhook.com/api/forward_result)

### server api

以下是服务器提供的api（下面链接只是示例，具体域名要根据部署来确定）

* 客服的会话列表 : [https://chatbot.qcloud.com/svrapi/channel\_list](https://chatbot.qcloud.com/svrapi/channel_list)
* 获取某个人历史消息: [https://chatbot.qcloud.com/svrapi/channel\_history\_msg](https://chatbot.qcloud.com/svrapi/channel_history_msg)
* 客服发送消息: [https://chatbot.qcloud.com/svrapi/agent\_send\_msg](https://chatbot.qcloud.com/svrapi/agent_send_msg)
* 客服关闭通道 : [https://chatbot.qcloud.com/svrapi/agent\_close\_channel](https://chatbot.qcloud.com/svrapi/agent_close_channel)
* 客服列表 : [https://chatbot.qcloud.com/svrapi/agent\_list](https://chatbot.qcloud.com/svrapi/agent_list)
* 客服转接 : [https://chatbot.qcloud.com/svrapi/agent\_forward\_user](https://chatbot.qcloud.com/svrapi/agent_forward_user)
* 客服退出 : [https://chatbot.qcloud.com/svrapi/update\_agent\_info](https://chatbot.qcloud.com/svrapi/update_agent_info)

## 技术实现

### 数据签名

* 密钥
* 数据字段进行字典排序
* 密钥加上字典序的字符串，进行md5得到签名\(小写\)

  举例：以[获取客服列表]()协议举例

  ```javascript
    secretKey密钥为：123456
    数字为：
    {
        "productID":"chatbot",
        "createTs":123456789,
    }
    签名原串为：raw=${secretKey}&ksort(fieldKey=fieldVal)=123456&createTs=123456789&productID=chatbot
    签名为：md5sum(${raw})=md5sum(123456&createTs=123456789&productID=chatbot)=8f621189327a845f1fffad8c45746041
  ```

### 协议设计

#### 获取客服列表

获取服务这个机器人的客服的列表，然后在有用户请求人工客服时候，会在此处取可用的客服提供人工客服的服务

```javascript
//server->agent
//POST /api/get_agent_info

//request
{
    "productID":"product id",
    "createTs":0,
    "sign":"sign"
}

//response
{
    "retCode":0,    //0:success,others:fail
    "retMsg":"success",
    "agentList":[
        {
            "agentID":"agent id,callee should granted unique",
            "agentName":"agent name",
            "agentStatus":"0:none,1:online,2:offline",
            "userNum":0,
        }
    ]
}
```

#### 客服退出

告知服务器，某个客服退出了（下线）

> 此时服务器会清理其还存在的会话，不会再给此客服分配用户，若是不调用这个告知客服下线，有可能已经存在的会话需要等待超时才能关闭（分配用户功能是正常的，是以此api是option）

```javascript
@TODO
//agent->server
//POST /api/update_agent_info

//request
{
    "productID":"product id",
    "agentID":"agent id",
    "agentStatus":0,//"0:none,1:online,2:offline",
    "createTs":0,
    "sign":"sign"
}

//response
{
    "retCode":0,    //0:success,others:fail
    "retMsg":"success"
}
```

#### 获取会话列表

获取目前的属于此客服的会话列表，此api一般在初始进入或者重登进入时候，需要调用

```javascript
//agent->server
//POST /api/channel_list

//request
{
    "productID":"product id",
    "agentID":"agent id",
    "createTs":0,
    "sign":"sign"
}

//response
{
    "retCode":0,    //0:success,others:fail
    "retMsg":"success",
    "channelList":[
        {
            "productID":"product id",
            "userID":"user id",
            "userName":"user name",
            "channelID":"channel id",
            "channelName":"channel name",
            "others":"",
        }
    ]
}
```

#### 新建会话

告知第三方，要建立一个新的会话，会话会带过去会话信息，会话双方（一个用户，一个客服）的信息

```javascript
//server->agent
//POST /api/channel_built

//request
{
    "productID":"product id",
    "channelID":"channel id",
    "channelName":"channel name",
    "agentID":"agent id",
    "userID":"user id",
    "userName":"user name", //此处注意，很有可能是空的，因为获取用户姓名信息有些敏感，非常可能获取不到
    "others":"",
    "createTs":0,
    "sign":"sign"
}

//response
{
    "retCode":0,    //0:success,others:fail
    "retMsg":"success"
}
```

#### 客服关闭通道

告知服务器，客服需要关闭某个通道（比如服务已经完成），此举是客服主动关闭通道

> 此关闭之后，服务器会再发送一条真正关闭的消息过来，才是真正关闭，此协议只是个通知，不代表关闭成功了

```javascript
//agent->server
//POST /api/agent_close_channel

//request
{
    "productID":"product id",
    "channelID":"channel id",
    "agentID":"agent id",
    "userID":"user id",
    "cuz":"close reason",
    "createTs":0,
    "sign":"sign"
}

//response
{
    "retCode":0,    //0:success,others:fail
    "retMsg":"success"
}
```

#### 通知通道被关闭

服务器告知第三方，某个通道被关闭了

```javascript
//server->agent
//POST /api/channel_finish

//request
{
    "productID":"product id",
    "channelID":"channel id",
    "agentID":"agent id",
    "userID":"user id",
    "cuz":"close reason",
    "createTs":0,
    "sign":"sign"
}

//response
{
    "retCode":0,    //0:success,others:fail
    "retMsg":"success"
}
```

#### 获取某个人历史消息

获取某个会话的历史消息，按照时间获取（即可能也会获取其和机器人的会话消息）

```javascript
//agent->server
//POST /api/channel_history_msg

//MsgPackageInfo
{
    "type":0, //nest pack msg,0:none,1:text,2:list,3:ImageUrl
    "header":"msg header,default null string",
    "body":"Type==1, text msg",
    "bodyList":"Type==2,list msg",
    "bodyListSep":"Type==2, the sep mark for list,default is crlf",
    "bodyUrl":"Type==3,a url",
    "footer":"msg footer,default null string",
}

//request
{
    "productID":"product id",
    "userID":"user id",
    "lastMsgId":"last msg id,null string mean lastest msg",
    "msgNum":"how many msg want to fetch,max=100",
    "createTs":0,
    "sign":"sign"
}

//response
{
    "retCode":0,    //0:success,others:fail
    "retMsg":"success",
    "msgList":[
        {
            "msgID":"msg id",
            "msgHeader":"msg header,json encoding object MsgPackageInfo",
            "msgFooter":"msg footer,json encoding object MsgPackageInfo",
            "msgBody":"msg body,json encoding object MsgPackageInfo",
            "sender":"who send this msg,userID|agentID|'' ('' mean robot service)",
            "recver":"who this msg been sent to,userID|agentID|'' ('' mean robot service)",
            "flow":0, //1:user->server,2:server->user
            "createTs":0, //msg create ts
            "serviceBy":0, //1:ai msg,2:customer service msg
            "channelID":"",
            "agentName":""
        }
    ]
}
```

#### 用户发送消息

用户发送消息

```javascript
//server->agent
//POST /api/user_send_msg


//消息组装信息
//MsgPackageInfo
{
    "type":0, //nest pack msg,0:none,1:text,2:list,3:ImageUrl
    "header":"msg header,default null string",
    "body":"Type==1, text msg",
    "bodyList":"Type==2,list msg",
    "bodyListSep":"Type==2, the sep mark for list,default is crlf",
    "bodyUrl":"Type==3,a url",
    "footer":"msg footer,default null string",
}

//request
{
    "productID":"product id",
    "agentID":"agent id",
    "channelID":"channel id",
    "userID":"user id",
    "msgId":"msg id",
    "msgHeader":"msg header,json encoding object MsgPackageInfo",
    "msgFooter":"msg footer,json encoding object MsgPackageInfo",
    "msgBody":"msg body,json encoding object MsgPackageInfo",
    "createTs":0,
    "sign":"sign"
}

//response
{
    "retCode":0,    //0:success,others:fail
    "retMsg":"success"
}
```

#### 客服发送消息

客服发送消息

```javascript
//agent->server
//POST /api/agent_send_msg

//request
{
    "productID":"product id",
    "agentID":"agent id",
    "channelID":"channel id",
    "userID":"user id",
    "msgType":0, //0:text,1:audio,2:pic
    "msgHeader":"msg header",
    "msgFooter":"msg footer",
    "msgText":"content when msgType==0",
    //"msgText":[], //"menu content when msgType==0"
    "msgMediaBase64Encoding":"base64 content when msgType==1",
    "imageUrl":"img url when msgType==2",
    "createTs":0,
    "sign":"sign"
}

//response
{
    "retCode":0,    //0:success,others:fail
    "retMsg":"success",
    "msgId":"msgId"
}
```

#### 客服列表

获取目前可用的客服列表

```javascript
//agent->server
//POST /api/agent_list

//request
{
    "productID":"product id",
    "agentID":"agent id",
    "createTs":0,
    "sign":"sign"
}

//response
{
    "retCode":0,    //0:success,others:fail
    "retMsg":"success",
    "agentList":[
        {
            "agentID":"agent id,callee should granted unique",
            "agentName":"agent name",
            "agentStatus":"0:none,1:online,2:offline",
            "userNum":0,
        }
    ]
}
```

#### 客服转接

客服转接用户给另外一个客服,此协议只是发起，后续的“客服转接结果”会告知是否成功

```javascript
@TODO
//agent->server
//POST /api/agent_forward_user

//request
{
    "productID":"product id",
    "channelID":"channel id",
    "fromAgentID":"from agent id",
    "toAgentID":"forward to another agent id",
    "userID":"user id",

    "createTs":0,
    "sign":"sign"
}

//response
{
    "retCode":0,    //0:success,others:fail
    "retMsg":"success",
    "forwardID":"",
}
```

#### 客服转接结果

通知转接结果

```javascript
@TODO
//server->agent(会给fromAgentID和toAgentID都发一次此消息)
//POST /api/forward_result

//request
{
    "retCode":0,    //0:success,others:fail
    "retMsg":"success",
    "targetAgentID":"this msg is sent to target agent",
    "productID":"product id",
    "forwardID":"",
    "fromAgentID":"from agent id",
    "toAgentID":"forward to another agent id",
    "channelID":"channel id",
    "userID":"user id",

    "createTs":0,
    "sign":"sign"
}

//response
{
    "retCode":0,    //0:success,others:fail
    "retMsg":"success"
}
```


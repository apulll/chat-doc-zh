# 人工客服websocket接入

## protocol

* package=cmd+body
* cmd=0xXXXX,cmd lenght is 6 byte at package head
* body=json object

## how it work



![wsmsg.png](https://i.loli.net/2020/12/18/wcKJRP6etrY1uAN.png)

## protocol detail

### agent login

```javascript
//cmd=0x2000
//client->server

//request
{
    "account":"login account,provide by tencent",
    "password":"jwt string for password,${secret_key} provide by tencent",
    "products":[
        "", //which products this agent want to service
    ]
}

//response
{
    "retCode":0, //0:success,others:fail
    "retMsg":"description for ${retCode}",
    "sid":"session id",
    "cid":"channel id"
}

//example
//request
0x2000{
    "account": "chatly@JHtUqFHMdHGe.chat.com"
    "password": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MDEzNjcyODQsImRhdGEiOnsiYWNjb3VudCI6ImNoYXRseUBKSHRVcUZITWRIR2UuY2hhdC5jb20iLCJpZCI6ImI1YWIyMTIwLWVlNWEtMTFlYS1iNmY4LTMxNmMyOGY1Nzg1MyIsIm5hbWUiOiJjaGF0bHkifSwiaWF0IjoxNjAxMzYzNjg0fQ.SkkyvQnbMhtNXAnM-fBdgbxCxf_phkXMZJJZiKNZ2-o"
    "products": ["chatly_cn", "chatly_en"]
}
//response
0x2000{
    "retCode": 0,
    "retMsg": "",
    "cid": "",
    "sid": "w7FfofVmWGNOwEco"
}
```

### agent logout

```javascript
//cmd=0x2001
//client->server

//request
{
    "account":"account",
    "products":[
        "", //which products this agent want to service
    ]
}

//response
{
    "retCode":0, //0:success,others:fail
    "retMsg":"description for ${retCode}",
}
```

### user in

```javascript
//cmd=0x2002
//server->client

//request
{
    "account":"account",
    "product_id":"which product user in",
    "cid": "channel id",
    "sid": "session id"
}

//response
{
    "retCode":0, //0:success,others:fail
    "retMsg":"description for ${retCode}",
    "account":"account",
    "product_id":"which product user in",
    "cid": "channel id",
    "sid": "session id"
}
```

### user out

```javascript
//cmd=0x2003
//server->client

//request
{
    "account":"account",
    "product_id":"which product user in",
    "cid": "channel id",
    "sid": "session id"
}

//response
{
    "retCode":0, //0:success,others:fail
    "retMsg":"description for ${retCode}",
    "account":"account",
    "product_id":"which product user in",
    "cid": "channel id",
    "sid": "session id"
}
```

### agent been kick\(cuz this agent relogin somewhere else\)

```javascript
//cmd=0x2004
//server->client

//request
{
    "account":"account",
    "product_id":"which product user in",
}

//response
{
    "retCode":0, //0:success,others:fail
    "retMsg":"description for ${retCode}",
}
```

### agent recv msg

```javascript
//cmd=0x2005
//server->client

//request
{
    "cid": "channel id",
    "sid": "session id",
    "msg": {
        "msgId"     :"msg id",
        "msgType"   :0, //"0:text,1:audio,2:pic",
        "msgHeader" :"head msg",
        "msgFooter" :"footer msg",
        //body
        "msgText"   :"content when msgType==0",
        "imageUrl"  :"url when msgType=2",
        "sender"    :"sender id", //user id
        "recver"    :"recver id", //agent id
        "createTs"  :0,//create timestamp for this msg
    }
}

//response
{
    "retCode":0, //0:success,others:fail
    "retMsg":"description for ${retCode}"
}
```

### agent send msg

```javascript
//cmd=0x2006
//client->server

//request
{
    "cid": "channel id",
    "sid": "session id",
    "msg": {
        "msgId"     :"msg id",
        "msgType"   :0, //"0:text,1:audio,2:pic",
        "msgHeader" :"head msg",
        "msgFooter" :"footer msg",
        //body
        "msgText"   :"content when msgType==0",
        "imageUrl"  :"url when msgType=2",
        "sender"    :"sender id", //agent id
        "recver"    :"recver id", //user id
        "createTs"  :0,//create timestamp for this msg
    }
}

//response
{
    "retCode":0, //0:success,others:fail
    "retMsg":"description for ${retCode}",
    "msgId":"msg id"
}
```

### sync user info

```javascript
//cmd=0x2007
//server->client

//request
{
    "userList":[{
        "account":"user id",
        "product_id":"product id",
        "sid":"session id",
        "cid":"channel id",
        "is_forward":true, //is forward by other agent,true:yes,false:no
        "forward_from":"" //forward by
    }]
}

//response
{
    "retCode":0, //0:success,others:fail
    "retMsg":"description for ${retCode}"
}
```

### agent close channel

```javascript
//cmd=0x2008
//client->server

//request
{
    "cid": "channel id",
    "sid": "session id"
}

//response
{
    "retCode":0, //0:success,others:fail
    "retMsg":"description for ${retCode}"
}
```

### channel been close

```javascript
//cmd=0x100B
//server->client

//request
{
    "cid": "channel id",
    "sid": "session id",
    "retCode":0, //0:success,others:fail
    "retMsg":"description for ${retCode}"
}

//response
{
    "retCode":0, //0:success,others:fail
    "retMsg":"description for ${retCode}",
    "cid": "channel id",
    "sid": "session id"
}
```

### history msg

```javascript
//cmd=0x2009
//client->server

//request
{
    "cid": "channel id",
    "sid": "session id",
    "lastMsgId": "last msg id, null mean latest msg",
    "msgNum": 0, //msg num want to fetch
}

//response
{
    "retCode":0, //0:success,others:fail
    "retMsg":"description for ${retCode}",
    "msgList":[{
        "msgId"     :"msg id",
        "msgType"   :0, //"0:text,1:audio,2:pic",
        "msgHeader" :"head msg",
        "msgFooter" :"footer msg",
        //body
        "msgText"   :"content when msgType==0",//此处格式为[]string|string，若是[]string,代表是一个列表
        "imageUrl"  :"url when msgType=2",
        "sender"    :"sender id", //agent id
        "recver"    :"recver id", //user id
        "createTs"  :0,//create timestamp for this msg
    }]
}
```

### agent list

```javascript
//cmd=0x2010
//client->server

//request
{
    "product_id":"account",
    "cid": "channel id",
    "supporter_id": "agent id"
}

//response
{
    "retCode":0, //0:success,others:fail
    "retMsg":"description for ${retCode}",
    "agentList":[{
        "product_id":"product id",
        "agentID":"agent id",
        "agentStatus":0, //0:none,1:online,2:offline,3:hold
        "userNum":0 //user num this agent service
    }]
}
```

### agent want to forward user to another agent

```javascript
//cmd=0x2011
//client->server

//request
{
    "sid":"session id",
    "channel_id":"channel id",
    "supporter_id": "agent id",
    "user_id": "user id",
    "forward_to_supporter_id": "agent to forward",
}

//response
{
    "retCode":0, //0:success,others:fail
    "retMsg":"description for ${retCode}",
    "forward_info":{
        "forward_id"        :"forward  id",
        "forward_status"    :0, //0:none,1:wait for response,2:success,3:fail
        "from_supporter_id" :"from which agent",
        "to_supporter_id"   :"to which agent",
        "channel_id"        :"channel id",
        "user_id"           :"user id",
        "create_ts"         :0 //create timestamp
    }
}
```

### notify agent ready to recver user from another agent

```javascript
//cmd=0x2012
//client->server

//request
{
    "sid":"session id",
    "supporter_id": "agent id"
}

//response
{
    "retCode":0, //0:success,others:fail
    "retMsg":"description for ${retCode}",
    "forward_info_list":[{
        "forward_id"        :"forward  id",
        "forward_status"    :0, //0:none,1:wait for response,2:success,3:fail
        "from_supporter_id" :"from which agent",
        "to_supporter_id"   :"to which agent",
        "channel_id"        :"channel id",
        "user_id"           :"user id",
        "create_ts"         :0 //create timestamp
    }]
}
```

### forward result

```javascript
//cmd=0x2015
//server->client(from_supporter_id and to_supporter_id)

//request
{
    "forward_info":{
        "forward_id"        :"forward  id",
        "forward_status"    :0, //0:none,1:wait for response,2:success,3:fail
        "from_supporter_id" :"from which agent",
        "to_supporter_id"   :"to which agent",
        "channel_id"        :"channel id",
        "user_id"           :"user id",
        "create_ts"         :0 //create timestamp
    }
}

//response
{
    "retCode":0, //0:success,others:fail
    "retMsg":"description for ${retCode}",
}
```

### err happen

```javascript
//cmd=0x2016
//server->client, if msg.text or msg.picurl is sensitive

//request
{
    "cid": "channel id",
    "sid": "session id",
    "msgId":"which msg err",
    "retCode":0, //0:success,others:fail
    "retMsg":"description for ${retCode}",
    "otherInfo": "other info"
}

//response
{
    "retCode":0, //0:success,others:fail
    "retMsg":"description for ${retCode}"
}
```


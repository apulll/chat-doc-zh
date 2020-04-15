# API 文档

{% api-method method="post" host="http://chatbot.myqcloud.com" path="/dev/svrapi/ask" %}
{% api-method-summary %}
msg for voice app
{% endapi-method-summary %}

{% api-method-description %}
This endpoint allows you to get free cakes.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-body-parameters %}
{% api-method-parameter name="language" type="string" required=false %}
language for this productID,zh\|en
{% endapi-method-parameter %}

{% api-method-parameter name="productID" type="string" required=true %}
product id
{% endapi-method-parameter %}

{% api-method-parameter name="userID" type="string" required=true %}
user id
{% endapi-method-parameter %}

{% api-method-parameter name="text" type="string" required=true %}
user question
{% endapi-method-parameter %}

{% api-method-parameter name="createTs" type="integer" required=true %}
create timestamp, e.g. 1586936603
{% endapi-method-parameter %}

{% api-method-parameter name="sign" type="string" required=true %}
md5sum\(${secretKey}&createTs=${createTs}&language=${language}&productID=${productID}&text=${text}&userID=${userID}\)
{% endapi-method-parameter %}
{% endapi-method-body-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}
Cake successfully retrieved.
{% endapi-method-response-example-description %}

```
{
  "code": 0,
  "msg": "",
  "replyType": 0,
  "replyText": "replyType=1,reply content",
  "replyMenu": [
    "replyType=2,reply list content",
    "replyType=2,reply list content"
  ]
}
```
{% endapi-method-response-example %}

{% api-method-response-example httpCode=404 %}
{% api-method-response-example-description %}
Could not find a cake matching this query.
{% endapi-method-response-example-description %}

```
{
  "code": 1,
  "msg": "err msg",
  "replyType": 0,
  "replyText": "",
  "replyMenu": []
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}




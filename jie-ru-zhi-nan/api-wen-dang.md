# API document

{% api-method method="post" host="https://api.cakes.com" path="/v1/cakes/:id" %}
{% api-method-summary %}
msg for chat
{% endapi-method-summary %}

{% api-method-description %}
This endpoint allows you to get free cakes.  
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-body-parameters %}
{% api-method-parameter name="sign" type="string" required=true %}
md5sum\(${secretKey}&ksort\(${args}\)
{% endapi-method-parameter %}

{% api-method-parameter name="create Ts" type="integer" required=true %}
create timestamp
{% endapi-method-parameter %}

{% api-method-parameter name="text" type="string" required=true %}
user questions
{% endapi-method-parameter %}

{% api-method-parameter name="userID" type="string" required=true %}
user id
{% endapi-method-parameter %}

{% api-method-parameter name="productID" type="string" required=true %}
product id
{% endapi-method-parameter %}

{% api-method-parameter name="language" type="string" required=false %}
language for this product ID
{% endapi-method-parameter %}
{% endapi-method-body-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}
Cake successfully retrieved.
{% endapi-method-response-example-description %}

```
{    "name": "Cake's name",    "recipe": "Cake's recipe name",    "cake": "Binary cake"}
```
{% endapi-method-response-example %}

{% api-method-response-example httpCode=404 %}
{% api-method-response-example-description %}
Could not find a cake matching this query.
{% endapi-method-response-example-description %}

```
{    "message": "Ain't no cake like that."}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}




---
title: CORS跨域资源共享
date: 2019/03/09 13:09
categories:
- Web
tags:
- CROS
- 跨域
---

## 简介

> 跨域资源共享(CORS)标准新增了一组 HTTP 首部字段,允许服务器声明哪些源站有权限访问哪些资源.另外,规范要求,`对那些可能对服务器数据产生副作用的HTTP 请求方法(特别是 GET 以外的 HTTP 请求,或者搭配某些 MIME 类型的 POST 请求)`,浏览器必须首先使用 OPTIONS 方法发起一个预检请求(preflight request),从而获知服务端是否允许该跨域请求.服务器确认允许之后,才发起实际的 HTTP 请求.在预检请求的返回中,服务器端也可以通知客户端,是否需要携带身份凭证(包括 Cookies 和 HTTP 认证相关数据).

## 两种请求

浏览器将CORS请求分成两类:简单请求和非简单请求

只要满足一下两个条件,就属于简单请求:

> 1. 请求方法是以下三种方法之一:
>    - HEAD
>    - GET
>    - POST
> 2. HTTP的`header`信息不超出以下几种字段
>    - Accept
>    - Accept-Language
>    - Content-Language
>    - Last-Event-ID
>    - Content-Type只限于:`application/x-www-form-urlencoded`,`multipart/form-data`,`text/plain`

不满足以上条件的就属于非简单请求

### 简单请求

对于简单请求,浏览器直接发出CORS请求.在`RequestHeader`中添加`Origin`字段

`Origin`字段用来说明本次请求的`请求源(协议+域名+端口)`.服务器根据这个值决定是否同意这次请求

如果`Origin`指定的请求源不在许可范围内,服务器会返回一个正常的`HTTP`回应.这个响应的`header`信息中没有包含`Access-Control-Allow-Origin`字段,浏览器发现后就知道出错.从而抛出一个,然后被``XMLHttpRequest`的`onerror`回调函数捕获.

> 这种错误无法通过状态码识别,因为是一个正常的HTTP回应

如果`Origin`指定的域名在许可范围内,服务器返回的响应会多出几个字段.

```
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: FooBar
Content-Type: text/html; charset=utf-8
```

以`Access-Control-`的字段与CORS请求相关

- Access-Control-Allow-Origin

该字段是必须的.它的值是请求时`Origin`字段的值,或者为`*`表示接受任意域名的请求.

- Access-Control-Allow-Credentials

该字段可选,它的值是一个布尔值,表示是否允许发送Cookies.**默认情况下,Cookies不包括在CORS请求之中**.设置为`true`则Cookie可以包含在请求中,一起发送给服务器.如果服务器不要浏览器发送 Cookie,删除该字段即可.

> Cookie中携带有Session Id相关信息,所以在请求一些需要得到Session进行身份验证的服务时,需要加上这个字段设置为true.比如:登录操作.
>
> 而客户端也需要在Ajax请求中打开`withCredentials`属性.
>
> ```javascript
>     xhrFields: {
>         withCredentials: true
>     },
>     crossDomain: true
> ```
>
> **注意:**如果要发送Cookie,`Access-Control-Allow-Origin`就不能为`*`.必须明确指定与网页一致的域名.Cookie也遵循同源策略,只有用服务器域名设置的Cookie才会上传,其他域名的Cookie不会上传.(防止了伪造Cookie的攻击手段)

- Access-Control-Expose-Headers

该字段可选.CORS请求时,`XMLHTTPRequest`对象的`getResponseHeader()`方法只能拿到6个基本字段:`Cache-Control`、`Content-Language`、`Content-Type`、`Expires`、`Last-Modified`、`Pragma`.如果想要拿到其他的字段,就需要在此字段中指定.上面的例子中指定,`getResponseHeader('FooBar')`可以返回`FooBar`字段的值

### 非简单请求

#### 预检请求

非简单请求是对服务器有特殊要求的请求,如请求方法为`PUT`或`DELETE`,或者`Content-Type`字段的类型是`application/json`

非简单请求的CORS请求,会在正式通信之前,增加一次HTTP查询请求,即**预检请求**.

浏览器查询服务器,当前网页所在的域名是否在服务器的许可名单之中,以及可以使用哪些HTTP动词和头信息字段.只有在得到肯定答复后,浏览器才会发出正式的`XMLHttpRequest`请求,否则就报错

```javascript
var url = 'http://api.alice.com/cors';
var xhr = new XMLHttpRequest();
xhr.open('PUT', url, true);
xhr.setRequestHeader('X-Custom-Header', 'value');
xhr.send();
```

上面构造了一个非简单请求.请求方法是`PUT`,并且发送一个自定义字段`X-Custom-Header`

浏览器就会自动发出一个`预检请求`.确认服务器是否接受这样的请求.预检请求的HTTP头信息如下

```
OPTIONS /cors HTTP/1.1
Origin: http://api.bob.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```

预检请求的请求方法是`OPTIONS`,表示这个请求是用来查询的.头信息里,关键字段是`Origin`,表示请求源

- Access-Control-Request-Method

该字段是必须的,用来列出浏览器的CORS请求会用到哪些HTTP方法,这里是`PUT`

- Access-Control-Request-Headers

该字段可以有多个值,用`,`分隔.指定浏览器CORS请求会额外发送的头信息字段,这里是`X-Custom-Header`

#### 预检请求的回应

服务器收到预检请求后,检查`Origin`、`Access-Control-Request-Method`和`Access-Control-Request-Headers`字段以后,确认是否允许跨域请求,并作出回应

```

HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1728000
Content-Type: text/html; charset=utf-8
Content-Encoding: gzip
Content-Length: 0
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain
```

上面的HTTP回应中,关键的是`Access-Control-Allow-Origin`字段,表示`http://api.bob.com`可以请求数据.如果为`*`表示接受任意域名的跨域请求.

如果服务器否定了预检请求,会返回一个正常的HTTP回应,但是没有任何CORS相关的头信息字段.这是浏览器会认为服务器不同意预检请求,便会出发一个错误,被`XMLHttpRequest`对象的`onerror`回调函数捕获.控制台会打印出如下的报错信息:`XMLHttpRequest cannot load http://api.alice.com.Origin http://api.bob.com is not allowed by Access-Control-Allow-Origin.`

- Access-Control-Allow-Methods

该字段必须,可以有多个值,用`,`分隔.表明服务器支持的**所有**跨域请求的方法.返回所有是为了避免多次预检请求.

- Access-Control-Allow-Headers

如果浏览器请求包括`Access-Control-Request-Headers`字段,则`Access-Control-Allow-Headers`字段是必须的.也可以由多个值,用`,`分隔.表明服务器至支持的**所有**头信息字段,不限于浏览器在预检请求中请求的字段.

- Access-Control-Allow-Credentials

同简单请求.

- Access-Control-Max-Age

  该字段可选.用来指定本次预检请求的有效期,单位为秒.在有效期内,不用发出另一条预检请求.

#### 服务器正常请求和回应

一旦服务器通过了预检请求,以后每次浏览器正常的CORS请求就会和简单请求一样,会有一个`Origin`头信息字段,服务器的回应也会有一个`Access-Control-Allow-Origin`头信息字段.
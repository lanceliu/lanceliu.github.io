---
layout: post
title:  "jQuery请求BODY数据格式"
date:   2016-03-31 23:14:52
categories: http jquery
published: true
comments: true
thread: 20160420224352555
---

发现一个特别有意思的问题, JS发起ajax请求时，参数设置的位置不同，在服务器有可能getParamter时取不到。

## 源码一：
```html5
<html>
  <head>
    <script type="text/javascript" src="jquery.js"></script>
    <script type="text/javascript">
      $(document).ready(function(){
        $(document).ajaxSend(function(e, req, settings) {
            settings.data = "hello=world"
            console.log("ajaxSend")
          });
        $("button").click(function(){
          $.ajax({
            method:"post",
               url: "index.jsp"
           })
        });
      });
    </script>
  </head>
  <body>
    <button>改变内容</button>
  </body>
</html>
```

点击页面按钮【改变内容】，打开chrome开发者工具发现headers中如下，
content-type=text/plain,并且request方式为payload。
这时服务器端getParamter得到值为空，只能够通过读取request的输入流来获取。
```html5
Response Headers
Content-Type:text/html;charset=UTF-8
Date:Wed, 20 Apr 2016 14:36:15 GMT
Server:Apache-Coyote/1.1
Transfer-Encoding:chunked
Request Headers
view source

Request Headers
Accept:*/*
Accept-Encoding:gzip, deflate
Accept-Language:zh-CN,zh;q=0.8,en;q=0.6
Connection:keep-alive
Content-Length:11
Content-Type:text/plain;charset=UTF-8
Cookie:JSESSIONID=1CC8EEE67B6FD157AD5621075DA4DBE5
Host:localhost:8080
Origin:http://localhost:8080
Referer:http://localhost:8080/test.html
User-Agent:Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/47.0.2526.106 Safari/537.36
X-Requested-With:XMLHttpRequest

Request Payload
hello=world
```


## 源码二：
```html5
<html>
  <head>
    <script type="text/javascript" src="jquery.js"></script>
    <script type="text/javascript">
      $(document).ready(function(){
        $(document).ajaxSend(function(e, req, settings) {
            console.log("ajaxSend")
          });
        $("button").click(function(){
          $.ajax({
            method:"post",
            data:"hello=world",
               url: "index.jsp"
           })
        });
      });
    </script>
  </head>
  <body>
    <button>改变内容</button>
  </body>
</html>
```

点击页面按钮【改变内容】，打开chrome开发者工具发现headers中如下，
content-type=application/x-www-form-urlencoded,并且request方式为Form Data。
这时服务器端getParamter得到值为world，当然也可以通过读取request的输入流。
```html5
Response Headers
view source
Content-Type:text/html;charset=UTF-8
Date:Wed, 20 Apr 2016 14:50:37 GMT
Server:Apache-Coyote/1.1
Transfer-Encoding:chunked

Request Headers
view source
Accept:*/*
Accept-Encoding:gzip, deflate
Accept-Language:zh-CN,zh;q=0.8,en;q=0.6
Connection:keep-alive
Content-Length:11
Content-Type:application/x-www-form-urlencoded; charset=UTF-8
Cookie:JSESSIONID=1CC8EEE67B6FD157AD5621075DA4DBE5
Host:localhost:8080
Origin:http://localhost:8080
Referer:http://localhost:8080/test.html
User-Agent:Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/47.0.2526.106 Safari/537.36
X-Requested-With:XMLHttpRequest

Form Data
hello:world
```

## 答疑
（Content-Type不是application/x-www-form-urlencoded）数据格式不固定，不一定是名值对的方式，所以服务器无法知道具体的处理方式，所以只能通过获取原始数据流的方式来进行解析。

那jQuery通过xhr发送请求时是如何设置content-type的呢？我们以jQuery1.12.3为例解释：

片段一：jQuery.ajax  line 9688~9750
```javascript
// 发起请求时如果有设定data并且没有设定contentType，则设定请求contentType=ajaxSettings的默认设置 "application/x-www-form-urlencoded; charset=UTF-8"。 设定到ajax方法中的局部变量responseHeaders当中。

.......
// 调用用户的beforeSend处理事件
if ( s.beforeSend &&
  ( s.beforeSend.call( callbackContext, jqXHR, s ) === false || state === 2 ) ) {
  // Abort if not done already and return
  return jqXHR.abort();
}

.......
  // 调用全局处理事情ajaxSend事件处理函数
  if ( fireGlobals ) {
    globalEventContext.trigger( "ajaxSend", [ jqXHR, s ] );
  }
.......
    // 调用jQuery.ajaxTransport.options.send方法
    transport.send( requestHeaders, done );
......
```

片段二： jQuery.ajaxTransport.options.send   line 10201~
```javascript
send: function( headers, complete ) {
..........
    // 设定Content-Type如果上面调用代码传过来的话
		for ( i in headers ) {
			if ( headers[ i ] !== undefined ) {
				xhr.setRequestHeader( i, headers[ i ] + "" );
			}
		}
		// 向服务器发起请求
		xhr.send( ( options.hasContent && options.data ) || null );
.......
```


---
参考：
[AJAX POST请求中参数以form data和request payload形式在servlet中的获取方式](http://blog.csdn.net/mhmyqn/article/details/25561535)

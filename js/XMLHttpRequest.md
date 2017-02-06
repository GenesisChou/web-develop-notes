# XHR对象
使用XHR对象时，要调用的第一个方法是open(),它接受3个参数：要发送的请求的类型（"get","post"),请求的url和表示是否异步发送请求的布尔值。
```javascript
xhr.open('get','example.php',false);
```
关于这行代码，需要说明2点：
- url相对于执行代码的当前页面（也可以使用绝对路径）
- 调用open()方法并不会真正的发送请求，而只是启动一个请求以备发送。

**只能向同一个域中使用相同端口和协议的url发送请求。如果url与启动请求的页面有任何差别，都会引发安全错误。**
```javascript
    var xhr = new XMLHttpRequest();
    xhr.addEventListener('readystatechange',function(){
        if (xhr.readyState == 4) {
            if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304) {
                console.log(xhr.responseText);
            }
        }
    })
    xhr.open('get', 'http://localhost:3000/test', true);
    xhr.send(null);
```
**该例子没有使用this对象，原因是onnnreadystatechange事件处理程序的作用域问题。如果使用this对象，在有的浏览器中会导致函数执行失败，或者导致错误发生。**

为xhr添加事件readystatechange监听事件，可以检测xhr对象的readyState属性，属性表示请求／响应过程的当前活动阶段。这个属性可取的值如下。
- 0:未初始化。未调用open()
- 1:启动。已调用open()
- 2:发送。已调用send()，未接收到响应
- 3:接收。接收到部分响应数据。
- 4:完成。接收到全部响应数据。

收到响应后，响应的数据会自动填充xhr对象的属性，相关的属性简介如下。
- responseText:作为响应主体被返回的文本。
- responseXML:如果响应的内容类型是'text/xml'或'application/xml',这个属性中将保存包含着响应数据的XML DOM文档。
- status:响应的HTTP状态。
- statusText:HTTP状态的说明。

一般以status为200为成功标志，这时responseText属性的内容已经就绪。status为304表示请求资源没被修改，可以直接使用浏览器中缓存的版本。

## HTTP头部信息
每个HTTP请求和响应都会带有相应的头部信息。
在发送XHR请求的同时，还会发送下列同步信息。
- Accept
- Accept-Charset
- Accept-Encoding
- Accept-Language
- Connection
- Cookie
- Host
- Referer
- User-Agent

## get请求
GET是最常见的请求类型，最常用于向服务器查询某些信息。必要时，可以将查询字符串参数尾追加到URL的末尾。所有名-值对儿都必须由&分隔。
```javascript
xhr.open('get','url?name1=value1&name2=value2&name3=value3',true);
```
## post请求
使用频率仅此于GET的是POST请求，通常用于向服务器发送应该被保存的数据。POST请求应该把数据作为请求的主体提交，而GET请求传统上不是这样。POST请求的主体可以包含非常多的数据，而且格式不限。
```javascript
xhr.open('post','url',true);
xhr.setHeader('Content-Type','application/json');
//xhr.setHeader('Content-Type','application/x-www-form-urlencoed');
xhr.send(JSON.stringify({name1:value1,name2:value2}));
```
**与GET请求相比，POST请求消耗的资源会更多一些。从性能角度来看，以发送相同的数据记，GET请求的速度最多可达到POST请求的两倍**

## express获取http参数的几种方式

- 对于path中的变量，使用req.params
- 对于get请求的?xxxx=,使用req.query
- 对于post请求中的变量，需要先引入body-parse中间件
  ```javascript
  var bodyParser = require('body-parser');
  app.use(bodyParser.json());
  ```
  之后并可使用req.body获取参数
- 以上三种情形，均可以使用req.param()方法，所以说req.param()是req.query、req.body、以及req.params获取参数的三种方式的封装。
 
# XMLHttpRequest 2
## FormData
## 超时设定
IE8为XHR对象添加了一个timeout属性，表示请求在等待响应多少毫秒之后就终止。类似xhr.abort()。
```javascript
xhr.timeout=1000;
xhr.addEventListener('timeout',function(){
    console.log('time out');
})
```
## overrideMimeType方法
## 进度事件
ProgressEvents规范是W3C的一个工作草案，定义了与客户端服务器通信有关的事件。有以下6个进度事件
- loadstart:接收到响应数据的第一个字节时触发
- progress:接收期间持续不断的触发。
- error:错误时触发。
- abort:调用abort()时触发。
- load:接收到完整数据时触发，等同于readystatechange事件触发后xhr.readyState为4
- loadend:通信完成或触发error,abort,load时触发。

## 跨资源共享
### IE对CORS的实现
使用XDR对象（XDomainRquest）
- 不发送cookie,不响应cookie
- 只能设置Content-Type字段
- 不能反问响应头部信息
- 只支持get与post请求

### 其他浏览器对CORS的实现
在xhr.open()时传入绝对URL即可实现，但有以下限制
- 不能使用setRequestHeader()设置自定义头部
- 不能发送和接收cookie
- 调用getAllResponseHeader()方法总会返回空字符串

### Preflighted Requests
CORS通过一种叫Preflighted Requests的透明服务器验证机制支持开发人员使用自定义的头部，GET或POST的方法，以及不同类型的主体内容。在使用下列高级选项来发送请求时，就会向服务器发送一个Preflight请求。这种请求使用OPTIONS方法，发送下列头部。
- Origin:与简单的请求相同。
- Access-Control-Request-Method:请求自身使用的方法。
- Access-Control-Request-Headers:(可选)自定义的头部信息，多个头部以逗号分隔，以下是一个带有自定义头部NCZ的使用POST方法发送的请求。

Preflight请求的浏览器包括Firefox 3.5+,Safari 4+和Chrome，IE 10以及更早版本都不支持。
### 带凭据的请求
### 跨浏览器的CORS
##其他跨域技术
### 图像ping
### JSONP
JSONP是通过动态<script>元素使用的。
```javascript
function handleResponse(response){
    console.log(response);
}
var script=document.createElement('script');
script.src='http://xxx?callback=handleResponse';
document.body.insertBefore(script,document.body.firstChild);
```
缺陷
- JSONP是从其他域加载代码执行的。不安全。
- 确定JSONP是否失败不容易。
### Comet



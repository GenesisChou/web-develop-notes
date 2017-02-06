# 限制
cookie在性质上是绑定在特定的域名下的。当设定一个cookie后，再给创建它的域名发送请求时，都会包含这个cookie。这个限制确保了储存在cookie中的信息只能让批准的接受者访问，而无法被其他域访问。
每个域的cookie总数是有限的，各浏览器之间各有不同。
- Firefox 50
- Opera 30
- Safari,Chrome 无限制
cookie的长度最好限制在4095B以内，超出的部分会丢失掉。

# 构成
- 名称：唯一的。不区分大小写的。必须是URL编码。
- 值：储存在cookie中的字符串值。必须是URL编码。
- 域：所有向指定域都包含这个cookie信息。若没有明确设定，那么这个域会被认作来自设置cookie的那个域。
- 路径：对于指定域中的那个路径，应该向服务器发送cookie。例如，你可以指定cookie只有从http://www.wrox.com/books/中才能访问，那么http://www.wrox.com的页面酒不会发送cookie信息，即使请求都是来自同一个域的。
- 失效时间：默认情况下，浏览器会话结束时删除所有cookie；也可以自己设置删除时间。值是GMT格式的日期。若设置的时间早于当前时间，则cookie会被立刻删除。
- 安全标志：指定后，cookie只有在使用SSL连接的时候才发送到服务器。例如，cookie信息只能发送给https://www.wrox.com，而http://www.wrox.com的请求则不能发送cookie.
每段信息都作为Set-Cookie头的一部分，使用分号加空格分隔每一段。例：
```
Set-Cookie:name=value;expires=Mon,22-Jan-07 07:10:24 GMT;domain=.wrox.com
```
该头信息指定了一个叫做name的cookie,它会在格林威治时间2007年1月22日 7:10:24 实效，同时于www.wrox.com和wrox.com的任何子域（p2p.wrox.com）都有效。
```
Set-Cookie:name=value;domain=.wrox.com;PATH=/;SECURE
```
因为设置了secure标志，这个cookie只能通过SSL连接才能传输。
**注意**，域，路径，实效时间和secure标志都是服务器给浏览器的指示，以指定何时应该发送cookie。这些参数并不会作为发送到服务器的cookie信息的一部分，只有名值对儿才会被发送。
# javascript中的cookie
使用document.cookie返回当前页面可用的所有cookie的字符串，一系列由分号隔开的名值对儿，如下。
```
name1=value1;name2=value2;name3=value3
```
所有名和值都经过URL编码，必须使用decodeURLComponent()来解码。
设置cookie的格式如下，和Set-Cookie头中使用的格式一样。
```
name=value;expires=expiration_time;path=doman_path;domain=domain_name;secure
```
只有name与value是必须的
```javascript
document.cookie=encodeURLComponent('name')+'='+encodeURLComponent('Nicholas');
```
# 子cookie
为了绕开cookie数限制，开发人员使用一种称为子cookie(subcookie)的概念。利用value来存储多个名值对儿。
```
name=name1=value1&name2=value2&name3=value3
```
# cookie的思考
所有cookie都会由浏览器作为请求头发送，应尽可能在cookie中少存储信息，以避免影响性能。
**cookie数据并非存储在安全环境中，可被他人访问。所以不要在cookie中存储重要数据**
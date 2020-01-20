# 游览器是如何工作的1

#### 游览器的工作过程：

1、游览器首先使用HTTP协议或者HTTPS协议，向服务端发送请求。

2、把请求回来的HTML代码经过解析，构成DOM树。

3、计算DOM树上的CSS属性。

4、最后根据CSS属性对元素逐个进行渲染，得到内存中的位图。

5、一个可选步骤是对位图进行合成，这会极大的增加后续绘制的速度。

6、合成之后，再绘制到界面上。

简单描述就是：先HTTP请求到数据，然后构建DOM树、CSS计算、渲染、合成、绘制。我们可以把它理解成一个流水线，并不是等一个全部完成才会进行下一步的。

![Image text](https://github.com/zhangbqplus/doc/blob/master/img/qianduan/_1.jpg)

#### HTTP协议

协议就是约定。

游览器通过URL把数据取回来，取回数据使用的就是HTTP协议。

取回数据前还用到了DNS查询。（域名系统查询）

HTTP标准是由IETF组织制定的。

跟它相关的标准有两份：

[HTTP1.1]: https://tools.ietf.org/html/rfc2616
[HTTP1.1]: https://tools.ietf.org/html/rfc7234

HTTP协议是基于TCP协议出现的。

TCP协议是一条双向通讯通道，HTTP在TCP的基础上规定了Request-Response的模式。这个模式决定了通讯必定是由游览器端首先发起。

游览器实现者只需要一个TCP库，甚至一个现成的HTTP库就可以搞定游览器的网络通讯部分。

HTTP是纯粹的文本协议，它规定了使用TCP协议来传输文本格式的一个应用层协议。

#### HTTP协议格式

假使你通过telnet客户端请求一个网站的话结构分为两部分：

请求部分request line分为：HTTP Method （请求的方法），请求的路径，请求的协议和版本。

相应部分response line 分为：协议和版本，状态码，状态文本

结构如图：

![Image text](https://github.com/zhangbqplus/doc/blob/master/img/qianduan/_2.jpg)

path 是请求路径，完全由服务端定义。

version 几乎都是固定字符

response body 是我们比较熟的HTML

#### HTTP Method (方法)

GET： 请求指定的页面信息，并返回实体主体

HEAD： 只请求页面的首部，和GET请求相同，但不返回消息体。

POST： 请求服务器接受所指定的文档作为对所标识的URI的新的从属实体。

PUT： 从客户端向服务器传送的数据取代指定的文档的内容。

DELETE： 请求服务器删除指定的页面。

OPTIONS： 允许客户端查看服务器的性能。

TRACE： 请求服务器在响应中的实体主体部分返回所得到的内容。

PATCH： 实体中包含一个表，表中说明与该URI所表示的原内容的区别。

MOVE： 请求服务器将指定的页面移至另一个网络地址。

COPY： 请求服务器将指定的页面拷贝至另一个网络地址。

LINK： 请求服务器建立链接关系。

UNLINK： 断开链接关系。

WRAPPED： 允许客户端发送经过封装的请求。

Extension-mothed：在不改动协议的前提下，可增加另外的方法。

#### HTTP Status code（状态码）和 Status text（状态文本）

###### 1xx：临时回应，表示客户端请继续。（指示信息--表示请求已接收，继续处理）

###### 2xx：请求成功。（成功--表示请求已被成功接收、理解、接受。）

200：请求成功。

###### 3xx: 表示请求的目标有变化，希望客户端进一步处理。（重定向--要完成请求必须进行更进一步的操作。）

301&302：永久性与临时性跳转。

304：跟客户端缓存没有更新。

###### 4xx：客户端请求错误。（客户端错误--请求有语法错误或请求无法实现。）

400 Bad Request：客户端请求有语法错误，不能被服务器所理解。

401 Unauthorized：请求未经授权，这个状态代码必须和WWW-Authenticate报头域一起使用。

403  Forbidden：服务器收到请求，但是拒绝提供服务。即：无权限。

404  Not Found：请求资源不存在，举个例子：输入了错误的URL。

418：It’s a teapot. 这是一个彩蛋，来自 ietf 的一个愚人节玩笑。（超文本咖啡壶控制协议）

###### 5xx：服务端请求错误。（服务器端错误--服务器未能实现合法的请求。）

500  Internal Server Error：服务器发生不可预期的错误。

503  Server Unavailable：服务器当前不能处理客户端的请求，一段时间后可能恢复正常，举个例子：HTTP/1.1 200 OK（CRLF）。

对我们前端来说：

1xx 系列的状态码是非常陌生的，原因是 1xx 的状态被浏览器 HTTP 库直接处理掉了，不会让上层应用知晓。

2xx 系列的状态最熟悉的就是 200，这通常是网页请求成功的标志。

3xx 系列比较复杂，301 和 302 两个状态表示当前资源已经被转移，只不过一个是永久性转移，一个是临时性转移。实际上 301 更接近于一种报错，提示客户端下次别来了。

304  一个神奇的状态码，产生这个状态的前提是：客户端本地已经有缓存的版本，并且在 Request 中告诉了服务端，当服务端通过时间或者 tag，发现没有更新的时候，就会返回一个不含 body 的 304 状态。

#### HTTP Head (HTTP 头)

HTTP 头可以看作一个键值对。

HTTP 头也是一种数据，我们可以自由定义 HTTP 头和值。不过在 HTTP 规范中，规定了一些特殊的 HTTP 头。

在 HTTP 标准中，有完整的请求 / 响应头规定，这里我们挑几个重点的说一下：

Request Header

![Image text](https://github.com/zhangbqplus/doc/blob/master/img/qianduan/_3.jpg)

Response Header

![Image text](https://github.com/zhangbqplus/doc/blob/master/img/qianduan/_4.jpg)

#### HTTP Request Body

HTTP 请求的 body 主要用于提交表单场景。实际上，HTTP 请求的 body 是比较自由的，只要浏览器端发送的 body 服务端认可就可以了。一些常见的 body 格式是：

application/json

application/x-www-form-urlencoded

multipart/form-data

text/xml

我们使用 HTML 的 form 标签提交产生的 HTML 请求，默认会产生 application/x-www-form-urlencoded 的数据格式，当有文件上传时，则会使用 multipart/form-data。

#### HTTPS

在 HTTP 协议的基础上，HTTPS 和 HTTP2 规定了更复杂的内容，但是它基本保持了 HTTP 的设计思想，即：使用上的 Request-Response 模式。

HTTPS 有两个作用，一是确定请求的目标服务端身份，二是保证传输的数据不会被网络中间节点窃听或者篡改。

HTTPS 的标准也是由 RFC 规定的，你可以查看它的详情链接：
https://tools.ietf.org/html/rfc2818

HTTPS 是使用加密通道来传输 HTTP 的内容。但是 HTTPS 首先与服务端建立一条 TLS 加密通道。TLS 构建于 TCP 协议之上，它实际上是对传输的内容做一次加密，所以从传输内容上看，HTTPS 跟 HTTP 没有任何区别。

#### HTTP 2

HTTP 2 是 HTTP 1.1 的升级版本，你可以查看它的详情链接。
https://tools.ietf.org/html/rfc7540

HTTP 2.0 最大的改进有两点，一是支持服务端推送，二是支持 TCP 连接复用。

服务端推送能够在客户端发送第一个请求到服务端时，提前把一部分内容推送给客户端，放入缓存当中，这可以避免客户端请求顺序带来的并行度不高，从而导致的性能问题。

TCP 连接复用，则使用同一个 TCP 连接来传输多个 HTTP 请求，避免了 TCP 连接建立时的三次握手开销，和初建 TCP 连接时传输窗口小的问题。

需要注意的是： 很多优化涉及更下层的协议。IP 层的分包情况，和物理层的建连时间都是需要被考虑的。
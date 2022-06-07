- POST和GET都是HTTP请求的基本方法
### 主要区别
#### 功能不同
- get是从服务器上获取数据。
- post是向服务器传送数据。
#### 过程不同
- get是把参数数据队列加到提交表单的ACTION属性所指的URL中，值和表单内各个字段一一对应，在URL中可以看到。
- post是通过HTTP post机制，将表单内各个字段与其内容放置在HTML HEADER内一起传送到ACTION属性所指的URL地址。用户看不到这个过程。
#### 获取值不同
- 对于get方式，服务器端用Request.QueryString获取变量的值。
- 对于post方式，服务器端用Request.Form获取提交的数据。
#### 传送数据量不同
- get传送的数据量较小，不能大于2KB。
- post传送的数据量较大，一般被默认为不受限制。但理论上，IIS4中最大量为80KB，IIS5中为100KB。
#### 安全性不同
- get安全性非常低
- post安全性较高

### URL解析过程
#### 1、用户输入网址，浏览器发起DNS查询请求
- 用户访问网页，DNS服务器（域名解析系统）会根据用户提供的域名查找对应的IP地址
#### 2、建立TCP连接
- 浏览器通过DNS查询到web服务器的真实IP地址后，便向服务器发起TCP连接请求，通过TCP的三次握手建立连接后，浏览器便可以将数据通过http请求发送给服务器了
#### 3、浏览器向web服务器发送一个http请求
- http请求是基于TCP协议之上的应用层协议--超文本传输协议。
- http请求包括：
- 请求行：请求方法 url 协议/版本 
- 请求头：数据以键值对的形式存放，其中包括Host、Connection、Accept、Accept-Encoding等
- 请求空行
- 请求体：一般包含请求的数据
#### 4、发送响应数据给客户端
- web服务器通过监听80端口来获取客户端的http请求。
- 与客户端建立连接后，web服务器开始接受客户端发来的数据，并通过http解码，从接收到的网络数据中解析出请求的url信息，如Accept-Encoding等。web服务器将根据http请求的信息，响应相应的数据给客户端，典型的http响应数据如下：
- 状态行：协议/版本 状态码 状态描述
- 响应头：数据以键值对的形式存放，其中包括Location、Server、Content-Type、Content-Length等
- 响应空行
- 响应体
- 至此，一个HTTP通信完成。服务器会根据http请求头中的Connection字段值决定是否关闭TCP连接通道，若其值为keep-alive时，web服务器不会立即关闭连接
#### 5、浏览器解析http响应
- html文档解析(DOM Tree)
- 浏览器发送获取嵌入到html中的对象
- css解析 、js解析

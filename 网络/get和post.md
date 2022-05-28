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

# python 实现http服务

标签（空格分隔）： python

---

python可以方便的实现http服务, 有以下三种方式

- `BaseHTTPServer` : 提供基本的Web服务和处理器类, 分别是`HTTPServer`和`BaseHTTPRequestHandler`
- `SimpleHTTPServer` : 包含执行GET和HEAD请求的`SimpleHTTPRequestHandler`
- `CGIHTTPServer` : 包含执行POST请求和执行的`CGITTPRequestHandler`

BaseHTTPServer是一个HTTP服务器库。它理解HTTP协议并让你的代码处理请求。它自己没有任何“逻辑”。

SimpleHTTPServer构建于BaseHTTPServer之上，以类似于普通HTTP服务器的方式处理请求，即从文件系统提供文件


# SimpleHTTPServer

可以完成一个简单的内建 HTTP 服务器。于是，你可以把你的目录和文件都以HTTP的方式展示出来

```
$ cd /home/haoel
$ python -m SimpleHTTPServer
```

这就行了，而我们的HTTP服务在8000号端口上侦听。你会得到下面的信息：

```
Serving HTTP on 0.0.0.0 port 8000 ...
```

你可以打开你的浏览器（IE或Firefox），然后输入下面的URL：
`http://192.168.1.1:8000`
如果你的目录下有一个叫 index.html 的文件名的文件，那么这个文件就会成为一个默认页，如果没有这个文件，那么，目录列表就会显示出来。

如果你想改变端口号，你可以使用如下的命令：

```
$ python -m SimpleHTTPServer 8080
```

# BaseHTTPServer

主要包含两个类`HTTPServer`和`BaseHTTPRequestHandler

- `HTTPServer`: 继承`SocketServer.TCPServer`，用于获取请求，并将请求分配给应答程序处理
- `BaseHTTPRequestHandler`: 继承`SocketServer.StreamRequestHandler`，对http连接的请求作出应答（response）

基于BaseHTTPServer 的Http Server的处理流程：

- 1.HTTPServer绑定对应的应答类（BaseHTTPRequestHandler ）
```
http_server = HTTPServer(('', int(port)), ServerHTTP)
```
- 2.监听端口：
```
http_server.serve_forever() 
```
> serve_forever()方法使用select.select()循环监听请求，当接收到请求后调用当监听到请求时，取出请求对象

- 3.应答：
创建新线程以连接对象(开始理解成请求对象)为参数实例化应答类:ServerHTTP()
应答类根据请求方式调用`ServerHTTP.do_XXX`处理方法

常用方法/属性：

```
BaseHTTPRequestHandler.path                    #包含的请求路径和GET请求的数据
BaseHTTPRequestHandler.command                 #请求类型GET、POST...
BaseHTTPRequestHandler.request_version         #请求的协议类型HTTP/1.0、HTTP/1.1
BaseHTTPRequestHandler.headers                 #请求的头
BaseHTTPRequestHandler.responses               #HTTP错误代码及对应错误信息的字典
BaseHTTPRequestHandler.handle()                #用于处理某一连接对象的请求，调用handle_one_request方法处理
BaseHTTPRequestHandler.handle_one_request()    #根据请求类型调用do_XXX()方法，XXX为请求类型
BaseHTTPRequestHandler.do_XXX()                #处理请求
BaseHTTPRequestHandler.send_error()            #发送并记录一个完整的错误回复到客户端,内部调用send_response()方法实现
BaseHTTPRequestHandler.send_response()         #发送一个响应头并记录已接收的请求
BaseHTTPRequestHandler.send_header()           #发送一个指定的HTTP头到输出流。 keyword 应该指定头关键字，value 指定它的值
BaseHTTPRequestHandler.end_headers()           #发送一个空白行，标识发送HTTP头部结束
BaseHTTPRequestHandler.wfile    #self.connection.makefile('rb', self.wbufsize) self.wbufsize = -1 应答的HTTP文本流对象，可写入应答信息
BaseHTTPRequestHandler.rfile    #self.connection.makefile('wb', self.rbufsize) self.rbufsize = 0  请求的HTTP文本流对象，可读取请求信息

```




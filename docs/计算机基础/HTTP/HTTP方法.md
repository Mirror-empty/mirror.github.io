### HTTP方法

客户端发送的请求报文第一行为请求行，包含了方法字段。

### GET

    获取资源

当前网络请求中，绝大部分使用的是GET方法。

### HEAD

    获取报文首部

和GET方法类似，但是不返回报文实体主体部分。

主要用于确认URL的有效性以及资源更新的日期时间等。

### POST

    传输实体主体

POST主要用来传输数据，而GET主要用来获取资源。

更多区别下面有讲。

### PUT

    上传文件

由于自身不带验证机制，任何人都可以上传文件，因此存在安全性问题，一般不使用该方法。

```
PUT /new.html HTTP/1.1
Host: example.com
Content-type: text/html
Content-length: 16

<p>New File</p>
```


### PATCH(修正)

    对资源进行部分修改

PUT也可以用于修改资源，但是只能完全替代原始资源，PATCH允许部分修改。

```
PATCH /file.txt HTTP/1.1
Host: www.example.com
Content-Type: application/example
If-Match: "e0023aa4e"
Content-Length: 100

[description of changes]
```

### OPTIONS
    查询支持的方法

查询指定的URL能够支持的方法。

会返回Allow：Get，POST，HEAD，OPTIONS这样的内容。

一种预请求，在自定义

### CONNECT

    要求在于代理服务器通信时建立隧道

使用SSL（Secure Sockets Layer，安全套接层）和TLS（Transport Layer Security，传输层安全）协议把通信内容加密后经网络隧道传输。

```
CONNECT www.example.com:443 HTTP/1.1
```

![zvkoUs.png](https://s1.ax1x.com/2022/12/24/zvkoUs.png)

隧道代理

![zvkzVJ.png](https://s1.ax1x.com/2022/12/24/zvkzVJ.png)


### TRACE
    追踪路径

服务器会将通信路径返回给客户端。

发送请求时，在Max-Forwards首部自动中填入数值，每经过一个服务器就会减1，当数值为0时就停止传输。

通常不会使用TRACE，并且它容易受到XST攻击（Cross-Site Tracing，跨站追踪）。

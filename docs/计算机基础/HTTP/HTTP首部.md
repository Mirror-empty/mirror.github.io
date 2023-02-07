### HTTP首部

有 4 种类型的首部字段：通用首部字段、请求首部字段、响应首部字段和实体首部字段。

各种首部字段及其含义如下（不需要全记，仅供查阅）：

### 通用首部字段

|首部字段名|说明|
|--|--
|Cache-Control|	控制缓存的行为
Connection	|控制不再转发给代理的首部字段、管理持久连接
Date	|创建报文的日期时间
Pragma|	报文指令
Trailer|	报文末端的首部一览
Transfer-Encoding|	指定报文主体的传输编码方式
Upgrade|	升级为其他协议
Via|	代理服务器的相关信息
Warning|	错误通知


### 请求首部字段

|首部字段名|说明|
|--|--
Accept|	用户代理可处理的媒体类型
Accept-Charset	|优先的字符集
Accept-Encoding	|优先的内容编码
Accept-Language	|优先的语言（自然语言）
Authorization	|Web 认证信息
Expect	|期待服务器的特定行为
From	|用户的电子邮箱地址
Host	|请求资源所在服务器
If-Match	|比较实体标记（ETag）
If-Modified-Since	|比较资源的更新时间
If-None-Match	|比较实体标记（与 If-Match 相反）
If-Range|	资源未更新时发送实体 Byte 的范围请求
If-Unmodified-Since	|比较资源的更新时间（与 If-Modified-Since 相反）
Max-Forwards	|最大传输逐跳数
Proxy-Authorization	|代理服务器要求客户端的认证信息
Range	|实体的字节范围请求
Referer	|对请求中 URI 的原始获取方
TE	|传输编码的优先级
User-Agent|	HTTP 客户端程序的信息

```
GET http://www.example.com/ HTTP/1.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cache-Control: max-age=0
Host: www.example.com
If-Modified-Since: Thu, 17 Oct 2019 07:18:26 GMT
If-None-Match: "3147526947+gzip"
Proxy-Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 xxx

param1=1&param2=2
```

### 响应首部字段

|首部字段名|说明|
|--|--
首部字段名|	说明
Accept-Ranges|	是否接受字节范围请求
Age	|推算资源创建经过时间
ETag|	资源的匹配信息
Location	|令客户端重定向至指定 URI
Proxy-Authenticate|	代理服务器对客户端的认证信息
Retry-After	|对再次发起请求的时机要求
Server|	HTTP 服务器的安装信息
Vary	|代理服务器缓存的管理信息
WWW-Authenticate|	服务器对客户端的认证信息


```
HTTP/1.1 200 OK
Age: 529651
Cache-Control: max-age=604800
Connection: keep-alive
Content-Encoding: gzip
Content-Length: 648
Content-Type: text/html; charset=UTF-8
Date: Mon, 02 Nov 2020 17:53:39 GMT
Etag: "3147526947+ident+gzip"
Expires: Mon, 09 Nov 2020 17:53:39 GMT
Keep-Alive: timeout=4
Last-Modified: Thu, 17 Oct 2019 07:18:26 GMT
Proxy-Connection: keep-alive
Server: ECS (sjc/16DF)
Vary: Accept-Encoding
X-Cache: HIT

```

### 实体首部字段

针对请求报文和响应报文的实体部分使用的首部。补充了资源内容更新时间等与实体有关的信息。

|首部字段名|说明|
|--|--
Allow	|资源可支持的 HTTP 方法
Content-Encoding	|实体主体适用的编码方式
Content-Language	|实体主体的自然语言
Content-Length|	实体主体的大小
Content-Location|	替代对应资源的 URI
Content-MD5	|实体主体的报文摘要
Content-Range	|实体主体的位置范围
Content-Type	|实体主体的媒体类型
Expires|实体主体过期的日期时间
Last-Modified	|资源的最后修改日期时间


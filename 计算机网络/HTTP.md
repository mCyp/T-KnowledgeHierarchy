# HTTP

## 一、简介（HTTP 1.1）

### 1. 定义

`HTTP`（HyperText Transfer protocal），超文本传输协议，作用于与应用层。

### 2. 作用

与`TCP/IP`协议簇内的其他协议一样，用于客户端和服务端之前的通信，也就是定义了应用进程间的通信准则。

### 3. 工作方式

- 采用`C/S`（Client/Server）结构，两台计算机使用`HTTP`协议通信的时候，在一条通信路线中，必定一方是客户端，另一方是服务端。
- 请求和响应的交换达成通信。客户端发出请求，服务端回复响应。

### 4. 特点

- 传输效率高
  - 无状态：不会保存状态，保存状态需要借助`Cookie`
- 安全性高
  - 使用`TCP`作为运输层的协议

## 二、报文

### 1. 定义

**用于HTTP协议交互的信息**称为报文。那么，再细一点讲呢？引用《图解HTTP》中的原文：

> 是HTTP中的基本通信单位，由8位组字节流（octet sequence，其中octet为8个比特），通过HTTP通信传输

### 2. 请求报文和响应报文

客户端发出的报文被称为**请求报文**，服务端返回的信息被称为**响应报文**。

### 3. HTTP报文结构

讲解结构之前，我们先看一下正常的报文结构应该是什么样的：

*图片来自《图解HTTP》*

|   名称   |                             图片                             |
| :------: | :----------------------------------------------------------: |
| 请求报文 | ![请求报文](https://teaof-konwleadge-1255982134.cos.ap-shanghai.myqcloud.com/blog/%E7%BD%91%E7%BB%9C/HTTP/%E8%AF%B7%E6%B1%82%E6%8A%A5%E6%96%87.png) |
| 响应报文 | ![响应报文](https://teaof-konwleadge-1255982134.cos.ap-shanghai.myqcloud.com/blog/网络/HTTP/响应报文.png) |

对于小白来说，这张可能还是不懂，这两张总清晰点了吧😏：

*图片来自[《计算机网络：这是一份全面& 详细 HTTP知识讲解》](https://www.jianshu.com/p/a6d086a3997d)*

|     名称     |                             图片                             |
| :----------: | :----------------------------------------------------------: |
| 请求报文结构 | ![请求报文结构](https://teaof-konwleadge-1255982134.cos.ap-shanghai.myqcloud.com/blog/%E7%BD%91%E7%BB%9C/HTTP/%E8%AF%B7%E6%B1%82%E6%8A%A5%E6%96%87%E7%BB%93%E6%9E%84.png) |
| 响应报文结构 | ![响应报文结构](https://teaof-konwleadge-1255982134.cos.ap-shanghai.myqcloud.com/blog/%E7%BD%91%E7%BB%9C/HTTP/%E5%93%8D%E5%BA%94%E6%8A%A5%E6%96%87%E7%BB%93%E6%9E%84.png) |

以下是各个部分：

#### 3.1 请求行 & 响应行

##### HTTP方法（请求行）

`HTTP`方法的作用就是告诉服务器的意图，比如说我要得到你这里的信息，我要上传我这里的信息，我要删除某个信息等，常见的方法有：

- `GET`：获取资源
- `POST`：上传实体主体
- `PUT`：传输文件。鉴于`HTTP/1.1`方法自身不带验证机制，任何人都可以上传文件，因此存在安全性问题
- `HEAD`：获得报文首部
- `DELETE`：与`PUT`相反，按请求的`URI`删除指定的文件
- `OPTIONS`：询问支持的方法
- `TRACE`：追踪踪迹
- `CONNECT`：要求用隧道协议代替代理

常用的有`GET`、`POST`和`HEAD`，如果服务器使用RESTFUL接口，则会用到`GET`、`PUT`、`POST`和`DELETE`。`HTTP方法`的最后，谈一下`POST`和`GET`的区别：

| HTTP方法 |                      传递参数的长度限制                      | 传递的参数类型 | 安全性                          | 使用场景             |
| :------: | :----------------------------------------------------------: | -------------- | ------------------------------- | -------------------- |
|   GET    | `GET`方法的参数加在`URL`的后面，使用“?”代表`URL`的结尾，使用“&”代表请求参数的开始 | ASCII字符      | 不安全，因为参数都暴露在`URL`中 | 参数不多，数据不敏感 |
|   POST   |                            无限制                            | 任何类型       | 安全                            | 量大，或者数据敏感   |

##### URI（request-URI）（请求行）

指明请求访问的资源对象，如果我们输入的请求访问地址为`www.teaOf.com/index.html`，那么我们此时的`URI`就是`/index.html`。

##### 协议版本

`HTTP`的版本号，比如`HTTP 1.1`、`HTTP\2`。

##### 状态码、状态短语（响应行）

|       | 类别                             | 原因短语                   |
| :---- | -------------------------------- | -------------------------- |
| `1XX` | Infromational（信息状态码）      | 接收的请求正在处理         |
| `2XX` | Success（成功状态码）            | 请求正常处理完毕           |
| `3XX` | Redirection（重定向状态码）      | 需要进行附加操作以完成请求 |
| `4XX` | Client Error（客户端错误状态码） | 服务器无法处理请求         |
| `5XX` | Server Error（服务端错误状态码） | 服务端处理请求出错         |

#### 3.2 HTTP头部

HTTP头部存放着HTTP报文的重要信息

##### 通用首部字段

| 名称              | 作用                       |
| ----------------- | -------------------------- |
| Cache-Control     | 控制缓存的行为             |
| Connection        | 逐跳首部，连接管理         |
| Date              | 创建报文的日期时间         |
| Pragma            | 报文指令                   |
| Trailer           | 报文末端的首部一览         |
| Transfer-Encoding | 指定报文主体的传输编码方式 |
| Upgrade           | 升级为其他协议             |
| Via               | 代理服务器的相关信息       |
| Warning           | 错误通知                   |

最常见的应该就是`Cache-Control`了

##### 请求首部字段

| 名称                | 作用                                          |
| ------------------- | --------------------------------------------- |
| Accept              | 用户代理可处理的类型                          |
| Accept-Charset      | 优先的字符集                                  |
| Accept-Encoding     | 优先的编码方式                                |
| Accept-Language     | 优先的语言（自然语言）                        |
| Authorization       | web认证信息                                   |
| Expect              | 期待服务器的特定行为                          |
| From                | 用户的电子邮箱地址                            |
| Host                | 请求资源的所在服务器                          |
| If-Match            | 比较实体标记（ETag）                          |
| If-Modified-Match   | 比较资源的更新时间                            |
| If-None-Match       | 比较实体标记（与If-Match相反）                |
| If-Range            | 资源未更新时发送实体Byte的范围请求            |
| If-Unmodified-Since | 比价资源的更新时间（与If-Modified-Match相反） |
| Max-Forwards        | 最大传输跳数                                  |
| Proxy-Authorization | 代理服务器要求客户端的认证信息                |
| Range               | 实体的字节范围请求                            |
| Refer               | 队请求中URI的原始获取方                       |
| TE                  | 传输编码的优先级                              |
| User-Agent          | HTTP客户端程序的信息                          |

上面的几个好像都是我们常见的

##### 响应首部字段

| 名称                | 作用                         |
| ------------------- | ---------------------------- |
| Accept-Ranges       | 是否接受字节范围请求         |
| Age                 | 推算资源创建的时间           |
| ETag                | 资源的匹配时间               |
| Location            | 令客户端重定向至指定的URI    |
| Proxy-Authenticatie | 代理服务器对客户端的认证信息 |
| Retry-After         | 对再次发起请求的时机要求     |
| Server              | HTTP服务器的安装信息         |
| Vary                | 代理服务器缓存的管理信息     |
| WWW-Authenticate    | 服务器对客户端的认证信息     |

##### 实体首部字段

| 名称             | 作用                   |
| ---------------- | ---------------------- |
| Allow            | 资源可支持的HTTP方法   |
| Content-Encoding | 实体主体适用的编码方式 |
| Content-Language | 实体主体的自然语言     |
| Content-Length   | 实体主题的大小         |
| Content-Location | 替代对应资源的URI      |
| Content-MD5      | 实体主体的报文摘要     |
| Content-Range    | 实体主题的位置范围     |
| Content-Type     | 实体主体的媒体类型     |
| Expires          | 实体主体的过期时间     |
| Last-Modified    | 资源的最后修改日期时间 |

##### 非HTTP首部字段

比较出名的就是`Cookie`和`set-Cookie`。

#### 3.3 主体

主体存放主要的数据内容信息。

**请求体**和**响应体**存放格式（*图片来自于[《计算机网络：这是一份全面& 详细 HTTP知识讲解》](https://www.jianshu.com/p/a6d086a3997d)*）：

![主体](https://teaof-konwleadge-1255982134.cos.ap-shanghai.myqcloud.com/blog/%E7%BD%91%E7%BB%9C/HTTP/HTTP%E4%B8%BB%E4%BD%93.png)

需要注意的是，`GET`方法没有请求体。

## 三、更加安全的HTTPS 

`HTTP`虽然非常优秀，但是它也有一些**安全方面**缺点：

- 通信使用明文，内容可能会被窃听
- 不能验证对方身份，可能遭遇伪装
- 无法证明报文的完整性，报文可能会被篡改

因此，为了解决安全方面的问题，引入了`HTTPS`。

### 1. HTTPS

#### 1.1 HTTPS介绍

由于`HTTP`协议中没有加密机制，但可以通过**SSL**（Secure Secoket Layer，安全套接层）或**TLS**（Transport Layer Security，安全层传输协议，基于SSL）添加了加密及认证机制。**与SSL组合使用的HTTP被称为HTTPS（HTTP Secure，超文本传输安全协议）或HTTP over SSL**。

**HTTPS = HTTP + 加密 + 认证 + 完整性保护**

#### 1.2 HTTP原理

`HTTPS`并非是应用层新的协议，而是`HTTP`通信接口部分用`SSL`和`TLS`协议代替而已。

通常，`HTTP`直接和`TCP`通信，当使用`SSL`的时候，则演变成先和`SSL`通信，接着，再由`SSL`和`HTTP`通信了。

<img src="https://teaof-konwleadge-1255982134.cos.ap-shanghai.myqcloud.com/blog/%E7%BD%91%E7%BB%9C/HTTP/HTTPS%E4%BD%9C%E7%94%A8%E5%8E%9F%E7%90%86.png" alt="HTTP" style="zoom:50%;" />

#### 1.3 HTTPS缺点

**速度慢**

与`HTTP`相比，虽然`HTTPS`的安全性提高了，但是速度降低了，一般情况下，HTTPS会比HTTP慢2-100倍，究其原因，主要是`SSL`通信慢并且大量消耗CPU和内存资源，导致处理速度慢。

**证书价格略贵**

要进行HTTPS通信，证书是不可少的，每年的费用大概是600软妹币，对于个人网站而言并不是特别划算。

#### 1.4 HTTPS的使用

- 服务器使用
- 客户端使用 - 确认访问用户的认证

## 四、HTTP/2

### 1. 背景

随着时代的发展，`HTTP 1.1`俨然已不能够满足一些网站的需求了，主要受限于以下的`HTTP`标准：

- 一条连接上只可以发送一个请求
- 请求只能从客户端开始
- 请求\响应首部未经压缩就发送，首部信息越多发送延迟就越大
- 发送冗长的首部。每次发送相同的首部造成的浪费比较多
- 可以任意选择压缩格式。非强制压缩发送

对此，人们也作出了很多探索，比如`Ajax`、`Comet`、`SPDY`和`WebSocket`。

#### 1.1 Ajax

`Ajax`（Asynchoronus JavaScript and XML，异步JavaScript与XML技术）是一种有效利用`JavaScript`和`DOM`（Doucument Object Model，文档对象模型的操作）的操作，以达到局部Web页面替换加载的异步通信手段。和以前的通信手段相比，由于它只更新一部分页面，响应中传输的数据会减少，这一优点显而易见。

原理（*来自《图解HTTP》*）：

<img src="https://teaof-konwleadge-1255982134.cos.ap-shanghai.myqcloud.com/blog/%E7%BD%91%E7%BB%9C/HTTP/Ajax%E5%8E%9F%E7%90%86.jpeg" alt="Ajax原理" style="zoom:50%;" />

#### 1.2 Comet

一旦服务器内容更新了，`Comet`不会让请求等待，而是直接给客户端返回响应。这是通过延迟应答，模拟实现服务器端向客户端推送（Server Push）的功能。

原理（*来自《图解HTTP》*）：

<img src="https://teaof-konwleadge-1255982134.cos.ap-shanghai.myqcloud.com/blog/%E7%BD%91%E7%BB%9C/HTTP/Comet%E5%8E%9F%E7%90%86.jpeg" alt="Comet原理" style="zoom:50%;" />

#### 1.3 SPDY

**SPDY**（读作"SPeeDy"），是Google开发的基于TCP会话层协议。`Ajax`和`Comet`的出现，并不能解决根本性问题，所以出现了在协议层级解决HTTP瓶颈的**SPDY协议**。

`SPDY`没有完全改写HTTP协议，而是在`TCP/IP`的应用层与传输层之间通过新加会话层的形式运作。同时，考虑到安全性的问题，`SPDY`规定通信中使用`SSL`。`SPDY`以会化层的形式加入，控制数据的流动，但还是采用`HTTP`通信建立连接。

`SPDY`的特点：

- 多路复用流：单一的`TCP`连接，可以无限制的处理多个`HTTP`请求
- 赋予请求优先级：`SPDY`不仅可以无限制地并发处理请求，还可以为每个请求分配优先级
- 强制压缩HTTP首部
- 推送功能：支持服务器主动向客户端推送数据的功能
- 服务器提示功能：服务器可以主动提示客户端需要的资源

#### 1.4 WebSocket

**WebSocket**，即`Web浏览器`和`Web服务器`之间全双工通信标准。

一旦`Web`服务器与客户端建立起WebSocket协议的通信连接，之后所有的通信都依靠这个专有协议进行。通信过程中可相互发送`Json`、`XML`、`HTML`或图片任意格式的数据。

`WebSocket`的特点：

- 推送功能
- 减少通信量：只要建立起`WebSocket`，就希望一直保持连接。

原理（*来自《图解HTTP》*）：

<img src="https://teaof-konwleadge-1255982134.cos.ap-shanghai.myqcloud.com/blog/%E7%BD%91%E7%BB%9C/HTTP/WebSocket%E5%8E%9F%E7%90%86.jpeg" alt="WebSocket原理" style="zoom:50%;" />



### 2. 定义

维基百科：

> **HTTP/2**（超文本传输协议第二版，最初命名为HTTP 2.0），简称为h2（TLS/1.2或以上的加密连接）或h2c（非加密连接），是HTTP协议的第二个版本，适用于万维网。HTTP/2是HTTP协议自1999年HTTP 1.1发布后的首个更新，主要基于SPDY协议。该项协议于2015年2月17日正式被批准。

### 3. 特点

由于`HTTP/2`基于`SPDY`，所以它的很多特点来自于`SPDY`：

- 对`HTTP`头字段进行数据压缩（HPACK算法）
- `HTTP/2`服务端推送
- 请求管线化
- 修复1.0版本未修复的队头阻塞问题
- 对数据传输采用多路复用，让多个请求合并在一个TCP链接内

需要注意的是，`SPDY`的安全协议基于`SSL`，而`HTTP/2`则基于`TLS`。

## 五、知识点

### 1. WEB服务器相关

- HTTP/1.1规范允许一台HTTP服务器搭建多个Web站点
- 通信数据转发程序：代理、网关和隧道
  - 代理 - 客户端和服务端的信息转发工作
    - 缓存代理
    - 透明代理
  - 网关 - 工作机制同代理，不过可以使用非HTTP协议服务
  - 隧道 - 确保客户端和服务器之间的通信安全




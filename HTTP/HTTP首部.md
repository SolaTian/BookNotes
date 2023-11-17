# 6、`HTTP`首部

`HTTP`首部分类：

- 通用首部字段：请求和响应报文都会使用的字段
- 请求首部字段：请求报文时使用的首部
- 响应首部字段：响应报文时使用的首部
- 实体首部字段：针对请求和响应报文的实体部分使用的首部。

`HTTP`首部还可以分成：
- 端到端首部：此类首部会转发请求/响应对应的最终接收目标，且必须保存在缓存生成的响应中，必须被转发；
- 逐跳首部：只对单次转发有效，会因通过缓存或者代理而不再转发，除了以下字段，其他均为端到端首部，包括
  - `Connection`
  - `Keep-alive`
  - `Proxy-Authenticate`
  - `Proxy-Authorization`
  - `Trailer`
  - `TE`
  - `Transfer-Encoding`
  - `Upgrad`

非`HTTP`首部字段：

除了`HTTP`首部字段，还有`Cookie`、`Set-Cookie`和`Content-Disposition`等其他字段的使用频率也很高。



## 6.1、通用首部字段

**<font size =4 color = red>`Cache-Control`</font>**

> `Cache-Control`：操作缓存的控制机制。多个指令之间通过`,`分割。分为缓存请求指令和缓存响应指令。

|表示是否能缓存的指令|说明|
|-|-|
|`Cache-Control:public`|表明其他用户可以使用缓存|
|`Cache-Control:private`|响应只以特定的用户作为对象，与`public`相反，只会对该特定用户提供资源缓存的服务，其他用户发过来的请求，代理服务器不会返回缓存|
|`Cache-Control:no-cache`|防止从缓存中返回过期的资源；<li>若为客户端请求，则表示客户端不会接收缓存过的响应，中间的缓存代理会将客户端请求转发给源服务器。<li>若为服务端响应，则表示缓存服务器不可以对资源进行缓存。<li>服务端响应的字段中若加上参数，`no-cache=Location`（只能在响应指令中指定），则表示客户端在接收到响应报文的时候也不可以进行缓存。|


|控制可执行缓存的对象的指令|说明|
|-|-|
|`Cache-Control:no-store`|暗示请求或者响应中包含机密信息，规定不允许进行缓存|


|指定缓存期限和认证的指令|说明|
|-|-|
|`Cache-Control:s-maxage=604800`|单位为秒，与`max-age`指令相同。但仅适用于供多位用户使用的公共缓存服务器，忽视`Expires`和`max-age`指令的处理|
|`Cache-Control:max-age=604800`|单位为秒，<li>客户端角度，如果缓存的时间数值比指定时间小，则客户端接收缓存资源，当指定`max-age=0`，那么缓存服务器通常需要将请求转发给源服务器。<li>服务端角度，缓存服务器不再对资源的有效性进行确认，而表示缓存资源保存的最长时间。忽视`Expires`字段。|
|`Cache-Control:min-fresh=60`|单位为秒，要求缓存服务器返回至少还没过指定时间的缓存资源，意思为在60s之内，如果有超过有效期限的资源都无法作为响应返回|

**<font size =4 color = red>`Connection`</font>**

> `Connection`：控制不再转发给代理的首部字段，管理持久连接。

控制代理不再转发的首部字段：如`Connection:Upgrad`表示让代理服务器将`Upgrad`字段删除后再转发。

`HTTP/1.1`默认都是持久连接，`Connection:Keep-alive`，当服务器想要断开连接时，则指定`Connection:close`

**<font size =4 color = red>`Date`</font>**

> `Date`：创建`HTTP`报文的日期和时间，如`Date: Tue, 03 Jul 2012 04:40:59 GMT`。


**<font size =4 color = red>`Pragma`</font>**

> `Pragma`：只用在客户端发送的请求中，要求所有的中间服务器不返回缓存的资源。用于`HTTP/1.1`之前的版本，通常和`Cache-Control: no-cache`一起使用，兼容`HTTP/1.0`和`HTTP/1.1`。

**<font size =4 color = red>`Trailer`</font>**

> `Trailer`：事先说明在报文主体后记录了哪些首部字段，可用于`HTTP/1.1`分块传输编码。

`Trailer: Expires`表示后面会出现`Expires`字段。

**<font size =4 color = red>`Transfer-Encoding`</font>**

> `Transfer-Encoding`：规定传输报文主体时使用的编码方式，仅对分块传输编码有效。

**<font size =4 color = red>`Upgrade`</font>**

> `Upgrade`：用于检测`HTTP`协议以及其他协议是否可以使用更高的版本进行通信。

`Upgrade`首部字段产生作用的`Upgrade`对象仅限于客户端和邻接服务器之间，因此使用首部字段`Upgrade`时，需要额外指定`Connection: Upgrade`。

对于首部含有`Upgrad`字段的请求，服务器可以用`101 Switching Protocols`返回。


**<font size =4 color = red>`Via`</font>**

> Via：追踪客户端和服务器之间的请求和响应报文的传输路径。报文经过代理或者网关时，会首先在`Via`字段中附上该服务器的信息，然后再进行转发，经常会和`TRACE`方法一起使用。


**<font size =4 color = red>`Warning`</font>**

> `Warning`：告知用户一些与缓存相关的警告。格式为`Warning: [警告码][警告主机:端口]"[警告内容]([日期时间])"`

## 6.2、请求首部字段

**<font size =4 color = red>`Accept`</font>**

>`Accept`：通知服务器，用户代理能够处理的媒体类型以及媒体类型的相对优先级。

例如：
- 文本文件：`text/html`，`text/plain`：
- 图片文件：`image/jpeg`，`image/gif`，`image/png`；
- 视频文件：`video/mpeg`，`video/quicktime`；
- 应用程序可以使用的二进制文件：`application/octet-stream`，`application/zip`等。

可以使用`q=`来表示权重，使用分号`;`隔开，权重q的范围为0-1，不指定时默认为1，当服务器提供多种内容时，会首先返回权重值最高的媒体类型。

**<font size =4 color = red>`Accept-Charset`</font>**

>`Accept-Charset`：用来通知服务器用户代理支持的字符集以及字符集的相对优先顺序，可以一次性指定多种字符集，也可以使用`q=`来表示相对优先级。

**<font size =4 color = red>`Accept-Encoding`</font>**

>`Accept-Encoding`:用来通知服务器用户代理支持的内容编码以及内容编码的优先级顺序，可一次性指定多种内容编码。

例如：
- `gzip`：由文件压缩程序`gzip`生成的编码格式
- `compress`：由`UNIX`文件压缩程序`compress`生成的编码格式
- `deflate`：组合使用`zlib`格式以及由`deflate`算法生成的编码格式
- `identity`:不执行压缩或不会变化的默认格式。

也可以使用`q`来表示相对优先级。也可以使用`*`来表示指定任意的编码格式。

**<font size =4 color = red>`Accept-Language`</font>**

>`Accept-Language`：用来通知服务器用户代理能够处理的自然语言集以及自然语言集的相对优先级。可以一次指定多种自然语言集。使用`q=`来表示相对优先级。

**<font size =4 color = red>`Authorization`</font>**

>`Authorization`：用来告知服务器，用户代理的认证信息。

通常，想要通过服务器认证的用户代理会在接收到返回的401状态码响应之后，把首部字段`Authorization`加入请求中。

**<font size =4 color = red>`Expect`</font>**

>`Expect`：用来告知服务器，期望出现的某种特定行为。

因服务器无法理解客户端的期望做出回应而发生错误时，会返回状态码`417 Expectation Failed`。`HTTP/1.1`规范只定义了`100-continue`，即`Expect: 100-continue`


**<font size =4 color = red>`From`</font>**

>`From`：用来告知服务器使用用户代理的用户的电子邮件地址。


**<font size =4 color = red>`Host`</font>**

>`Host`：告知服务器，请求的资源所处的互联网主机名和端口号。是`HTTP/1.1`中唯一一个必须包括在请求内的字段。若服务器未设置主机名，直接发送一个空值

**<font size =4 color = red>`If-xxx`</font>**

>`If-xxx``：表示条件请求，服务器在接收到附带条件的请求后，只有判断指定条件为真时，才会执行请求。

包括以下：
- `If-Match`：告知服务器匹配资源所用的`ETag`，仅当两者一致时，才会执行请求，否则返回状态码`412 Precdondition Failed`
- `If-Modified-Since`：指定日期，告知服务器若该字段值早于资源的更新时间，则希望处理该请求，而在指定的`If-Modified-Since`字段时间之后，如果请求的资源都没有更新过，则返回`304 Not Modified`
- `If-None-Match`：与`If-Match`作用相反。
- `If-Range`：告知服务器，如果指定的`If-Range`字段和请求的资源的`ETag`或者时间一致，则作为范围请求，否则返回全部资源。
- `If-Unmodified-Since`：与`If-Modified-Since`作用相反。


**<font size =4 color = red>`Proxy-Authorization`</font>**

接收到从代理服务器发来的认证问询时，客户端会发送该首部字段`Proxy-Authorization`

**<font size =4 color = red>`Range`</font>**

> `Range`：对于只获取部分资源的范围请求，包含首部字段`Range`即可告知服务器资源的指定范围，如`Range: bytes=5001-10000`表示请求获取第5001字节至10000字节。

当可以处理时，返回`206 Partital Content`，不能处理该请求时，会返回`200 OK`

**<font size =4 color = red>`Referer`</font>**

> `Referer`：告知服务器请求的原始资源的`URI`。只要查看`Referer`就可以知道请求的`URI`是从哪个`Web`页面发起的。

客户端一般都会发送该字段给服务器，但是原始的`URI`中可能含有`ID`和密码等保密信息，可能会导致信息泄露。

**<font size =4 color = red>`User-Agent`</font>**

>`User-Agent`:会将创建请求的浏览器和用户代理名称等信息传达给服务器。

由网络爬虫发起请求时，有可能会在字段内添加爬虫作者的电子邮件地址。如果经过代理，也可能将代理服务器的名称放在其中。

## 6.3、响应首部字段

**<font size =4 color = red>`Accept-Ranges`</font>**

>`Accept-Ranges`：首部字段用来告知客户端服务器能否处理范围请求，以指定获取服务器某个部分的资源。可处理范围请求时指定其为`bytes`，反之则指定其为`none`。


**<font size =4 color = red>`Age`</font>**

>`Age`:告知客户端，源服务器在多久前创建了响应，单位为秒。

如果创建响应的是缓存服务器，`Age`指缓存之后的响应再次发起认证到认证完成的时间值，代理创建响应时必须加上首部字段`Age`。

**<font size =4 color = red>`ETag`</font>**

> `ETag`：告知客户端实体标识，是一种将资源以字符串形式做唯一性标识的方式。服务器会为每份资源分配对应的`ETag`，当资源发生更新时，`ETag`值也需要更新，虽然资源的`URI`没有发生变化。

比如浏览器的中英文界面，`URI`不变，但是对应的资源是不同的，`ETag`也是不同的。

`ETag`分为：
- 强`ETag`：不管实体发生什么样的细微变化都会改变其值
- 弱`ETag`：只用于提示资源是否相同，只有资源发生了根本变化，产生差异才会改变`ETag`值。

**<font size =4 color = red>`Location`</font>**

>`Location`:将响应接收方引导至某个与请求`URI`位置不同的资源。

该字段基本会配合`3XX: Redirection`，提供重定向的`URI`。


**<font size =4 color = red>`Proxy-Authenticate`</font>**

> `Proxy-Authenticate`：会把由代理服务器所要求的认证信息发送给客户端。

**<font size =4 color = red>`Retry-After`</font>**

>`Retry-After`:告知客户端多久之后再发送请求。

主机要配合`503 Service Unavailable`响应

**<font size =4 color = red>`Server`</font>**

>`Server`:告知客户端当前服务器上安装的`HTTP`服务器应用程序的信息，包括服务器上的软件应用名称，还有可能包括版本号和安装时启动的可选项。


**<font size =4 color = red>`Vary`</font>**
>`Vary`:对缓存进行控制，源服务器会向代理服务器传达关于本地缓存使用方法的命令。代理服务器仅对请求中含有相同`Vary`指定首部字段的请求返回缓存。否则，必须要从源服务器中获取资源。

**<font size =4 color = red>`WWW-Authenticate`</font>**
>`WWW-Authenticate`:用于`HTTP`访问认证，告知客户端适用于访问请求`URI`所指定资源的认证方案(`Basic`或`Digest`)

一般配合状态码`401 Unauthorized`

## 6.4、实体首部字段

实体首部字段是包含在请求报文和响应报文中的实体部分所使用的首部。

**<font size =4 color = red>`Allow`</font>**
>`Allow`:用于通知客户端能够支持请求指定资源的所有的`HTTP`方法。

当服务器接收到不支持的`HTTP`方法时，会以状态码`405 Method Not Allowed`作为响应返回。

**<font size =4 color = red>`Content-Encoding`</font>**
>`Content-Encoding`:告知客户端，服务器对实体的主体部分选用的内容编码方式。

主要采用`gzip`，`compress`，`deflate`，`identity`

**<font size =4 color = red>`Content-Language`</font>**
>`Content-Language`:告知客户端，实体主体使用的自然语言


**<font size =4 color = red>`Content-Length`</font>**
>`Content-Length`:表示实体主体部分的大小，单位为字节。当对实体的主体进行编码传输时，不能再使用`Content-Length`首部字段。

**<font size =4 color = red>`Content-Location`</font>**
>`Content-Location`:给出与报文主体部分相对应的`URI`。

和`Location`不同，`Content-Location`表示的是报文主体返回资源对应的`URI`。

**<font size =4 color = red>`Content-MD5`</font>**
>`Content-MD5`：一串由`MD5`算法生成的值，其目的在于检查报文主体在传输过程中是否保持完整，以及确认传输到达。

作为接收端的客户端会对报文主体再执行一次`MD5`算法，计算出的值与字段值进行比较，获得报文的准确性。

**<font size =4 color = red>`Content-Range`</font>**
>`Content-Range`：针对范围请求，返回响应时使用的首部字段`Content-Range`。

字段值以字节为单位，表示当前发送部分以及整个整体的大小，如`Content-Range: bytes 5001-10000/10000`。


**<font size =4 color = red>`Content-Type`</font>**
>`Content-Type`:说明实体主体内对象的媒体类型，和首部字段`Accept`一样，字段用`type/subtype`形式进行赋值。

**<font size =4 color = red>`Expires`</font>**
>`Expires`:将资源失效的日期告诉客户端.

**<font size =4 color = red>`Last-Modified`</font>**
>`Last-Modified`:资源的最终的修改时间。

## 6.5、为`Cookie`服务的首部字段

`Cookie`的工作机制是用户识别及状态管理，`Web`网站为了管理用户的状态会通过`Web`浏览器，把一些数据临时写入用户的计算机中，接着当用户访问该`Web`网站时，可通过通信方式取回之前存放的`Cookie`。

**<font size =4 color = red>`Set-Cookie`</font>**
>`Set-Cookie`:服务器用来管理客户端的状态的字段。

`Set-Cookie`字段的属性：

- `expire`属性：指定浏览器可以发送`Cookie`的有效期。省略时表示有效期仅限于维持浏览器会话的时间段内。通常限于浏览器被关闭之前。
- `path`属性：用于限制指定`Cookie`的发送范围的文件目录。哪些文件是`Cookie`的适用对象。若不指定则默认为文档所在的文件目录。
- `domain`属性：指定作为`Cookie`适用对象的域名。若不指定则默认为创建`Cookie`服务器的域名。`domain`指定的域名可以做到与结尾匹配一致，如指定`example.com`，则除了`example.com`之外，其他结尾为`example.com`的都可以发送`Cookie`
- `secure`属性：限制`Web`页面仅在`HTTPS`安全连接时才可以发送`Cookie`。当省略`secure`时，不论是`HTTP`还是`HTTPS`，都会对`Cookie`进行回收。
- `HttpOnly`属性：使得`JavaScript`脚本无法获得`Cookie`，目的为了防止跨站脚本攻击对`Cookie`信息的窃取。

**<font size =4 color = red>`Cookie`</font>**
>`Cookie`：告知服务器，当客户端想要获得`HTTP`状态管理支持时，就会在请求中包含从服务器中接收的`Cookie`。

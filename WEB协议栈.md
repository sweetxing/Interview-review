

## HTTP报文

### 1. 基于 ABNF 描述的 HTTP 协议格式

**HTTP-message = start-line *( header-field CRLF ) CRLF [ message-body ]**

- start-line = request-line / status-line
  - request-line=methodSPrequest-targetSPHTTP-versionCRLF
  - status-line=HTTP-versionSPstatus-codeSPreason-phraseCRLF

- header-field = field-name ":" OWS field-value OWS
  - OWS=*(SP/HTAB)
  - field-name = token
  - field-value = *( field-content / obs-fold )
- message-body = *OCTET

### 2. HTTP请求行（start-line）

- **request-line = method SP request-target SP HTTP-version CRLF**
  - method 方法:指明操作目的，动词
  - request-target = origin-form / absolute-form / authority-form / asterisk-form
    - origin-form = absolute-path [ "?" query ]
      -  向 origin server 发起的请求，path 为空时必须传递 /
    - absolute-form = absolute-URI
      - 仅用于向正向代理 proxy 发起请求时，详见正向代理与隧道
    - authority-form = authority
      - 仅用于 CONNECT 方法，例如 CONNECT www.example.com:80 HTTP/1.1
    - asterisk-form = "*“
      - 仅用于 OPTIONS 方法
  - HTTP-version
    - HTTP/0.9:只支持 GET 方法，过时
    - HTTP/ 1.0:RFC1945，1996， 常见使用于代理服务器(例如 Nginx 默认配置)
      - 使用非持久方法，一个tcp连接只能传递一个web对象
    - HTTP/ 1.1:RFC2616，1999
      - 默认使用长连接，一个tcp连接可以传递多个web对象
    - HTTP/ 2.0:2015.5 正式发布
- **method**
  - 常见方法
    - GET:主要的获取信息方法，大量的性能优化都针对该方法，幂等方法
    - HEAD:类似 GET 方法，但服务器不发送 BODY，用以获取 HEAD 元数据，幂等方法
    - POST:常用于提交 HTML FORM 表单、新增资源等
    - PUT:更新资源，带条件时是幂等方法
    - DELETE:删除资源，幂等方法
    - CONNECT:建立 tunnel 隧道
    - OPTIONS:显示服务器对访问资源支持的方法，幂等方法
    - TRACE:回显服务器收到的请求，用于定位问题。有安全风险
  - 用于文档管理的 WEBDAV 方法
    - PROPFIND:从 Web 资源中检索以 XML 格式存储的属性。它也被重载，以允 许一个检索远程系统的集合结构(也叫目录层次结构)
    - PROPPATCH:在单个原子性动作中更改和删除资源的多个属性
    - MKCOL:创建集合或者目录
    - COPY:将资源从一个 URI 复制到另一个 URI
    - MOVE:将资源从一个 URI 移动到另一个 URI
    - LOCK:锁定一个资源。WebDAV 支持共享锁和互斥锁。
    - UNLOCK:解除资源的锁定

### 3. HTTP 响应行

- **status-line = HTTP-version SP status-code SP reason-phrase CRLF**
  - status-code = 3DIGIT
  - reason-phrase = *( HTAB / SP / VCHAR / obs-text )

- **响应码分类**

  - 1xx:请求已接收到，需要进一步处理才能完成，HTTP1.0 不支持

    - 100 Continue:上传大文件前使用
      - 由客户端发起请求中携带 Expect: 100-continue 头部触发
    - 101 Switch Protocols:协议升级使用
      - 由客户端发起请求中携带 Upgrade: 头部触发，如升级 websocket 或者 http/2.0
    - 102 Processing:WebDAV 请求可能包含许多涉及文件操作的子请求，需要很长时间 才能完成请求。该代码表示服务器已经收到并正在处理请求，但无响应可用。这样可 以防止客户端超时，并假设请求丢失

  - 2xx:成功处理请求

    -  200 OK: 成功返回响应。

    - 201 Created: 有新资源在服务器端被成功创建。

    - 202 Accepted: 服务器接收并开始处理请求，但请求未处理完成。这样一个模 糊的概念是有意如此设计，可以覆盖更多的场景。例如异步、需要长时间处理 的任务。

    - 203 Non-Authoritative Information:当代理服务器修改了 origin server 的 原始响应包体时(例如更换了HTML中的元素值)，代理服务器可以通过修改 200为203的方式告知客户端这一事实，方便客户端为这一行为作出相应的处理。 203响应可以被缓存。

    - 204 No Content:成功执行了请求且不携带响应包体，并暗示客户端无需 更新当前的页面视图。

    - 205 Reset Content:成功执行了请求且不携带响应包体，同时指明客户端 需要更新当前页面视图。

    - 206 Partial Content:使用 range 协议时返回部分响应内容时的响应码

    - 207 Multi-Status:RFC4918 ，在 WEBDAV 协议中以 XML 返回多个资源

      的状态。

    - 208 Already Reported:RFC5842 ，为避免相同集合下资源在207响应码 下重复上报，使用 208 可以使用父集合的响应码。

  - 3xx:重定向使用 Location 指向的资源或者缓存中的资源。在 RFC2068 中规定客户端重定向次数不应超过 5 次，以防止死循环。

    - 300 Multiple Choices:资源有多种表述，通过 300 返回给客户端后由其 自行选择访问哪一种表述。由于缺乏明确的细节，300 很少使用。
    -  301 Moved Permanently:资源永久性的重定向到另一个 URI 中。
    - 302 Found:资源临时的重定向到另一个 URI 中。
    - 303 See Other:重定向到其他资源，常用于 POST/PUT 等方法的响应中。
    - 304 Not Modified:当客户端拥有可能过期的缓存时，会携带缓存的标识 etag、时间等信息询问服务器缓存是否仍可复用，而304是告诉客户端可以 复用缓存。
    - 307 Temporary Redirect:类似302，但明确重定向后请求方法必须与原 请求方法相同，不得改变。
    - 308 Permanent Redirect:类似301，但明确重定向后请求方法必须与原请 求方法相同，不得改变。

  - 4xx:客户端出现错误

    - 400 Bad Request:服务器认为客户端出现了错误，但不能明确判断为以下哪种错误

      时使用此错误码。例如HTTP请求格式错误。

    - 401 Unauthorized:用户认证信息缺失或者不正确，导致服务器无法处理请求。

    - 407 Proxy Authentication Required:对需要经由代理的请求，认证信息未通过代理 服务器的验证

    - 403 Forbidden:服务器理解请求的含义，但没有权限执行此请求

    - 404 Not Found:服务器没有找到对应的资源

    - 410 Gone:服务器没有找到对应的资源，且明确的知道该位置永久性找不到该资源

    - 405 Method Not Allowed:服务器不支持请求行中的 method 方法

    - 406 Not Acceptable:对客户端指定的资源表述不存在(例如对语言或者编码有要

      求)，服务器返回表述列表供客户端选择。

    - 408 Request Timeout:服务器接收请求超时

    - 409 Conflict:资源冲突，例如上传文件时目标位置已经存在版本更新的资源

    - 411 Length Required:如果请求含有包体且未携带 Content-Length 头部，且不属于 chunk类请求时，返回 411

    - 412 Precondition Failed:复用缓存时传递的 If-Unmodified-Since 或 If- None-Match 头部不被满足

    - 413 Payload Too Large/Request Entity Too Large:请求的包体超出服 务器能处理的最大长度

    - 414 URI Too Long:请求的 URI 超出服务器能接受的最大长度

    - 415 Unsupported Media Type:上传的文件类型不被服务器支持

    - 416 Range Not Satisfiable:无法提供 Range 请求中指定的那段包体

    - 417 Expectation Failed:对于 Expect 请求头部期待的情况无法满足时的 响应码

    - 421 Misdirected Request:服务器认为这个请求不该发给它，因为它没有能力 处理。

    - 426 Upgrade Required:服务器拒绝基于当前 HTTP 协议提供服务，通过 Upgrade 头部告知客户端必须升级协议才能继续处理。

    - 428 Precondition Required:用户请求中缺失了条件类头部，例如 If-Match

    - 429 Too Many Requests:客户端发送请求的速率过快

    - 431 Request Header Fields Too Large:请求的 HEADER 头部大小超过限制

    - 451 Unavailable For Legal Reasons:RFC7725 ，由于法律原因资源不可访问

  - 5xx:服务器端出现错误

    - 500 Internal Server Error:服务器内部错误，且不属于以下错误类型 • 501 Not Implemented:服务器不支持实现请求所需要的功能
    - 502 Bad Gateway:代理服务器无法获取到合法响应
    - 503 Service Unavailable:服务器资源尚未准备好处理当前请求
    - 504 Gateway Timeout:代理服务器无法及时的从上游获得响应
    - 505 HTTP Version Not Supported:请求使用的 HTTP 协议版本不支持
    - 507 Insufficient Storage:服务器没有足够的空间处理请求
    - 508 Loop Detected:访问资源时检测到循环
    - 511 Network Authentication Required:代理服务器发现客户端需要进 行身份验证才能获得网络访问权限

**补充知识点：长连接与短链接**

- Connection 头部
  -  Keep-Alive:长连接
    - 客户端请求长连接  Connection: Keep-Alive
    - 服务器表示支持长连接  Connection: Keep-Alive
    - 客户端复用连接
    - HTTP/1.1 默认支持长连接
      - Connection: Keep-Alive 无意义
  - Close:短连接对代理服务器的要求
  - 对代理服务器的要求
    - 不转发 Connection 列出头部，该 头部仅与当前连接相关
- Connection仅对当前连接有效
- 代理服务器对长连接的支持
  - 问题:各方间错误使用了长连接
    - 客户端发起长连接
    - 代理服务器陈旧，不能正确的处理请求的 Connection 头部，将客户端请求中的 Connection: Keep-Alive 原样转发给上游服务器
    - 上游服务器正确的处理了 Connection 头部，在发送响应后没有关闭连接，而试图保持、复用与不认长连接的代理服务器的连接
    - 代理服务器收到响应中 Connection: Keep-Alive 后不认，转发给 客户端，同时等待服务器关闭短连接
    - 客户端收到了 Connection: Keep-Alive，认为可以复用长连接，继 续在该连接上发起请求
    - 代理服务器出错，因为短连接上不能发起两次请求
  - Proxy-Connection
    - 陈旧的代理服务器不识别该头部:退化为短连接
    - 新版本的代理服务器理解该头部 
      -  与客户端建立长连接
      - 与服务器使用 Connection 替代 Proxy-Connect 头部

### 4. HTTP头部

**HOST**

Host 头部  Host = uri-host [ ":" port ]

- HTTP/1.1 规范要求，不传递 Host 头部则返回 400 错误响应码
- 为防止陈旧的代理服务器，发向正向代理的请求 request-target 必须以 absolute-form 形式出现
  - request-line = method SP request-target SP HTTP-version CRLF
  - absolute-form = absolute-URI
    - absolute-URI = scheme ":" hier-part [ "?" query ]

**消息转发相关的头部**

- Max-Forwards 头部
  - 限制 Proxy 代理服务器的最大转发次数，仅对 TRACE/OPTIONS 方法有效 
  - Max-Forwards = 1*DIGIT
- Via 头部
  - 指明经过的代理服务器名称及版本
  - Via = 1#( received-protocol RWS received-by [ RWS comment ] )
    - received-protocol = [ protocol-name "/" ] protocol-version
    - received-by = ( uri-host [ ":" port ] ) / pseudonym
    - pseudonym = token
- Cache-Control:no-transform
  - 禁止代理服务器修改响应包体
- X-Forwarded-For
  - 传递IP，最开始的是离服务端最远的设备 IP，然后是每一级代理设备的 IP。IP数组
- X-Real-IP
  - 传递用户IP

**请求上下文相关的头部**

User-Agent (指明客户端的类型信息，服务器可以据此对资源的表述做抉择)

- User-Agent = product *( RWS ( product / comment ) )*
  - *product = token ["/" product-version]*
  - *RWS=1*(SP/HTAB)

Refer(浏览器对来自某一页面的请求自动添加的头部)

- Referer = absolute-URI / partial-URI
  - Referer: https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/User-Agent
- Referer 不会被添加的场景
  - 来源页面采用的协议为表示本地文件的 "file" 或者 "data" URI
  - 当前请求页面采用的是 http 协议，而来源页面采用的是 https 协议
- 服务器端常用于统计分析、缓存优化、防盗链等功能

From:告知服务器使用用户代理的用户的电子邮件地址

- From: mailbox

通常其使用目的就是为了显示搜索引擎等用户代理的负责人的电子邮件联系方式。使用代理时，应尽可能包含From首部字段（但可能会应代理不同，将电子邮件地址记录在User-Agent首部字段内）。

**响应上下文相关的头部**

Server:指明服务器上所用软件的信息，用于帮助客户端定位问题或者统计数据

- Server = product *( RWS ( product / comment ) )
  - product = token ["/" product-version]

Allow:告诉客户端，服务器上该 URI 对应的资源允许哪些方法的执行

- Allow = #method

Accept-Ranges:告诉客户端服务器上该资源是否允许 range 请求

- Accept-Ranges = acceptable-ranges
  - Accept-Ranges: bytes 
  - Accept-Ranges: none

**内容协商相关的头部**

1. 定义

每个 URI 指向的资源可以是任何事物，可以有多种不同的表述，例如一份 文档可以有不同语言的翻译、不同的媒体格式、可以针对不同的浏览器提 供不同的压缩编码等。

2. 两种方式

- Proactive 主动式内容协商:
  - 指由客户端先在请求头部中提出需要的表述形式，而服务器根据这些请求头部提供特定的 representation 表述
- Reactive 响应式内容协商:
  - 指服务器返回 300 Multiple Choices 或者 406 Not Acceptable，由客户端 选择一种表述 URI 使用

3. 协商要素

- 质量因子q: 内容的质量、可接受类型的优先级
- 媒体资源的 MIME 类型及质量因子
  - Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  - Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp, image/apng,*/*;q=0.8,application/sign
- 字符编码:由于 UTF-8 格式广为使用， Accept-Charset 已被废弃
  - Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
-  内容编码:主要指压缩算法
  - Accept-Encoding: gzip, deflate, br
- 表述语言
  - Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
  - Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
- internationalization(i18n，i 和 n 间有 18 个字符)
  - 指设计软件时，在不同的国家、地区可以不做逻辑实现层面的修改便能够以不同的语言显示
-  localization(l10n，l 和 n 间有 10 个字符)
  - 指内容协商时，根据请求中的语言及区域信息，选择特定的语言作为资源表述

**资源表述的元数据头部**

- 媒体类型、编码
  - content-type: text/html; charset=utf-8
- 内容编码
  - content-encoding: gzip
- 语言
  - Content-Language: de-DE, en-CA

### 5. HTTP报文主体

**HTTP包体**

1. 功能：承载的消息内容

- 请求或者响应都可以携带包体
  - HTTP-message = start-line *( header-field CRLF ) CRLF [ message-body ]
    - message-body = *OCTET:二进制字节流
- 以下消息不能含有包体
  - HEAD 方法请求对应的响应
  - 1xx、204、304 对应的响应
  - CONNECT 方法对应的 2xx 响应

2. 两种传输 HTTP 包体的方式

- 发送 HTTP 消息时已能够确定包体的全部长度
  - 使用 Content-Length 头部明确指明包体长度
    - Content-Length = 1*DIGIT
    - 用 10 进制(不是 16 进制)表示包体中的字节个数，且必须与实际传输的包体长度一致
  - 优点：接收端处理更简单
- 发送 HTTP 消息时不能确定包体的全部长度
  - 使用 Transfer-Encoding 头部指明使用 Chunk 传输方式
    - 含 Transfer-Encoding 头部后 Content-Length 头部应被忽略
  - 优点
    - 基于长连接持续推送动态内容
    - 压缩体积较大的包体时，不必完全压缩完(计算出头部)再发送，可以边发送边压缩
    - 传递必须在包体传输完才能计算出的 Trailer 头部

3. 不定长包体的 chunk 传输方式

- Transfer-Encoding头部

  - transfer-coding = "chunked" / "compress" / "deflate" / "gzip" / transfer-extension

  - Chunked transfer encoding 分块传输编码: Transfer-Encoding:chunked

    - chunked-body = *chunk

      ​							last-chunk

      ​							trailer-part

      ​							CRLF

    - chunk = chunk-size [ chunk-ext ] CRLF chunk-data CRLF

    - chunk-size = 1*HEXDIG:注意这里是 16 进制而不是10进制*

    - *chunk-data = 1*OCTET

    - last-chunk = 1*("0") [ chunk-ext ] CRLF

    - trailer-part = *( header-field CRLF )

4. Trailer 头部的传输

- TE 头部:**客户端**在请求在声明是否接收 Trailer 头部
  - TE: trailers
- Trailer 头部:**服务器**告知接下来 chunk 包体后会传输哪些 Trailer 头部
  - Trailer: Date
- 以下头部不允许出现在 Trailer 的值中:
  - 用于信息分帧的首部 (例如 Transfer-Encoding 和 Content-Length)
  - 用于路由用途的首部 (例如 Host)
  - 请求修饰首部 (例如控制类和条件类的，如 Cache-Control，Max-For wards，或者 TE)
  - 身份验证首部 (例如 Authorization 或者 Set-Cookie)
  - Content-Encoding, Content-Type, Content-Range，以及 Trailer 自身

5. MIME( Multipurpose Internet Mail Extensions )

HTTP 会为每一个通过 web 传输的对象添加上 MIME 类型的数据格式标签。

- content := "Content-Type" ":" type "/" subtype *(";" parameter)
  - type := discrete-type / composite-type
    - discrete-type := "text" / "image" / "audio" / "video" / "application" / extension-token
    - composite-type := "message" / "multipart" / extension-token
    - extension-token := ietf-token / x-token
  - subtype := extension-token / iana-token
  - parameter := attribute "=" value
- 大小写不敏感，但通常是小写
- 例如: Content-type: text/plain; charset="us-ascii“

6. Content-Disposition 头部

- disposition-type = "inline" | "attachment" | disp-ext-type
  - inline:指定包体是以 inline 内联的方式，作为页面的一部分展示
  - attachment:指定浏览器将包体以附件的方式下载
    - 例如: Content-Disposition: attachment
    - 例如: Content-Disposition: attachment; filename=“filename.jpg”

- 在 multipart/form-data 类型应答中，可以用于子消息体部分
  - 如 Content-Disposition: form-data; name="fieldName"; filename="filename.jpg"

**HTML FORM 表单**

- action:提交时发起 HTTP 请求的 URI

- method:提交时发起 HTTP 请求的 http 方法
  - GET:通过 URI，将表单数据以 URI 参数的方式提交
  - POST:将表单数据放在请求包体中提交
- enctype:在 POST 方法下，对表单内容在请求包体中的编码方式 
  - application/x-www-form-urlencoded
    - 数据被编码成以 ‘&’ 分隔的键-值对, 同时以 ‘=’ 分隔键和值，字符以 URL 编码方式编码
  - multipart/form-data
    - boundary 分隔符
    - 每部分表述皆有HTTP头部描述子包体，例如 Content-Type
    - last boundary 结尾
- Content-type 头部指明这是一个多表述包体
  - Content-type: multipart/form-data;boundar y=----WebKitFormBoundar yRRJKeWfHPGrS4LKe
- Boundary 分隔符的格式
  -  boundary := 0*69<bchars> bcharsnospace
    - bchars := bcharsnospace / " "
    - bcharsnospace:=DIGIT/ALPHA/"'"/"("/")"/"+"/"_"/","/"-"/"."/"/"/":" / "=" / "?"
- multipart-body = preamble 1*encapsulation close-delimiter epilogue
  - preamble := discard-text
  - epilogue := discard-text
    - discard-text := *(*text CRLF)
  - 每部分包体格式:encapsulation = delimiter body-part CRLF
    - delimiter = "--" boundary CRLF
    - body-part = fields *( CRLF *text )
      - field = field-name ":" [ field-value ] CRLF
        - content-disposition: form-data; name="xxxxx“
        - content-type 头部指明该部分包体的类型
  - close-delimiter = "--" boundary "--" CRLF

### 6. 多线程、断点续传、随机点播等场景的实现

**HTTP Range规范**

- 允许服务器基于客户端的请求只发送响应包体的一部分给到客户端，而客户端 自动将多个片断的包体组合成完整的体积更大的包体
  - 支持断点续传
  - 支持多线程下载
  - 支持视频播放器实时拖动
- 服务器通过 Accept-Range 头部表示是否支持 Range 请求
  - Accept-Ranges = acceptable-ranges
  - 例如:
    - Accept-Ranges: bytes:支持
    - Accept-Ranges: none:不支持

**Range 条件请求**

- 客户端请求
  - 如果客户端已经得到了 Range 响应的一部分，并想在这部分响应未过期 的情况下，获取其他部分的响应
    - 常与 If-Unmodified-Since 或者 If-Match 头部共同使用
  - If-Range = entity-tag / HTTP-date 
    - 可以使用 Etag 或者 Last-Modified
- 服务器端响应
  - 206 Partial Content
    -  Content-Range 头部:显示当前片断包体在完整包体中的位置
    -  Content-Range = byte-content-range / other-content-range
      - byte-content-range = bytes-unit SP ( byte-range-resp / unsatisfied-range )
        -  byte-range-resp = byte-range "/" ( complete-length / "*" )*
        - *complete-length = 1*DIGIT
          - 完整资源的大小，如果未知则用*号替代
        - byte-range = first-byte-pos "-" last-byte-pos 
  - 416 Range Not Satisfiable
    - 请求范围不满足实际资源的大小，其中 Content-Range 中的 complete- length 显示完整响应的长度，例如:
      - Content-Range: bytes */1234
  - 200 OK
    - 服务器不支持 Range 请求时，则以 200 返回完整的响应包体
- 多重范围与 multipart
  - 请求:
    - Range: bytes=0-50, 100-150
  - 响应:
    - Content-Type:multipart/byteranges; boundary=...

**Range（随机点播）例子**

客户端请求报文

```http
GET /06 HTTP/1.1
Host: 10.160.105.77:8080
Connection: keep-alive
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.114 Safari/537.36
Accept-Encoding: identity;q=1, *;q=0
Accept: */*
Referer: http://10.160.105.77:8080/06
Accept-Language: zh-CN,zh;q=0.9
Range: bytes=333938688-476582243
If-Range: Fri, 26 Feb 2021 03:14:06 GMT
```

服务器端响应报文

```http
HTTP/1.1 206
Last-Modified: Fri, 26 Feb 2021 03:14:06 GMT
Accept-Ranges: bytes
Date: Tue, 13 Apr 2021 06:38:30 GMT
Content-Type: video/mp4
Content-Range: bytes 333938688-476582243/476582244
Content-Length: 142643556
```

服务器处理代码

```java
public void handleRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    Resource resource = this.getResource(request);
    if (resource == null) {
        logger.debug("Resource not found");
      	// 文件不存在
        response.sendError(404);
    } else if (HttpMethod.OPTIONS.matches(request.getMethod())) {
        response.setHeader("Allow", this.getAllowHeader());
    } else {
        this.checkRequest(request);
        if ((new ServletWebRequest(request, response)).checkNotModified(resource.lastModified())) {
            logger.trace("Resource not modified");
        } else {
            this.prepareResponse(response);
            MediaType mediaType = this.getMediaType(request, resource);
          	// HEAD仅获得首部
            if ("HEAD".equals(request.getMethod())) {
                this.setHeaders(response, resource, mediaType);
            } else {
                ServletServerHttpResponse outputMessage = new ServletServerHttpResponse(response);
              	// 没有Range就不是条件请求
                if (request.getHeader("Range") == null) {
                    Assert.state(this.resourceHttpMessageConverter != null, "Not initialized");
                    this.setHeaders(response, resource, mediaType);
                    this.resourceHttpMessageConverter.write(resource, mediaType, outputMessage);
                } else {
                    Assert.state(this.resourceRegionHttpMessageConverter != null, "Not initialized");
                    response.setHeader("Accept-Ranges", "bytes");
                    ServletServerHttpRequest inputMessage = new ServletServerHttpRequest(request);

                    try {
                        List<HttpRange> httpRanges = inputMessage.getHeaders().getRange();
                        response.setStatus(206);
                        this.resourceRegionHttpMessageConverter.write(HttpRange.toResourceRegions(httpRanges, resource), mediaType, outputMessage);
                    } catch (IllegalArgumentException var8) {
                        response.setHeader("Content-Range", "bytes */" + resource.contentLength());
                        response.sendError(416);
                    }
                }

            }
        }
    }
}
```


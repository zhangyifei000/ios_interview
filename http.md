### HTTP协议

http属于网络结构中的应用层，当初是在wwww中用来进行超文本传输的。后来在使用过程中不断的扩展和完善逐渐被人们利用起来。

特点：

1. 支持c/s模式
1. 简单快速：协议简单，只需传送的请求和方法。
1. 灵活：可以根据content-type传各种数据类型。
1. 无连接：服务器处理完客户请求，并收到客户应答后及断开连接。
1. 无状态：单纯的使用http基础请求头发送两次请求，服务器不知道是否是同一个用户发起的。但是可以使用cookie或者session.这样就有状态了。

### Http版本之间的区别

1. 0.9版只有一个get命令用来请求服务器端的html,服务器发送完毕就关闭tcp连接。
1. 1.0 增加了post和head命令，除了数据部分外，每次请求还要加上请求头header,还增加了缓存，多部分发送，状态码。

     1.Content-Type

    服务器来告诉客户端返回的数据类型。

    * text/plain
    * text/html
    * text/css
    * image/jpeg
    * image/png
    * image/svg+xml
    * audio/mp4
    * video/mp4
    * application/javascript
    * application/pdf
    * application/zip
    * application/atom+xml

    2. Content-Encoding

    由于发送的数据可以是任何格式的，所以碰到大数据的时候可以进行压缩发送

    * Content-Encoding: gzip
    * Content-Encoding: compress
    * Content-Encoding: deflate

    客户端在请求的时候可以告诉服务器端自己接受什么样的压缩格式。

    Accept-Encoding: gzip, deflate

    由于http1.0每次连接只能发送一个请求，每次请求又进行三次握手。所以是比较消耗性能的。有的浏览器就提出了一个非标准的Connection字段。

    Connection: keep-alive

3. 1.1增加了持久连接，既Tcp连接默认不关闭，可以被多个请求复用，客户端和服务器端发现没有活动连接就可以关闭连接，客户端可以向服务器端发送关闭连接请求connection: close,管道机制：同一个tcp里可以发送多个请求。可以进行分块传输，最后返回一个大小为零的块代表传送完了。虽然1.1版本中可以对一个Tcp连接进行复用，但是数据通信是按序进行的。队列阻塞模式。

4. http 2.0

    1.1版本中请求头是文本格式的，数据体是文本或者二进制的。2.0则都是二进制的。为了解析方便。
    双向的，实时的通信。同时处理多个请求，无需按照顺序一一对应
    由于无状态，所以请求头中很多信息都要重复传，很浪费资源，头信息进行压缩在传送，客户端和服务器端都维护一张信息表。
    允许客户端没有请求，主动像客户端发送数据，服务器推送。

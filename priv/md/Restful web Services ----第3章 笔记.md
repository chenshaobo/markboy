
##Restful web Services ----第3章 笔记
###设计表述
客户端所关心的资源是一个抽象的实体，它是用URI来标识的。另一方面，表述是具体而真实的，你在客户端和服务器上针对它编写代码，进行操作。
HTTP在请求和响应中为表述提供了一种包装格式。设计表述设计：
1.使用HTTP提供的格式包含正确的标头，
2.当表述有正文时，为正文选择合适的媒体类型并设计一种格式。

###如何使用实体头来注解表述
表述不仅仅是以某种格式序列化后的数据，它是一连串字节加上用于描述那些字节的元数据。在HTTP中，表述元数据是由使用实体头的名值对来实现的。
使用以下标头来注解包含消息正文的表述：
- `Conent-Type`，用于描述表述类型，包含charset 参数或其他针对该媒体类型而定义的参数。
- `Content-Length`,用于指定表述正文的字节大小。HTTP1.1支持一种更有效的机制，名为分块转移编码（chuncked transfer encoding)
- `Content-Language`,如果您以某种语言对表述进行本地化，用该标头来指定语言。
- `Content-MD5`，工具/软件 在处理或存储表述时，可能存在错误，需要提供一致性校验，用该标头来包含一个表述正文的MD5摘要,在进行内容编码(gzip,compress等）之后，转移编码（即chunked）之前计算摘要值。请注意，TCP使用checksum在传输层提供一致性校验。
- `Content-Encoding`，当你使用gzip，compress或deflate对表述正文进行编码时，使用该标头。
- `Last-Modified` ，用来说明服务器修改表述或资源的最后时间。

HTTP的设计是这样的，发送方可以用一系列名为实体头的标头来描述表述
正文（也称为实体正文或消息正文）。有了这些标头，接收方可以在无须查看正文的情况下决定如何处理正文。它们还可以将解析正文所需要提前了解及猜测的内容减到最小程度。

###如何解释实体头
当服务器或客户端接受到表述时，在处理请求前正确地解释实体头是很重要的。本节讨论了如何从所包含的标头中解释表述。

- `Content-Type` :接收到不带Content-Type的表述时，客户端将其视为不正确的响应，服务器返回错误码400
- `Content-Length`在没有确定接受到的表述不带Transfer-Encoding：chunked前，不要检查Content-Length头是否存在。
- `Content-Encoding` 让您的网络库代码来解压那些压缩过的表述。
- `Content-Language` 如果存在该标头，读取并存储它的值，记录下所使用的语言。


###3.6如何设计JSON表述
 在每个表述中，包含一个指向该资源的self链接，对于那些组成资源的应用程序领域实体，在表述中包含它们的标识符。如果表述中的对象是本地化的，添加一个属性来表示本地化内容的语言。

###如何设计集合表述
客户端会在集合成员中进行迭代，由于某些集合会包含大量成员资源，客户端需要对集合进行分页或滚动显示，在每个集合表述中包含以下内容：
- 一个指向集合资源的self链接
- 如果集合是分页的，并且还有下一页，要有一个指向下一页的链接。
- 如果集合是分页的，并且还有上一页，要有一个指向上一页的链接。
- 一个集合大小的指示符


###3.11如何在表述中编码二进制数据
使用multipart/mixed,multipart/related 或multipart/alternative这样的分段媒体类型，避免使用Base64编码将二进制数据与文本格式编码在一起。
分段消息让你能把不同格式的数据整合到一个HTTP消息中。一个分段消息包含多个消息部分，由边界进行分割，每个部分都可以包含不同媒体类型的消息。
```Content-type：multipart/mixed;boundary="abcd"
--abcd
Content-type:application/xml;charset=UTF-8
<movie> ... <./movie>
--abcd
Content-type:video/mpeg
<video></video>
--abcd--
```
该分段消息有两个部分，一部分包含了一个XML文档，另一部分包含一个视频。
分段媒体类型：
`multipart/form-data `   把键值对数据与任意媒体类型的数据编码在一起，与您用HTML表单上传文件时的用法一致。
`multipart/mixed  `         分段消息把application/xml表示的电影元数据和video/mpeg表示的视频整合进了一个HTTP消息中
`multipart/alternative   `
`multipart/related  `      当消息的各个部分是互相关联的，而且需要一起处理这些部分时，使用该媒体类型，第一部分是根，可以通过Content-ID头引用其他部分



###如何返回错误
HTTP    基于表述的交换，对于错误来说也是如此，当服务器发生错误时，都会返回一个反映错误在的表述。
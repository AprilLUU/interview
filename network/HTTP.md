## HTTP

应用层的传输协议，无连接无状态，明文传输

HTTP不安全 明文传输，内容可能被窃听，不校验身份，可以遭遇伪装，无法验证报文完整性

#### 状态码

1xx 中间状态，需要后续操作

2xx 成功响应

3xx 重定向

4xx 客户端错误

5xx 服务器端错误

#### HTTP缓存

强缓存 Cache-Control Expries

协商缓存 Etag If-None-Match Last-Modified If-Modified-Since

#### HTTP版本

HTTP1.0 短链接 每次请求都需要建立TCP连接

HTTP1.1 长连接 建立TCP连接之后可以复用，流水线方式，一次可以发送多个请求，非流水线方式，只能等待一个请求的响应到达之后再发送下一个请求，非流水线的方式解决了请求的队头阻塞问题，但没有解决响应的队头阻塞，服务器必须按请求的先后到达顺序依次进行处理

HTTP2 基于HTTPS，可以保障安全

HTTP2队头阻塞

![1651303942231](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\1651303942231.png)

对HTTP1.1的改进 

1. 头部压缩，对多个请求的相同头部进行压缩， `HPACK` 算法：在客户端和服务器同时维护一张头信息表，所有字段都会存入这个表，生成一个索引号，以后就不发送同样字段了，只发送索引号，这样就**提高速度**了。 
2. 采用二进制格式， 头信息和数据体都是二进制，并且统称为帧（frame）：**头信息帧（Headers Frame）和数据帧（Data Frame）**。  直接解析二进制报文，这**增加了数据传输的效率**。 
3. ![1651303717866](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\1651303717866.png)



4. 多路复用，可以并发多个请求和响应，解决队头阻塞

   ![1651303799337](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\1651303799337.png)

#### HTTP3把HTTP2的传输层改成基于UDP的QUIC

#### HTTPS

在HTTP和TCP加入一层SSL/TSL协议进行加密传输

信息加密，采用混合加密的方式对数据进行加密传输

校验机制，采用摘要算法生成唯一的指纹用于校验，保证数据完整性

身份证书，需要到CA申请数字证书，将服务器公钥放入数字证书中，进行身份校验

对称加密，加密解密都是采用同一个密钥

非对称加密，用公钥进行加密，私钥进行加密

通信建立前，数字签名，用私钥对数字签名进行加密，拿到服务器的公钥之后，对数字签名进行解密，验证对方的身份，用对方的公钥对自己的对称密钥进行加密，通过自己的私钥解密拿到对方的对称密钥

通信时使用摘要算法生成指纹，使用对称密钥对指纹和数据进行加密，服务器收到后用对称密钥进行解密，用相同的摘要算法生成指纹，校验指纹是否相同

![1651304551188](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\1651304551188.png)

#### SSL/TSL通信过程（4次通信）

1. 客户端发起加密通信请求

   ![1651305254547](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\1651305254547.png)

2. 服务器收到请求，返回数字证书

   ![1651305287850](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\1651305287850.png)

3. 客户端收到证书，用CA公钥对数字证书进行解密，校验身份，拿到服务器的公钥，生成会话密钥

   ![1651305440042](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\1651305440042.png)

4. 服务器生成会话密钥，随后进行加密通信

![1651305451212](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\1651305451212.png)
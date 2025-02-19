## cookie, session, token

session是基于cookie的，设置签名认证，防止session被篡改

cookie 缺点 明文传输不安全 每次请求都会携带 消耗流量 大小受限 4KB 浏览器外的其他客户端必须手动设置cookie 分布式系统 服务器集群如何保证其他系统和服务器正确解析session

jwt token鉴权（JSON Web Token）

header 一些信息 采用的加密算法 type：jwt base64编码

payload 携带的信息 令牌的签发时间 过期时间 base64编码

signature  密钥 对前两部分和密钥用密钥进行加密


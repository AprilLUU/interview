## XSS（跨站脚本注入）

#### 1. 非持久型XSS

​	往页面注入脚本,URL附加脚本

#### 2. 持久型XSS

​	表单提交等，往数据库中注入不安全的脚本，盗取用户信息

#### 3. 如何防御

​	1）CSP（内容安全策略），设置http头建立白名单

​		 Content-Security-Policy: default-src 'self' 

​	2）不要使用不安全的方式注入HTML，CSS，JS，保证页面渲染的数据都可控（不要使用eval， new Function（）等可以执行字符串代码的函数）,或者对字符串参数进行转义

​	3）cookie HttpOnly 设置cookie只在http协议中进行传输，不可以在客户端获取

​	4）cookie security  设置cookie只能通过https协议进行传输

vue中不要使用v-html去渲染用户输入的内容，innerHTML，进行转义净化，服务端也要对数据进行转义净化

保证用户身份信息cookie不可被获取

对用户输入内容进行过滤，防止使用动态执行用户输入的字符串

## CSRF（跨站请求伪造）

登录A站验证完身份后，没有登出A站（保存了cookie），B站向A站发出请求，浏览器携带对应的cookie去请求A站，因为携带用户信息会请求成功，从而可以进行操作

#### 如何防御

​	1） 同源策略，跨站请求不成功

​	2） 不让第三方网站访问到cookie， cookie设置SameSite，只在同域请求携带cookie

​	3） 请求携带验证信息， 验证码或者token校验

​	4） HTTP Referer是header的一部分，当浏览器向web服务器发送请求时，一般会带上Referer信息告诉服务器是从哪个页面链接过来的，服务器籍此可以获得一些信息用于处理。可以通过检查请求的来源来防御CSRF攻击。正常请求的referer具有一定规律，如在提交表单的referer必定是在该页面发起的请求。所以**通过检查http包头referer的值是不是这个页面，来判断是不是CSRF攻击**。

 

但在某些情况下如从https跳转到http，浏览器处于安全考虑，不会发送referer，服务器就无法进行check了。若与该网站同域的其他网站有XSS漏洞，那么攻击者可以在其他网站注入恶意脚本，受害者进入了此类同域的网址，也会遭受攻击。出于以上原因，无法完全依赖Referer Check作为防御CSRF的主要手段。但是可以通过Referer Check来监控CSRF攻击的发生。
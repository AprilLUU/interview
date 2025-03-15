## Node

#### 1. Node全局对象

global，与浏览器window对象类似

#### 2. require查找细节

- require('内置模块'),直接查找内置模块并返回

- require('./../ /X'),查找相对路径或绝对路径
  - 如果有后缀名，直接查找对应的文件
  - 没有后缀名
    - 查找文件X
    - 查找X.js文件
    - 查找X.json文件
    - 查找X.node文件
    - 当成目录X，查找index.js/json/node

- require("X")，从node_modules下查找

#### 3. CommonJS导出细节

- 源码内module.exports = exports

- exports并非最终导出的对象，module.exports为最终的导出对象
- require导入的对象即为module.exports，且为同一个引用
- require只会在第一次加载模块时执行，之后会缓存
- require是运行时加载，当模块之间互相依赖，就会形成依赖图，采用的是深度优先搜索

#### 4. ESM和CJS的区别

- ESM是静态解析的，编译时加载，且这个过程是异步的，不影响后续代码执行（type = module是异步的）
- CJS是动态加载的，同步加载，会阻塞后续代码

#### 5. 常用模块

- path模块，路径操作
- fs模块，操作文件系统，有异步操作API，也有同步API
- events模块，事件驱动模块，事件总线
- child_process模块，创建子进程，结合做过的项目
- http模块，创建http服务器
- cryptp模块，加密模块，结合项目MD5加密，密码通过md5加密之后存储到数据库（加盐）

#### 6. CLI工具开发

- 基于commander库进行命令行解析，解析对应的命令，执行脚本
- 创建项目命令，create <project>（动态参数）
  - 配置项目模板，放置到git仓库
  - 执行脚本命令
    - 拉取git模板
    - 创建子进程执行命令 git init 初始化git仓库
    - 创建子进程执行命令 npm install 安装依赖
    - 创建子进程执行命令 npm run prettier 格式化代码
    - 创建子进程执行命令 npm run serve 开启项目服务
    - 打开浏览器
- 创建组件命令，add2Cpn <name> -d 目标路径
  - 准备好ejs模板
  - 编译ejs模板，填入动态参数组件名
  - 判断目录是否存在，不存在则创建，然后写入文件到目标路径，异步写入不阻塞

#### 7. buffer和Stream

二进制相关的缓冲数组和读写流

#### 8. express

- multer中间件，处理form-data数据

#### 9. koa

- koa-multer中间件，处理form-data数据

#### 10. express和koa的区别

- 中间件机制：Koa支持async和await处理异步中间件，能更优雅地处理异步流程
- 核心设计：Koa更轻量，Express内置许多中间件和功能
- 错误处理：Express通过next(err)将错误传递到后续中间件处理，Koa可使用try，catch直接捕获错误，也可以将触发error事件emit(error)，通过on("error")监听error事件统一处理错误
- 上下文：express请求和响应对象分离req和res，koa统一请求和响应对象到ctx

#### 11. MVC和MVVM

**MVC**

- Model，对应Service层，负责与数据库进行交互
- View，视图层，仅展示从Model层获取的数据
- Controller，负责处理请求

Restful API HTTP方法代表操作 动态参数id代表操作的对象 URL代表操作的资源

**MVVM**

- Model，对应数据
- View，对应UI视图
- ViemModel，代表连接两者的桥梁，由框架来替代这个角色

#### 12. 登录凭证

JWT，私钥用来签证发布令牌，公钥用来验证令牌，私钥不泄露就无法伪造令牌

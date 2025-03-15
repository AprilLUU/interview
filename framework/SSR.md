## SSR

1. 由服务端渲染首页HTML字符串，但不具备交互
2. 返回首页HTML和客户端JS，接下来一切交互，包括路由切换等前端交互交由客户端执行（仅在第一次请求时进行服务端渲染，刷新浏览器时也是一样）

```javascript
let express = require("express");
let server = express();
import createApp from "../app";
import { renderToString } from "@vue/server-renderer";
import createRouter from "../router";
// 内存路由-> node用
import { createMemoryHistory } from "vue-router";
import { createPinia } from 'pinia'
// 部署 静态资源
server.use(express.static("build"));

let count = 0
// /  or /about
server.get("/*", async (req, res) => {
  console.log(++count)
  console.log(req.url)
  let app = createApp();

  // app 安装路由插件
  let router = createRouter(createMemoryHistory());
  app.use(router);
  await router.push(req.url || "/"); //  /  or /about 等待页面跳转好
  await router.isReady(); // 等待(异步)路由加载完成,在渲染页面

  // app 安装pinia插件
  let pinia = createPinia()
  app.use(pinia)

  let appStringHtml = await renderToString(app);
  console.log(appStringHtml)
  res.send(
    `
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta http-equiv="X-UA-Compatible" content="IE=edge">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
    </head>
    <body>
      <h1>Vue3 Serve Side Render</h1>
      <div id="app">
        ${appStringHtml}
      </div>
      <script src="/client/client_bundle.js"></script>
    </body>
    </html>
    `
  );
});

server.listen(3000, () => {
  console.log("start node server on 3000 ~");
});
```

```javascript
import { createApp } from "vue";
import { createWebHistory } from "vue-router";
import App from "../App.vue";
import createRouter from "../router";
import { createPinia } from "pinia";
// spa
let app = createApp(App);

// 安装路由插件
let router = createRouter(createWebHistory());
app.use(router);

// app 安装pinia插件
let pinia = createPinia();
app.use(pinia);

router.isReady().then(() => {
  app.mount("#app");
});
```

```javascript
import { createSSRApp } from "vue";
import App from "./App.vue";

// 这里为什么写一个函数来返回app实例 ?
// 通过函数来返回app实例,可以保证每个请求都会返回一个新的app实例, 来避免 跨请求状态的污染
export default function createApp() {
  let app = createSSRApp(App);
  return app;
}

```


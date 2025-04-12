## 事件循环

#### 1. 浏览器事件循环

事件循环周期

1. 执行同步代码
2. 同步代码执行完毕，从任务队列中取出任务执行
3. 先清空微任务队列
4. 取出宏任务队列中的任务执行，每执行完一个宏任务，检查微任务队列是否有新任务并执行
5. 如果有UI更新，则更新UI

宏任务：定时器，事件监听器，I/O操作（Ajax），动态创建script脚本（script标签）

微任务：promise，queueMicrotack，MutationObserver 回调

- **MutationObserver 核心作用**

  监听 **DOM 节点变化**（如属性修改、子节点增删等），并通过**微任务队列**异步触发回调，避免阻塞主线程。

微任务的存在意义是允许插队执行，具有较高的优先级，适合尽快执行的小任务

#### 2. node事件循环

- nodejs也是单线程模型

- 通过libuv库的线程池执行阻塞和耗时操作，比如文件读取，网络请求等

- 采用非阻塞式系统调用（线程不会处于阻塞态，异步非阻塞IO），线程轮询拿到执行结果，把回调推入队列，由主线程执行

执行顺序与浏览器类似，只不过微任务和宏任务队列划分得更细

微任务

1. process.nexttick()

2. promise.then() queueMicrotask()

宏任务

1. timer队列，setTimeout，setInterval
2. I/O callbacks，待定队列，对某些系统操作执行回调，除了close事件的回调，阻塞操作（文件读取）的回调。例如error的回调
3. idle，系统内部调用
4. poll，轮询队列，通常会阻塞在这里，IO事件，文件读写，网络IO等
5. check，检测队列，setImmediate
6. close，关闭回调，例如socket.on("close", () => {})

**`setImmediate` vs `setTimeout`**：

- 主模块中顺序不确定（受事件循环启动时间影响），事件循环启动事件大于1ms时，timer会先执行，否则推入下一次循环。
- **在 I/O 回调中**，`setImmediate` 总是先于 `setTimeout`，因为poll下一个阶段就是check。

setTimeout(() => {}, 0)

setImmediate(() => {})

事件循环启动需要时间，一般是setImmediate先执行，setTimeout(() => {}, 0)会推入下一次循环

```javascript
/*
【1】同步代码开始
【2】同步代码结束
【3】nextTick 1
【4】Promise 1
【5】Timeout 1
【6】Immediate 1
【7】文件读取完成
【8】nextTick 2
【9】Immediate 2
【10】Timeout 2
*/

const fs = require("fs")

// 1. 同步代码
console.log("【1】同步代码开始")

// 2. 定时器（Timers 阶段）
setTimeout(() => console.log("【5】Timeout 1"), 0)

// 3. setImmediate（Check 阶段）
setImmediate(() => console.log("【6】Immediate 1"))

// 4. Promise 微任务（微任务队列）
Promise.resolve().then(() => console.log("【4】Promise 1"))

// 5. process.nextTick（nextTick 队列）
process.nextTick(() => console.log("【3】nextTick 1"))

// 6. 异步文件读取（Poll 阶段）
fs.readFile(__filename, () => {
  console.log("【7】文件读取完成")

  // 在 I/O 回调中的定时器和 setImmediate
  setTimeout(() => console.log("【10】Timeout 2"), 0)
  setImmediate(() => console.log("【9】Immediate 2"))
  process.nextTick(() => console.log("【8】nextTick 2"))
})

// 7. 同步代码结束
console.log("【2】同步代码结束")
```


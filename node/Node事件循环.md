## 事件循环

#### 1. 浏览器事件循环

事件循环周期

1. 执行同步代码
2. 同步代码执行完毕，从任务队列中取出任务执行
3. 先清空微任务队列
4. 取出宏任务队列中的任务执行，每执行完一个宏任务，检查微任务队列是否有新任务并执行
5. 如果有UI更新，则更新UI

宏任务：定时器，事件监听器，I/O操作（Ajax）等

微任务：promise，queueMicrotack

微任务的存在意义是允许插队执行，具有较高的优先级，适合尽快执行的小任务

#### 2. node事件循环

libuv库，轮询拿到线程执行结果，把回调推入队列，由主线程执行

执行顺序与浏览器类似，只不过微任务和宏任务队列划分得更细

微任务

1. process.nexttick()

2. promise.then() queueMicrotask()

宏任务

1. timer队列，setTimeout，setInterval
2. 待定队列，对某些系统操作执行回调
3. 系统内部调用
4. 轮询队列，通常会阻塞在这里，IO事件，文件读写，网络IO等
5. 检测队列，setImmediate
6. 关闭回调，例如socket.on("close", () => {})

setTimeout(() => {}, 0)

setImmediate(() => {})

执行顺序，如果setTimeout函数的执行时间大于本次事件循环开启的时间，则先执行setImmediate

如果小于，则按顺序执行

一般是setImmediate先执行，setTimeout(() => {}, 0)会推入下一次循环

## Node事件循环

libuv 非阻塞式调用 轮询拿到线程执行结果 把回调推入队列 由主线程执行

microtask 

process.nexttick()

promise.then() queueMicrotask()

宏任务

1. timer队列，setTimeout，setInterval
2. 待定队列，对某些系统操作执行回调
3. 系统内部调用
4. 轮询队列，通常会阻塞在这里，IO事件
5. 检测队列，setImmediate
6. 关闭回调

setTimeout(() => {}, 0)

setImmediate(() => {})

执行顺序，如果setTimeout函数的执行时间大于本次事件循环开启的时间，则先执行setImmediate

如果小于，则按顺序执行
## JS手写题

#### 1. call

```javascript
Function.prototype.myCall = function(thisArg, ...args) {
  let fn = this

  if (thisArg !== null && thisArg !== undefined) {
    thisArg = Object(thisArg)
  } else {
    thisArg = window
  }
  
  thisArg.fn = fn
  const result = thisArg.fn(...args)
  delete thisArg.fn

  return result
}
```

#### 2. apply

```javascript
Function.prototype.myApply = function(thisArg, argArray) {
  let fn = this

  if (thisArg !== null && thisArg !== undefined) {
    thisArg = Object(thisArg)
  } else {
    thisArg = window
  }
  
  thisArg.fn = fn
  const argArray = argArray || []
  const result = thisArg.fn(...argArray)
  delete thisArg.fn
  
  return result
}
```

#### 3. bind

```javascript
Function.prototype.myBind = function(thisArg, ...argArray) {
  let fn = this

  if (thisArg !== null && thisArg !== undefined) {
    thisArg = Object(thisArg)
  } else {
    thisArg = window
  }

  function proxyFn(...args) {
    thisArg.fn = fn
    const finalArgs = [...argArray, ...args]
    const result = thisArg.fn(...finalArgs)
    delete thisArg.fn
    return result
  }

  return proxyFn
}
```

#### 4. new

```javascript
function createObject(constructor, ...args) {
  const newObj = Object.create(constructor.prototype)
  const result = constructor.apply(newObj, args)
  const isObj = result && (typeof result === 'object' || typeof result === 'function')
  return isObj ? result : newObj
}
```

#### 5. 防抖

- 先写出简单版本
- 优化返回值
- 封装首次执行功能
- 封装取消功能

```javascript
function debounce(fn, delay, immediate = false) {
  let timer = null
  let isInvoke = false
  
  const execFunWithError = function(fun, thisArg, args, resolve, reject) {
    try {
      const result = fun.apply(thisArg, args)
      resolve(result)
    } catch(err) {
      reject(err)
    }
  }

  const _debounce = function(...args) {
    return new Promise((resolve, reject) => {
      if (timer) {
        clearTimeout(timer)
      }
  
      if (immediate && !isInvoke) {
        execFunWithError(fn, this, args, resolve, reject)
        isInvoke = true
      } else {
        timer = setTimeout(() => {
          execFunWithError(fn, this, args, resolve, reject)
          timer = null
          isInvoke = false
        }, delay)
      }
    })
  }

  _debounce.cancel = function() {
    if (timer) clearTimeout(timer)
    timer = null
    isInvoke = false
  }

  return _debounce
}
```

#### 6. 节流

- 先写出简单版本
- 优化返回值
- 封装首次执行
- 封装取消功能

```javascript
function throttle(fn, interval, optinons = { leading: true, trailing: false }) {
  let lastTime = 0
  let timer = null
  const { leading, trailing } = optinons

  const _throttle = function(...args) {
    return new Promise((resolve, reject) => {
      const nowTime = new Date().getTime()
      if (!lastTime && !leading) lastTime = nowTime
      
      const remainTime = interval - (nowTime - lastTime)
      if (remainTime <= 0) {
        if (timer) {
          clearTimeout(timer)
          timer = null
        }
        const result = fn.apply(this, args)
        reslove(result)
        lastTime = nowTime
        return
      }
  
      if (trailing && !timer) {
        timer = setTimeout(() => {
          const result = fn.apply(this, args)
          reslove(result)
          lastTime = !leading ? 0 : new Date().getTime()
          timer = null
        }, remainTime)
      }
    })
  }

  _throttle.cancel = function() {
    if (timer) clearTimeout(timer)
    timer = null
    lastTime = 0
  }

  return _throttle
}
```

#### 7. 深拷贝

```javascript
function isObject(value) {
  const valueType = typeof value
  return (value !== null) && (valueType === 'object' || valueType === 'function')
}
 
function deepClone(originValue, map = new WeakMap()) {
  if (originValue instanceof Set) {
    return new Set([...originValue])
  }

  if (originValue instanceof Map) {
    return new Map([...originValue])
  }

  if (typeof originValue === 'symbol') {
    return Symbol(originValue.description)
  }

  if (typeof originValue === 'function') {
    return originValue
  }

  if (!isObject(originValue)) {
    return originValue
  }

  if (map.has(originValue)) {
    return map.get(originValue)
  }

  const newObj = Array.isArray(originValue) ? [] : {}
  map.set(originValue, newObj)
  for (const key in originValue) {
    newObj[key] = deepClone(originValue[key], map)
  }

  const sKeys = Object.getOwnPropertySymbols(originValue)
  for (const sKey of sKeys) {
    newObj[sKey] = deepClone(originValue[sKey], map)
  }

  return newObj
}
```

#### 8. 事件总线

```javascript

export class EventBus {
  constructor() {
    this.eventBus = {}
  }

  on(eventName: string, eventHandler: fn, context?: any) {
    let handlers = this.eventBus[eventName]
    if (!handlers) {
      handlers = []
      this.eventBus[eventName] = handlers
    }
    handlers.push({ eventHandler, context })
    return this
  }

  once(eventName: string, eventHandler: fn, context?: any) {
    const onceCallback = (...payload: any[]) => {
      this.off(eventName, onceCallback)
      eventHandler.apply(context, payload)
    }

    return this.on(eventName, onceCallback, context)
  }

  emit(eventName: string, ...payload: any[]) {
    const handlers = this.eventBus[eventName]
    if (!handlers) {
      throw new Error(`${eventName} is not register`)
    }
    handlers.forEach(handler => {
      handler.eventHandler.apply(handler.context, payload)
    })
    return this
  }

  off(eventName: string, eventHandler: fn) {
    const handlers = this.eventBus[eventName]
    if (!handlers) {
      throw new Error(`${eventName} is not register`)
    }

    const newHandlers = handlers
    for (let i = 0; i < newHandlers.length; i++) {
      const handler = newHandlers[i]
      if (handler.eventHandler === eventHandler) {
        const index = handlers.indexOf(handler)
        handlers.splice(index, 1)
      }
    }

    if (handlers.length === 0) {
      delete this.eventBus[eventName]
    }

    return this
  }
}
```

#### 9. eventStore

```javascript
export class EventStore {
  state: IStateType
  actions: IActionType
  eventBus: EventBus

  constructor(options: IStoreOptions) { 
    this.state = this._observer(options.state ?? {})
    this.actions = options.actions ?? {}
    this.eventBus = new EventBus()
  }

  _observer(state: IStateType) {
    // 浅层监听，没有深度监听
    return new Proxy(state, {
      get(target, key, recevier) {
        return Reflect.get(target, key, recevier)
      },
      set(target, key, newValue, recevier) {
        Reflect.set(target, key, newValue, recevier)
        if (this.eventBus.eventBus[key]) {
          this.eventBus.emit(key, newValue)
        }
        return true
      }
    })
  }

  onState(stateKey: string, stateCallback: stateCallback) {
    const keys = Object.keys(this.state)
    if (keys.indexOf(stateKey) === -1) {
      throw new Error("the state does not contain your key")
    }

    this.eventBusV1.on(stateKey, stateCallback)
    const value = this.state[stateKey]
    stateCallback.apply(this.state, [value])
  }

  dispatch(actionName: string, ...payload: any[]) {
    const keys = Object.keys(this.actions)
    if (keys.indexOf(actionName) === -1) {
      throw new Error(`${actionName} is exsit on actions`)
    }

    const action = this.actions[actionName]

    action.apply(this.state, payload)
  }
}
```

#### 10. Promise API

```javascript
class MyPromise {
  static resolve(value) {
    return new MyPromise(resolve => resolve(value))
  }

  static reject(reason) {
    return new MyPromise(reject => reject(reason))
  }

  static all(promises) {
    return new MyPromise((resolve, reject) => {
      const values = []
      promises.forEach((promise, index) => {
        promise.then(res => {
          // 调用splice，保证每个promise中的结果在数组中的顺序
          values.splice(index, 0, res)
          if (values.length === promises.length) {
            resolve(values)
          }
        }, err => {
          reject(err)
        })
      })
    })
  }

  static allSettled(promises) {
    return new MyPromise(resolve => {
      const results = []
      promises.forEach((promise, index) => {
        promise.then(res => {
          const promiseRes = { status: 'fulfilled', value: res }
          results.splice(index, 0, promiseRes)
          if (results.length === promises.length) {
            resolve(results)
          }
        }, err => {
          const promiseRes = { status: 'rejected', reason: err }
          results.splice(index, 0, promiseRes)
          if (results.length === promises.length) {
            resolve(results)
          }
        })
      })
    })
  }

  static race(promises) {
    return new MyPromise((resolve, reject) => {
      promises.forEach(promise => {
        promise.then(resolve, reject)
      })
    })
  }

  static any(promises) {
    const reasons = []
    return new MyPromise((resolve, reject) => {
      promises.forEach(promise => {
        promise.then(resolve, err => {
          reasons.push(err)
          if (reasons.length === promises.length) {
            reject(new AggregateError('all promises were rejected'))
          }
        })
      })
    })
  }
}
```


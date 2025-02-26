## JavaScript

#### 1. 箭头函数

- 没有this，super，prototype，arguments，this绑定上层作用域
- 不能用作构造函数，不能通过new调用
- 不能用作生成器函数，不能调用yield关键字
- 不能通过call，apply，bind绑定this

#### 2. new调用

1）创建以构造函数原型为原型的对象

```javascript
const obj = Object.create(constructor.prototype)
```

2）设置对象为构造函数的this

```javascript
constructor = constructor.bind(obj)
```

3）执行构造函数

```javascript
const res = constructor()
```

4）如果构造函数没有返回值或者返回值不是对象，返回创建的对象

```javascript
const isObj = res && (typeof res === "object" || typeof res === "function")
return isObj ? res : obj
```


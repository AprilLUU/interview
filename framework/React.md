## React

#### 1. React类组件绑定this

在React类组件种的render函数绑定事件时，会出现this指向丢失的问题，根本原因在于this的绑定规则，当通过赋值的形式传递时，例如const fun = this.fun，JavaScript只会将this.fun的函数体赋值给fun，不会将this进行传递。

解决方案主要有三种：

1. 显示绑定，使用bind函数为类方法绑定this

```jsx
<div onClick={this.click.bind(this)}></div>
```

2. 定义类方法为箭头函数或者使用箭头函数进行调用，箭头函数没有this绑定规则，只跟其定义时的作用域有关

```jsx
class Component {
	click1 = () => {
        console.log(this)
    }
    
    click2() {
        console.log(this)
    }
    
	render() {
		return (
			<div>
                <div onClick={this.click1}></div>
                <div onClick={() => this.click2()}></div>
            </div>
		)
	}
}
```

React18之后推荐使用函数式组件+Hooks进行开发，可以避免this绑定的问题

#### 2. React JSX 条件渲染方式

JSX经过Babel转换之后是React.createElement(type, attr, children)函数，{}内部执行的是JavaScript表达式，和Vue插值语法{{}}一致，因此不支持直接执行语句例如if...else...。

```jsx
render() {
    const { isReady, friend } = this.state

    // 1.条件判断方式一: 使用if进行条件判断
    let showElement = null
    if (isReady) {
      showElement = <h2>准备开始比赛吧</h2>
    } else {
      showElement = <h1>请提前做好准备!</h1>
    }

    return (
      <div>
        {/* 1.方式一: 根据条件给变量赋值不同的内容 */}
        <div>{showElement}</div>

        {/* 2.方式二: 三元运算符 */}
        <div>{ isReady ? <button>开始战斗!</button>: <h3>赶紧准备</h3> }</div>

        {/* 3.方式三: &&逻辑与运算 */}
        {/* 场景: 当某一个值, 有可能为undefined时, 使用&&进行条件判断 */}
        <div>{ friend && <div>{friend.name + " " + friend.desc}</div> }</div>
      </div>
    )
}
```

#### 3. 组件通信

- 父传子：props 

- 子传父：props传递回调函数，Vue中还可以通过自定义事件，子组件定义事件，父组件监听事件，跟事件总线一样

- 非父子：事件总线，全局状态管理， Vue中provide和inject，React中Context类似Vue中project和inject，为子组件注入数据，<Provider>组件包裹根组件，根组件内部所有子孙组件都可以通过<Consumer>组件来获取数据

#### 4. React插槽实现

1. props传递JSX
2. props.children，将JSX作为children传递给子组件
3. 作用域插槽，传递一个函数作为props，由子组件将内部数据传递过来

Vue中插槽实际上为一个函数，执行这个函数返回vnode，将vnode插入对应位置，作用域插槽实际就是在传递函数时传递对应的参数，与React中类似

#### 5. setState同步和异步的问题

setState默认是异步的，可以对DOM进行批量更新，如果是同步的，则每次都会调用render函数重新更新DOM，消耗性能，与Vue中更新DOM的机制类似，Vue通过queueJob(component.update)，将组件的更新加入到调度队列中，通过Promise.then加入微任务队列，同时将任务进行去重，只异步更新一次。

#### 6. 获取原生DOM元素，与Vue获取类似，都是通过给HTML元素绑定ref属性

#### 7. 受控组件，与Vue双向绑定表单元素类似，绑定表单元素的value值，在事件触发时设置新值

#### 8. 非受控组件，表单元素绑定defaultValue设置默认值，新值由DOM节点维护，可通过ref绑定来获取值

#### 9. 高阶组件，装饰器模式，接收一个原始组件，对原始组件进行功能增强，返回一个新组件

#### 10. Portals，与Vue的Teleport类似，都是挂载到指定DOM节点上

#### 11. React动画实现，依赖react-transition-group实现，与Vue类似的实现

- CSSTransition，与Vue类似，都是通过给DOM元素动态添加类名添加CSS属性实现
- SwitchTransition，组件切换时显示动画
- TransitionGroup，执行动画组

#### 12. React CSS

- 内联CSS，直接绑定style对象，与样式结构分离的原则冲突，代码混乱
- CSS文件，容易产生冲突
- CSS Modules，借助构建工具来隔离样式，每个xxx.module.css文件都是一个模块对象，需要用style.xxx来使用
- CSS in JS，直接CSS预处理器的很多功能。styled-components，通过标签模板字符串来进行函数调用，解析CSS，创建一个新的组件，添加上不重复的类名，实现样式隔离。传递属性时需要用forwardProps，来避免将属性绑定到DOM元素上。

#### 13. React Hook

- useState(initValue)，每次调用函数为什么useState返回值state不会变为初始值？React内部做了缓存
- useEffect，在DOM更新之后执行副作用函数，与之对应由useLayoutEffect，在重绘之前触发

1. 每次更新时都会重新执行effect，可以传递第二个参数让useEffect在指定state发生变化重新执行，传[]代表只在卸载时执行一次
2. 多个effect声明，会按声明顺序依次执行（队列？）
3. 返回值为清除effect的函数，由React在更新或卸载组件时执行，传[]代表只在卸载时执行一次

```javascript
useEffect(() => {
    console.log("修改title:", count)
}, [count])

useEffect(() => {
    console.log("监听redux中的数据")
    return () => {}
}, [])

useEffect(() => {
    console.log("监听eventBus的why事件")
    return () => {}
}, [])

useEffect(() => {
    console.log("发送网络请求, 从服务器获取数据")

    return () => {
      console.log("会在组件被卸载时, 才会执行一次")
    }
}, [])
```

- useContext，在函数式组件中获取context的值（取代Consumer），当Context.Provider发生变化时，会触发重新渲染
- useReducer，类似redux的reducer用法，但数据只在组件内生效，不会共享（用法：state更新时需要依赖之前的state）

- useCallback，返回一个记忆化的函数
- useMemo，返回记忆化的值
- useRef，返回的ref对象在组件的整个生命周期保持不变，用法获取DOM节点以及保存数据（需要依赖上一次的值时）
- useImperativeHandle，与forwardRef结合使用，返回暴露给父组件的对象而不是DOM节点

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

父传子：props 

子传父：props传递回调函数，Vue中还可以通过自定义事件，子组件定义事件，父组件监听事件，跟事件总线一样

非父子：事件总线，全局状态管理， Vue中provide和inject，React中Context类似Vue中project和inject，为子组件注入数据，<Provider>组件包裹根组件，根组件内部所有子孙组件都可以通过<Consumer>组件来获取数据

#### 4. React插槽实现

1. props传递JSX
2. props.children，将JSX作为children传递给子组件
3. 作用域插槽，传递一个函数作为props，由子组件将内部数据传递过来

Vue中插槽实际上为一个函数，执行这个函数返回vnode，将vnode插入对应位置，作用域插槽实际就是在传递函数时传递对应的参数，与React中类似

#### 5. setState同步和异步的问题

setState默认是异步的，可以对DOM进行批量更新，如果是同步的，则每次都会调用render函数重新更新DOM，消耗性能，与Vue中更新DOM的机制类似，Vue通过queueJob(component.update)，将组件的更新加入到调度队列中，通过Promise.then加入微任务队列，同时将任务进行去重，只异步更新一次。


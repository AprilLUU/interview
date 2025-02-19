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


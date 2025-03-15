## Router原理

#### 1. hash模式

监听浏览器hashChange事件，根据不同的hash值渲染对应的组件，不会重新刷新浏览器

```javascript
const contentEl = document.querySelector('.content')

window.addEventListener('hashchange', () => {
  switch (location.hash) {
    case '#/home':
      contentEl.innerHTML = 'Home'
      break
    case '#/about':
      contentEl.innerHTML = 'About'
      break
    default:
      contentEl.innerHTML = 'Default'
      break
  }
})
```

#### 2. history模式

使用history API操作url，监听popState事件和go事件，根据location.pathname来渲染不同的组件

```javascript
const contentEl = document.querySelector('.content')

const aEls = document.getElementsByTagName('a')
for (let aEl of aEls) {
  aEl.addEventListener('click', event => {
    event.preventDefault()

    const href = aEl.getAttribute('href')
    const content = href.replace('/', '')
    history.pushState({ content }, '', href)

    changeContent()
  })
}

window.addEventListener('popstate', event => {
  contentEl.innerHTML = event.state ? event.state.content : 'Default'
})

function changeContent() {
  switch(location.pathname) {
    case '/home':
      contentEl.textContent = 'Home'
      break
    case '/about':
      contentEl.textContent = 'About'
      break
    default:
      contentEl.textContent = 'Default'
      break
  }
}
```

#### 3. 组件懒加载，异步组件，可以用Suspense包裹，在未加载时显示占位
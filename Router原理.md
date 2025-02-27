## Router原理

#### 1. hash模式

监听浏览器hashChange事件，根据不同的hash值渲染对应的组件，不会重新刷新浏览器

#### 2. history模式

使用history API操作url，监听popState事件和go事件，根据location.pathname来渲染不同的组件

#### 3. 组件懒加载，异步组件，可以用Suspense包裹，在未加载时显示占位
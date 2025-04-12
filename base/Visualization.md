## 可视化

#### 1. 动画实现方案

- CSS动画，优先方式，可以利用浏览器进行优化（单独创建合成层），transform，opacity，will-change
- Canvas，通过JavaScript来操作Canvas制作动画，可以使用WebGL调用GPU进行加速绘制

- SVG，通过SVG制作动画
- 通过JS动画库制作动画

#### 2. Canvas和SVG的区别

- Canvas基于像素进行图形绘制，SVG基于矢量进行图形描述
- Canvas绘制的图形缩放会失真导致模糊，SVG图形支持无损缩放
- Canvas绘制的图形不是DOM元素，SVG集成到DOM Tree中

**适用场景：**

- Canvas更适用于频繁和动态更新的场景
- SVG更适用于移动端，适合响应式设计和高分辨率显示
- 在处理复杂场景时，如大量图形和复杂动画时，SVG性能可能会更差，Canvas更适用
  - SVG元素基于DOM，当有大量SVG动画时，DOM节点操作开销大，引起频繁重绘和回流
  - SVG动画使用CPU渲染，大量动画需要CPU计算绘制
  - canvas基于像素绘制，可以利用GPU并行处理，获得更高的渲染效率


#### 3. ECharts渲染器选择

- 在需要创建很多 ECharts 实例且浏览器易崩溃的情况下（可能因为 Canvas 数量多导致内存占用超出手机承受能力），可以使用 SVG 渲染器来进行改善
- 数据量较大（经验判断 > 1k）、较多交互时，建议选择 Canvas 渲染器


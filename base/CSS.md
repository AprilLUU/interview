## CSS

#### 1. 盒子模型

content + padding + border + margin

- **W3C盒子模型**

width = content

height = content

- **IE怪异盒模型**

width = content + padding + border

height = content + padding + border

box-sizing: border-box / content-box

#### 2. 浮动的缺点

- 脱离文档流，不会超出父元素，不会超过其前面的浮动元素

- 如果前面的元素没有浮动，无法上移

解决高度塌陷和外边距折叠

**原理**

- 添加after伪类，即在后面添加一个子元素，然后清除浮动，将高度汇报给父元素
- 添加before伪类，即在前面添加一个子元素，然后开启BFC，解决外边距折叠

```css
.clearfix::before,
.clearfix::after{
	content: "";
	display: table;
	clear: both;
}
```

#### 3. BFC

`BFC`（Block Formatting Context），即块级格式化上下文，它是页面中的一块渲染区域，并且有一套属于自己的渲染规则：

- 内部的盒子会在垂直方向上一个接一个的放置
- 对于同一个BFC的俩个相邻的盒子的margin会发生重叠，与方向无关。
- 每个元素的左外边距与包含块的左边界相接触（从左到右），即使浮动元素也是如此
- BFC的区域不会与float的元素区域重叠
- 计算BFC的高度时，浮动子元素也参与计算
- BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素，反之亦然

`BFC`目的是形成一个相对于外界完全独立的空间，让内部的子元素不会影响到外部的元素

触发`BFC`的条件包含不限于：

- **根元素**，即HTML元素
- **浮动元素**：float值为left、right
- **overflow值不为 visible**，为 auto、scroll、hidden
- display的值为inline-block、inltable-cell、table-caption、table、inline-table、flex、inline-flex、grid、inline-grid
- **position的值为absolute或fixed**

BFC的高度是auto的情况下，是如下方法计算高度的

- 如果只有inline-level，是行高的顶部和底部的距离；

- 如果有block-level，是由最顶层的块上边缘和最底层块盒子的下边缘之间的距离

- 如果有绝对定位元素，将被忽略；

- 如果有浮动元素，那么会增加高度以包括这些浮动元素的下边缘

BFC，块级格式化上下文，是一个独立的渲染区域，内部的子元素不会影响外面的元素，同样也不会被外部的元素影响。比如，BFC的渲染规则包括两个相邻的盒子会重叠外边距，两个元素位于同一个BFC就会发生外边距折叠的问题。而常见的解决方案就是开启一个新的BFC，让两个元素处于不同的BFC来解决。

```css
.container {
  overflow: hidden;
}

.box1 {
  height: 100px;
  background-color: #18f;
  margin-bottom: 100px;
}

<div class="container">
	<div class="box1"></div>
</div>
```

BFC的另一个作用就是解决高度塌陷问题，（高度塌陷问题是由于子元素脱离标准流，导致父元素没有获取到高度），但需要特定的条件，就是发生高度塌陷的父元素开启BFC并且height设置为auto，此时BFC的计算高度规则会把浮动子元素的高度也计算进去，但不包括绝对定位等导致的高度塌陷。

#### 4. 浏览器兼容性问题

通过工程化工具解决

1. 设置browserslist设置支持的浏览器，通过caniuse网站查询对应的浏览器

2. 通过postcss以及babel自动添加浏览器前缀和进行代码转换
3. 引入normalize.css以及polyfill等来解决css和js兼容性

#### 5. 物理像素和逻辑像素

物理像素：实际的屏幕像素，比如1920x1080，2k，4k

逻辑像素：由操作系统和浏览器抽象出来的像素单位，根据实际的物理像素进行转化，开发者不必考虑实际的物理像素

CSS像素：逻辑像素的一种

DPR（device pixel ratio）：设备像素比，逻辑像素和物理像素的比例，window.devicePixelRatio

#### 6. 移动端@2x，@3x的图片

通过@2x，@3x设置高分辨率图片到高分辨率的设备上，避免图片分辨率过低造成图片模糊

通过媒体查询resolution查询设备分辨率，**dppx**查询设备像素比

```css
@media only screen and (min-resolution: 2dppx) {
  .box {
    background-image: url(./img/zznh@2x.png);
  }
}
]"
"
@media only screen and (min-resolution: 3dppx) {
  .box {
    background-image: url(./img/zznh@3x.png);
  }
}
```

#### 7. 移动端1px边框

一些设备，旧的移动设备以及浏览器不支持0.5px

解决方案：transform和伪类，通过before或者after伪类，设置200%的宽高和1px的边框，然后通过transform缩放0.5，画出0.5px的边框

```css
.test {
  position: relative;
  display: inline-block;
  margin: 30px;
  padding: 8px;
}

.test::after {
  content: "";
  position: absolute;
  left: 0;
  top: 0;
  width: 200%;
  height: 200%;
  border: 1px solid #f00;
  transform: scale(0.5);
  transform-origin: 0 0;
}
```

#### 8. 移动端适配方案

- 动态获取设备宽度，设置html的fontSize大小，使用相对单位rem

```javascript
// 1.获取html的元素
const htmlEl = document.documentElement

function setRemUnit() {
  // 2.获取html的宽度(视口的宽度)
  const htmlWidth = htmlEl.clientWidth
  // 3.根据宽度计算一个html的font-size的大小
  const htmlFontSize = htmlWidth / 10
  // 4.将font-size设置到html上
  htmlEl.style.fontSize = htmlFontSize + "px"
}
// 保证第一次进来时, 可以设置一次font-size
setRemUnit()

// 当屏幕尺寸发生变化时, 实时来修改html的font-size
window.addEventListener("resize", setRemUnit)
```

- 使用相对单位vw和vh，根据设备的宽度和高度来动态计算（优先）

优势：不用修改html的fontSize，防止混乱，也不用给body设置fontSize防止继承

#### 9. 大屏适配方案

动态获取设备宽高，计算缩放比例，直接对body应用transform进行缩放

```javascript

// 设计稿:  1920 * 1080
// 目标适配:  1920 * 1080   3840 * 2160 ( 2 * 2 ) ;  7680 * 2160( 4 * 2)

// 1.设计稿的尺寸
let targetX = 1920
let targetY = 1080
let targetRatio = 16 / 9 // 宽高比率

// 2.拿到当前设备(浏览器)的宽度
let currentX = document.documentElement.clientWidth || document.body.clientWidth
let currentY = document.documentElement.clientHeight || document.body.clientHeight
//  1920 * 1080  -> 3840 * 2160

// 3.计算缩放比例
let scaleRatio = currentX / targetX; // 参照宽度进行缩放 ( 默认情况 )
let currentRatio = currentX / currentY // 宽高比率

// 超宽屏
if (currentRatio > targetRatio) {
  scaleRatio = currentY / targetY // 参照高度进行缩放 并让其居中显示
  document.body.style = `width:${targetX}px;height:${targetY}px;transform: scale(${scaleRatio}) translateX(-50%); left: 50%`
} else {
  // 4.开始缩放网页
  document.body.style = `width:${targetX}px; height:${targetY}px; transform: scale(${scaleRatio})`
}
```


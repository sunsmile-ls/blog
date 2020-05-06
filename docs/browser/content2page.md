从输入 URL 到页面展示，这中间发生了什么？

!> 由[浏览器工作原理](https://time.geekbang.org/column/article/117637)总结而来

# 渲染流程

> 流水线可分为如下几个子阶段：构建 DOM 树、样式计算、布局阶段、分层、绘制、分块、光栅化和合成。

## 构建 DOM 树

- **浏览器无法直接理解和使用 HTML，所以需要将 HTML 转换为浏览器能够理解的结构——DOM 树。**

DOM 树的构建过程
![dom树的构建过程](./images/125849ec56a3ea98d4b476c66c754f79.png)

在 chrome 控制台中可以通过 `document` 查看 dom 树

## 样式计算

!> 此阶段遵守 css 的继承和层叠两个规则，计算出 DOM 节点中每个元素的样式，保存在 ComputedStyle 的结构内

1. **把 CSS 转换为浏览器能够理解的结构**

> 将上面三种形式引入的 css 转换为浏览器容易理解的结构， 可以通过`document.styleSheets`查看

CSS 样式来源主要有三种：

- 通过 link 引用的外部 CSS 文件
- `<style>`标记内的 CSS
- 元素的 style 属性内嵌的 CSS

2. **转换样式表中的属性值，使其标准化**

将属性值进行标准化处理

![属性值进行标准化处理](./images/1252c6d3c1a51714606daa6bdad3a560.png)

3. **计算出 DOM 树中每个节点的具体样式**

如果相同样式，子元素中没有声明，则继承父元素中的样式，例如下面子元素 `p` 标签没有`font-size`属性，则继承父元素中的 `font-size` 的属性值

```html
<style>
  body {
    font-size: 20px;
  }
  p {
    color: red;
  }
</style>
```

## 布局阶段

> 计算出 DOM 树中可见元素的几何位置，我们把这个计算过程叫做布局。

Chrome 在布局阶段需要完成两个任务：创建布局树和布局计算。

1. **创建布局树**

> 构建一棵只包含可见元素布局树 [去掉 head，样式为`display:none` 等不可见元素]

![布局树构造过程示意图](./images/8e48b77dd48bdc509958e73b9935710e.png?300*400)

2. **布局计算**

- 计算布局树节点的坐标位置,在执行布局操作的时候，会把布局运算结果重新写回布局树中。

## 分层

> 渲染引擎需要为特定的节点生成专用的图层，并生成一棵对应的图层树（LayerTree）

具备分层条件的，分为单独的层级：

1. **拥有层叠上下文属性的会被提升为单独的层级**

包含： `position:fixed`, `z-index:2`, `filter:blue(5)`, `opacity:0.5`。具体可以[参考文章](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Understanding_z_index/The_stacking_context)

2. **需要剪裁（clip）的地方也会被创建为图层**

利用`overflow:auto`会被提升一个新的图层

## 图层绘制

> 为每个图层生成绘制列表，并且提交给合成线程

## 栅格化（raster）操作

合成线程会将图层划分为图块，通常图块为 `256x256` 或者 `512x512`。 合成线程会按照视口附近的图块来优先生成位图，实际生成位图的操作是由栅格化来执行的。所谓栅格化，是指将图块转换为位图

![](./images/a8d954cd8e4722ee03d14afaa14c3987.png)

> 栅格化过程都会使用 GPU 来加速生成，使用 GPU 生成位图的过程叫快速栅格化，或者 GPU 栅格化，生成的位图被保存在 GPU 内存中

## 合成和显示

一旦所有图块都被光栅化，合成线程就会生成一个绘制图块的命令——“DrawQuad”，然后将该命令提交给浏览器进程。

浏览器进程里面有一个叫 viz 的组件，用来接收合成线程发过来的 DrawQuad 命令，然后根据 DrawQuad 命令，将其页面内容绘制到内存中，最后再将内存显示在屏幕上。

## 浏览器相关概念的总结

- **更新了元素的几何属性（重排）**

修改了元素的几何属性，会触发重排，比如修改了元素的宽高

![重排](./images/b3ed565230fe4f5c1886304a8ff754e5.png)

- **更新元素的绘制属性（重绘）**

修改了背景颜色，不修改几何属性的，触发重绘机制

![重绘](./images/3c1b7310648cccbf6aa4a42ad0202b03.png)

- **直接合成阶段**

CSS 的 transform 来实现动画效果

![合成](./images/024bf6c83b8146d267f476555d953a2c.png)

- **重绘和回流和 Eventloop 的关系**

1. 当 Eventloop 执行完 Microtasks 后，会判断 `document` 是否需要更新，因为浏览器是 60Hz 的刷新率，每 16.6ms 才会更新一次。
2. 然后判断是否有 `resize` 或者 `scroll` 事件，有的话会去触发事件，所以 `resize` 和 `scroll` 事件也是至少 16ms 才会触发一次，并且自带节流功能。
3. 判断是否触发了 `media query`
4. `更新动画`并且发送事件
5. 判断是否有全屏操作事件
6. 执行 `requestAnimationFrame` 回调
7. 执行 `IntersectionObserver` 回调，该方法用于判断元素是否可见，可以用于懒加载上，但是兼容性不好
8. 更新界面
9. 如果在一帧中有空闲时间，就会去执行 `requestIdleCallback` 回调

## 减少重绘和重排的方法

- 使用 `translate` 替代 `top`
- 使用 `visibility` 替换 `display: none` ，因为前者只会引起重绘，后者会引发回流（改变了布局）
- 把 `DOM` 离线后修改，比如：先把 `DOM` 给 `display:none`(有一次 `Reflow`)，然后你修改`100`次，然后再把它显示出来
- 不要把 `DOM`结点的属性值放在一个循环里当成循环里的变量

```js
for (let i = 0; i < 1000; i++) {
  // 获取 offsetTop 会导致回流，因为需要去获取正确的值
  console.log(document.querySelector('.test').style.offsetTop)
}
```

- 使用 css 动画代替 js 动画
- 不要使用 `table` 布局，可能很小的一个小改动会造成整个 `table` 的重新布局 动画实现的速度的选择，动画速度越快，回流次数越多，也可以选择使用 `requestAnimationFrame`
- `CSS`选择符从右往左匹配查找，避免 `DOM` 深度过深
- 将频繁运行的动画变为图层，图层能够阻止该节点回流影响别的元素。比如对于 `video` 标签，浏览器会自动将该节点变为图层。

**设置节点为图层的方式有很多，我们可以通过以下几个常用属性可以生成新图层**

`will-change`

`video`、`iframe` 标签

## 指令式绘图系统：如何用 Canvas 绘制层次关系图

```javascript
<body>
  <canvas width="512" height="512"></canvas>
</body>
```

> `Canvas` 元素上的 `width` 和 `height` 属性不等同于 `Canvas` 元素的 `CSS` 样式的属性。 `CSS` 属性中的宽高影响 Canvas 在页面上呈现的大小，而 `HTML` 属性中的宽高则决定了 `Canvas` 的坐标系

`Canvas` X 轴为水平向左的， Y 轴为垂直向下的。

## Canvas 绘制几何图形

1. 获取 Canvas 上下文

```javascript
// 1. 获取到 canvas 标签
const canvas = document.querySelector('canvas')
// 2. 获取上下文对象
const context = canvas.getContext('2d')
```

2. 设置绘图状态，比如填充颜色 fillStyle，平移变换 translate 等等；
3. 调用 beginPath 指令开始绘制图形；
4. 调用绘图指令，比如 rect，表示绘制矩形；
5. 调用 fill 指令，将绘制内容真正输出到画布上

## 如何把元素移动到中心位置

1. x,y 都减去离中心的位置。例如 rect 指令的 x、y 参数，等于画布宽高的一半分别减去矩形自身宽高的一半。
2. 先给画布设置一个平移变换（Translate），然后再进行绘制

## 画布状态恢复

1. 反向平移

```javascript

// 平移
context.translate(-0.5 * rectSize[0], -0.5 * rectSize[1]);

... 执行绘制

// 恢复
context.translate(0.5 * rectSize[0], 0.5 * rectSize[1]);
```

2. 利用 Canvas 上下文还提供了 save 和 restore 方法

```javascript

context.save(); // 暂存状态
// 平移
context.translate(-0.5 * rectSize[0], -0.5 * rectSize[1]);

... 执行绘制

context.restore(); // 恢复状态
```

## cavans 的优缺点

优点： Canvas 的简单易操作和高效的渲染能力是它的优势

缺点： 不能方便地控制它内部的图形元素

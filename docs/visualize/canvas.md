##  指令式绘图系统：如何用Canvas绘制层次关系图

```javascript
<body>
  <canvas width="512" height="512"></canvas>
</body>
```
> `Canvas` 元素上的 `width` 和 `height` 属性不等同于 `Canvas` 元素的 `CSS` 样式的属性。 `CSS` 属性中的宽高影响 Canvas 在页面上呈现的大小，而 `HTML` 属性中的宽高则决定了 `Canvas` 的坐标系

`Canvas` X 轴为水平向左的， Y轴为垂直向下的。

## Canvas 绘制几何图形

1. 获取 Canvas 上下文
```javascript
// 1. 获取到 canvas 标签
const canvas = document.querySelector('canvas');
// 2. 获取上下文对象
const context = canvas.getContext('2d');
```
2. 用 `Canvas` 上下文绘制图形

`context api` 分类：

1. 设置状态的 API (设置或改变当前的绘图状态)
2. 绘制指令 API (绘制不同形状的几何图形)
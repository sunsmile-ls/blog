## SVG 需要声明

```html
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
  <circle
    cx="100"
    cy="50"
    r="40"
    stroke="black"
    stroke-width="2"
    fill="orange"
  />
</svg>
```

> 解释：

`svg` 元素是 `SVG` 的根元素，属性 `xmlns` 是 `xml` 的名字空间。那第一行代码就表示，svg 元素的 xmlns 属性值是 "http://www.w3.org/2000/svg" ，浏览器根据这个属性值就能够识别出这是一段 SVG 的内容了

SVG 元素要使用 `document.createElementNS` 方法来创建。`第一个参数是名字空间`，对应 SVG 名字空间 http://www.w3.org/2000/svg。 `第二个参数是要创建的元素标签名`，因为要绘制圆型，所以我们还是创建 circle 元素。

用 SVG 绘制可视化图形与用 Canvas 绘制有明显区别，SVG 通过创建标签来表示图形元素，然后将图形元素添加到 DOM 树中，交给 DOM 完成渲染。

使用 DOM 树渲染可以让图形元素的用户交互实现起来非常简单，因为我们可以直接对图形元素注册事件。但是这也带来问题，如果图形复杂，那么 SVG 的图形元素会非常多，这会导致 DOM 树渲染成为性能瓶颈。

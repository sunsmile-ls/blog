# 动手实现 React (二)

> 文章基于React 16.8

我们将从头开始一步一步重写React。遵循真实的React代码中的架构，但`没有所有的优化和非必要的功能`。

> 目录

1. [`createElement`](/react/sunsmile-react1#_1createelement)
2. [`render`](/react/sunsmile-react1#_2render)
3. [`Concurrent Mode`](/react/sunsmile-react2)
4. Fibers
5. Render 和 Commit 阶段
6. Reconciliation
7. Function Components
8. Hooks

### 回顾

```js
function render(element, container) {
  const dom = document.createElement(element.type) 

  element.props.children.forEach(child =>
    render(child, dom)
  )

  container.appendChild(dom)
}
```
回顾上一篇文章中的`render`函数，我们发现直到渲染完后，才会停止。如果`element`很大，则它可能会阻塞主线程太长时间。而此时如果浏览器需要执行高优先级的操作（例如处理用户输入或保持动画流畅），则必须等到渲染完成才会执行。

众所周知，主流的浏览器的刷新频率为`60HZ`,即每`16.6ms（1000ms/60HZ）`浏览器就会刷新一次。我们知道JS可以`操作DOM`，`GUI渲染线程`与`JS线程`是互斥的。所以`JS脚本执行`和`浏览器布局`、`绘制`不能同时执行。

为了保证用户体验，在`16.6ms`之内，浏览器需要执行以下流程

![浏览器执行流程](../_media/browser-logic.png)

> 为了解决 `render 阶段`花费大量的时间，造成页面卡顿。`react 16` 引入了 `Concurrent Mode`, 可帮助应用保持响应，并根据用户的设备性能和网速进行适当的调整。

我们先介绍一个浏览器api `requestIdleCallback` -- 在浏览器一帧的剩余空闲时间内执行优先度相对较低的任务。

![浏览器执行流程](../_media/browser-logic-1.png)

#### 插播
```javascript
var lowTasks = 10000

requestIdleCallback(unImportWork)

function unImportWork(deadline) {
  while (deadline.timeRemaining() && lowTasks > 0) {
    console.log(`执行了${10000 - lowTasks + 1}个任务`)
    lowTasks--
  }

  if (lowTasks > 0) { // 在未来的帧中继续执行
    requestIdleCallback(unImportWork)
  }
}
```
`deadline` 有两个参数

- timeRemaining(): 当前帧还剩下多少时间
- didTimeout: 是否超时

另外 `requestIdleCallback` 后如果跟上第二个参数 `{timeout: ...}` 则会强制浏览器在当前帧执行完后执行。

> `react`并没有利用这个属性完成 `Concurrent Mode`, 而是自己实现了 `scheduler`。原因有几个：1.兼容性；2. `requestIdleCallback` 的 `FPS` 只有 `20`; 3. 没有优先级调度概念。 但是对于此用例，它在概念上是相同的。

### 步入正题

经过上面的分析，我们可以得出以下结论 -- **`我们将工作 render 为多个单元，在完成每个单元后，如果需要执行其他任何操作，我们将让浏览器中断渲染`**
```javascript
let nextUnitOfWork = null
​
function workLoop(deadline) {
  let shouldYield = false
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(
      nextUnitOfWork
    )
    shouldYield = deadline.timeRemaining() < 1
  }
  requestIdleCallback(workLoop)
}
​
requestIdleCallback(workLoop)
​
function performUnitOfWork(nextUnitOfWork) {
  // TODO
}
```
通过上面代码，我们把下一个需要处理的单元存储在`nextUnitOfWork`中，在浏览器空闲的时候我们再继续处理，这样就实现`Concurrent Mode`

我们又发现了新的问题，`nextUnitOfWork` 的结构是什么呢？我们如何创建它？请看下一小节 -- `Fibers`

未完待续...... 防止走失，请关注作者。




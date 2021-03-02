# 动手实现React

> 文章基于React 16.8

我们将从头开始一步一步重写React。遵循真实的React代码中的架构，但`没有所有的优化和非必要的功能`。

> 目录

1. [`createElement`](#_1createelement)
2. [`render`](#_2render)
3. Concurrent Mode
4. Fibers
5. Render 和 Commit 阶段
6. Reconciliation
7. Function Components
8. Hooks

接下来我们就一步一步完成以上功能：

### 0.回顾

```javascript
const element = <h1 title="foo">Hello</h1>
const container = document.getElementById("root")
ReactDOM.render(element, container)
```
当我们执行`create-react-app xxx`在`index.js`文件，可以看到相似的代码。我们在第一行运用`JSX`声明了一个元素。

通过`Babel`编译，即可正常运行。

```javascript
const element = React.createElement(
  "h1",
  { title: "foo" },
  "Hello"
)
```
`React.createElement`根据其参数，除了一些验证之外，就是创建一个对象。所以我们就可以把上面代码替换为输出。
```javascript
const element = { // 包含 type 和 props
  type: "h1", // 表示创建一个HTML元素，也可以是 function
  props: { // 包含了h1标签上面的属性 和 children
    title: "foo",
    children: "Hello", // 通常是一个包含更多元素的数组
  },
}
```
现在唯一的谜题就是`ReactDOM.render(element, container)` 方法了。
```javascript
// 根据 type 创建 h1, 添加 属性，这里只有 title 属性
const node = document.createElement(element.type)
node["title"] = element.props.title

// 创建 文本节点
const text = document.createTextNode("")
text["nodeValue"] = element.props.children

// 添加 text -> h1 -> container
node.appendChild(text)
container.appendChild(node)
```
至此，我们已经实现了相同的功能。

虽然上面的代码已经实现了相同的功能，但是我相信你一定会吐槽，别急接下来我们就进入正题。

### 1.createElement

接下来我们创建另一个应用，我们用自己的代码替代`react`。
```javascript
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
)
const container = document.getElementById("root")
ReactDOM.render(element, container)
```
将`element`经过[`babel`](https://www.babeljs.cn/repl#?browsers=defaults%2C%20not%20ie%2011%2C%20not%20ie_mob%2011&build=&builtIns=false&spec=false&loose=false&code_lz=KYG2FtgOwFwAgLxwBQCg5wDwBMCWA3OXbBAIgDMB7S0gPnQywENaAjJgJ0wHoWGNMrON3oDuefPQCUQA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=env%2Creact%2Cstage-2&prettier=false&targets=&version=7.13.8&externalPlugins=)转换之后可以得到
```javascript
const element = React.createElement(
  "div",
  { id: "foo" },
  React.createElement("a", null, "bar"),
  React.createElement("b")
)
```
我们用扩展运算符处理 `props` 和剩余参数处理 `children` ，实现`createElement`
```javascript
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map(child =>
        typeof child === "object"
          ? child
          : createTextElement(child)
      ),
    },
  }
}
function createTextElement(text) {
  return {
    type: "TEXT_ELEMENT",
    props: {
      nodeValue: text,
      children: [],
    },
  }
}
```
例如：`createElement("div", null, a, b)` returns:
```javascript
{
  "type": "div",
  "props": { "children": [a, b] }
}
```
`children` 数组可以包含原始值，例如字符串或数字。我们原始值包装成对象，并为它们创建一个特殊的类型：`TEXT_ELEMENT`
> React不会包装原始值，并添加空数组的`children`，但我们为了使代码更加简单添加了此操作。

接下来我们为库起一个名字`SunsmileReact`，经过修改可以得到

```javascript
const SunsmileReact = {
  createElement,
}

const element = SunsmileReact.createElement(
  "div",
  { id: "foo" },
  SunsmileReact.createElement("a", null, "bar"),
  SunsmileReact.createElement("b")
)
```
此时我们需要告诉babel 转换为 `SunsmileReact.createElement`

```js
/** @jsx SunsmileReact.createElement */
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
)
```
经过一番折腾，我们已经实现了 `createElement`。如果还需要深入了解，请看[github createElement](https://github.com/sunsmile-ls/sunsmileReact/tree/0.0.1)。

### 2.render

经过前面的`createElement`，我们现在可以得如下类似的内容：
```javascript
{
  "type": "div",
  "props": { "title": "title", "children": [a, b] }
}
```
接下来我们开始新的征程，编写`ReactDOM.render`函数，目前我们只考虑新增，更新和删除我们之后再考虑。

基础架子已经搭建
```javascript
function render(element, container) {
  // TODO create dom nodes
}
const SunsmileReact = {
  createElement,
  render,
}

/** @jsx SunsmileReact.createElement */
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
)
const container = document.getElementById("root")
SunsmileReact.render(element, container)
```
我们为`SunsmileReact`添加了一个 `render` 属性。并且最后执行`SunsmileReact.render(element, container)`

```javascript
function render(element, container) {
  const dom = document.createElement(element.type) 

  element.props.children.forEach(child =>
    render(child, dom)
  )

  container.appendChild(dom)
}
```
创建了 `dom` 节点，并把node节点添加到容器中。并且递归每一个子孩子做同样的事情。

上面的代码我们忽略了文本节点和给节点添加属性的操作

```javascript
function render(element, container) {
  // 处理文本节点
  const dom =
    element.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(element.type)
  // 添加属性    
  const isProperty = key => key !== "children"
  Object.keys(element.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = element.props[name]
    })

  element.props.children.forEach(child =>
    render(child, dom)
  )

  container.appendChild(dom)
}
```
到这儿，我们已经实现了`render`函数。

了解更多请看 [github render](https://github.com/sunsmile-ls/sunsmileReact/tree/0.0.2)

未完待续......

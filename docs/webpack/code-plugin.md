#### 如何⾃⼰编写⼀个 Plugins

Plugin: 开始打包，在某个时刻，帮助我们处理⼀些什么事情的机制

plugin 要⽐ loader 稍微复杂⼀些，在 webpack 的源码中，⽤ plugin 的机制还是占有⾮常⼤的场景，可以 说 plugin 是 webpack 的灵魂

- 设计模式

  - 事件驱动
  - 发布订阅

  **plugin 是⼀个类，⾥⾯包含⼀个 apply 函数，接受⼀个参数，compiler**

> 任务：webpack 在文件

```js
class CopyrightWebpackPlugin {
  //compiler：webpack实例,包含配置信息
  apply(compiler) {
    compiler.hooks.compile.tap('CopyrightWebpackPlugin', compilation => {
      console.log('开始！')
    })
    compiler.hooks.emit.tapAsync(
      // 文件放到输出目录之前执行
      'CopyrightWebpackPlugin',
      (compilation, cb) => {
        compilation.assets['test.txt'] = {
          source: () => {
            return 'hello txt'
          },
          size: () => {
            return 1024
          }
        }
        cb()
      }
    )
  }
}
module.exports = CopyrightWebpackPlugin
```

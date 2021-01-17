#### 文件的预加载

在网络空闲的时候进行加载文件

```js
import(/* webpackPrefetch: true */ 'LoginModal');
```

#### 当前构建时的⽇志显示

统计信息 stats

![](.\images\20200411204435960.png)

* 如何优化命令⾏的构建⽇志

使⽤ friendly-errors-webpack-plugin 

* success: 构建成功的⽇志提示 
* warning: 构建警告的⽇志提示 
* error: 构建报错的⽇志提示 

stats 设置成 errors-only

```js
module.exports = {
    entry: {
        app: './src/app.js',
        search: './src/search.js'
    },
    output: {
        filename: '[name][chunkhash:8].js',
        path: __dirname + '/dist'
    },
    plugins: [
        new FriendlyErrorsWebpackPlugin()
    ],
    stats: 'errors-only'
};
```

* 如何判断构建是否成功？

  ​    构建异常和中断处理 webpack4 之前的版本构建失败不会抛出错误码 (error code) 

  Node.js 中的 process.exit 规范  

  *  0 表示成功完成，回调函数中，err 为 null 
  * ⾮ 0 表示执⾏失败，回调函数中，err 不为 null，err.code 就是传给 exit 的数字

* 如何主动捕获并处理构建错误？

  compiler 在每次构建结束后会触发 done 这 个 hook

  process.exit 主动处理构建报错

```js
plugins: [
    function() {
        this.hooks.done.tap('done', (stats) => {
            if (stats.compilation.errors &&
                stats.compilation.errors.length &&
                process.argv.indexOf('--watch') == -1)
            {
                console.log('build error');
                process.exit(1);
            }
        })
    }
]
```


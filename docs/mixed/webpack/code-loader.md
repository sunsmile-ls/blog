## 编写 loader 的必要条件

1.  Loader 就是⼀个函数，声明式函数，不能⽤箭头函数
2.  拿到源代码，作进⼀步的修饰处理, 再返回处理后的源码就可以了

> 任务：创建⼀个替换源码中字符串的 loader

- loader 的编写

```js
//index.js
console.log('hello kkb')

//replaceLoader.js
module.exports = function(source) {
  console.log(source, this, this.query)
  return source.replace('kkb', '开课吧')
}
//需要⽤声明式函数，因为要上到上下⽂的this,⽤到this的数据，该函数接受⼀个参数，是源码
```

- 在配置⽂件中使⽤ loader

```js
//需要使⽤node核⼼模块path来处理路径
const path = require('path')
module: {
    rules: [
        {
            test: /\.js$/,
            use: path.resolve(__dirname, "./loader/replaceLoader.js")
        }
    ]
},
```

- 如何给 loader 配置参数，loader 如何接受参数?
- this.query
- loader-utils

```js
//webpack.config.js
module: {
  rules: [
    {
      test: /\.js$/,
      use: [
        {
          loader: path.resolve(__dirname, './loader/replaceLoader.js'),
          options: {
            name: '开课吧'
          }
        }
      ]
    }
  ]
}

//replaceLoader.js
//const loaderUtils = require("loader-utils");//官⽅推荐处理loader,query的⼯具
module.exports = function(source) {
  //this.query 通过this.query来接受配置⽂件传递进来的参数
  //return source.replace("kkb", this.query.name);
  const options = loaderUtils.getOptions(this)
  return source.replace('kkb', options.name)
}
```

- **this.callback** :如何返回多个信息，不⽌是处理好的源码呢，可以使⽤ this.callback 来处理

```js
//replaceLoader.js
const loaderUtils = require('loader-utils') //官⽅推荐处理loader,query的⼯具
module.exports = function(source) {
  const options = loaderUtils.getOptions(this)
  const result = source.replace('kkb', options.name)
  this.callback(null, result)
}
//this.callback(
//	err: Error | null,
//  content: string | Buffer,
//	sourceMap?: SourceMap,
//	meta?: any
//);
```

- **this.async**：如果 loader ⾥⾯有异步的事情要怎么处理呢

```js
const loaderUtils = require('loader-utils')
module.exports = function(source) {
  const options = loaderUtils.getOptions(this)
  //定义⼀个异步处理，告诉webpack,这个loader⾥有异步事件,在⾥⾯调⽤下这个异步
  //callback 就是 this.callback 注意参数的使⽤
  const callback = this.async()
  setTimeout(() => {
    const result = source.replace('kkb', options.name)
    callback(null, result)
  }, 3000)
}
```

- 多个 loader 的使⽤

```js
//replaceLoader.js
module.exports = function(source) {
    return source.replace("开课吧", "word");
};
//replaceLoaderAsync.js
const loaderUtils = require("loader-utils");
module.exports = function(source) {
    const options = loaderUtils.getOptions(this);
    //定义⼀个异步处理，告诉webpack,这个loader⾥有异步事件,在⾥⾯调⽤下这个异步
    const callback = this.async();
    setTimeout(() => {
        const result = source.replace("kkb", options.name);
        callback(null, result);
    }, 3000);
};
//webpack.config.js
module: {
    rules: [
        {
            test: /\.js$/,
            use: [
                path.resolve(__dirname, "./loader/replaceLoader.js"),
                {
                    loader: path.resolve(__dirname, "./loader/replaceLoaderAsync.js"),
                    options: {
                        name: "开课吧"
                    }
                }
            ]
            // use: [path.resolve(__dirname, "./loader/replaceLoader.js")]
        }
    ]
},
```

- 处理 loader 的路径问题

```js
resolveLoader: {
    modules: ["node_modules", "./loader"]
},
module: {
    rules: [
        {
            test: /\.js$/,
            use: [
                "replaceLoader",
                {
                    loader: "replaceLoaderAsync",
                    options: {
                        name: "开课吧"
                    }
                }
            ]
            // use: [path.resolve(__dirname, "./loader/replaceLoader.js")]
        }
    ]
},
```

## 参考：loader API

https://webpack.js.org/api/loaders

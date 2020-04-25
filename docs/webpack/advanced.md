## 文件指纹

**Hash**: 和整个项目构建相关，只有项目文件有修改，整个项目构建的 hash 值就会改变

**Chunkhash**: 和 webpack 打包的 chunk 相关，不同的 entry 会生成不同的 chunkhash【js 使用】

**Contenthash**: 根据文件的内容定义 hash，文件内容不变，则 contenthash 不变【css 使用】

> 例子

- JS 的⽂件指纹设置

```js
module.exports = {
  output: {
    filename: '[name][chunkhash:8].js',
    path: __dirname + '/dist'
  }
}
```

- CSS 的⽂件指纹设置

设置 MiniCssExtractPlugin 的 filename， 使⽤ [contenthash]

```js
module.exports = {
    plugins: [
        new MiniCssExtractPlugin({
            filename: `[name][contenthash:8].css`
        });
    ]
};
```

- 图⽚的⽂件指纹设置

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|svg|jpg|gif)$/,
        use: [
          {
            loader: 'file-loader',
            options: {
              name: 'img/[name][hash:8].[ext]'
            }
          }
        ]
      }
    ]
  }
}
```

## 文件压缩

- JS ⽂件的压缩

内置了 uglifyjs-webpack-plugin

- CSS ⽂件的压缩

```js
// 以前版本css-loader 可以添加minify参数可以压缩css,之后版本中去掉这个参数
module.exports = {
  plugins: [
    new OptimizeCSSAssetsPlugin({
      assetNameRegExp: /\.css$/g,
      cssProcessor: require('cssnano')
    })
  ]
}
```

- html ⽂件的压缩

```js
module.exports = {
  plugins: [
    new HtmlWebpackPlugin({
      template: path.join(__dirname, 'src/search.html'),
      filename: 'search.html',
      chunks: ['search'],
      inject: true,
      minify: {
        html5: true,
        collapseWhitespace: true,
        preserveLineBreaks: false,
        minifyCSS: true,
        minifyJS: true,
        removeComments: false
      }
    })
  ]
}
```

## ⾃动清理构建⽬录

默认会删除 output 指定的输出⽬录

```js
plugins: [new CleanWebpackPlugin()]
```

## PostCSS 插件 autoprefixer ⾃动补⻬ CSS3 前缀

根据 Can I Use 规则（ https://caniuse.com/ ）

```js
rules: [
  {
    test: /\.less$/,
    use: [
      'style-loader',
      'css-loader',
      'less-loader',
      {
        loader: 'postcss-loader',
        options: {
          plugins: () => [
            require('autoprefixer')({
              browsers: ['last 2 version', '> 1%', 'iOS 7']
            })
          ]
        }
      }
    ]
  }
]
```

## 移动端 CSS px ⾃动转换成 rem

1. px2rem-loader

2. ⽤⼿淘的 lib-flexible 库

```js
rules: [
  {
    test: /\.less$/,
    use: [
      'style-loader',
      'css-loader',
      'less-loader',
      {
        loader: 'px2rem-loader',
        options: {
          remUnit: 75, // 一个rem代表多少个像素
          remPrecision: 8 // px 转换为rem小数点后面的位数
        }
      }
    ]
  }
]
```

## 资源内联的

- 意义

1. ⻚⾯框架的初始化脚本
2. 上报相关打点
3. css 内联避免⻚⾯闪动

HTML 和 JS 内联

- raw-loader 内联 html 和 JS

```html
<script>
  ${require(' raw-loader!babel-loader!. /meta.html')}
</script>
```

- CSS 内联

```js
rules: [
  {
    test: /\.scss$/,
    use: [
      {
        loader: 'style-loader',
        options: {
          insertAt: 'top', // 样式插入到 <head>
          singleton: true //将所有的style标签合并成一个
        }
      },
      'css-loader',
      'sass-loader'
    ]
  }
]
```

## 多⻚⾯打包通⽤⽅案(MPA)

```js
const glob = require('glob')
const path = require('path')
const webpack = require('webpack')
const HtmlWebpackPlugin = require('html-webpack-plugin')

const setMPA = () => {
  const entry = {}
  const htmlWebpackPlugins = []
  const entryFiles = glob.sync(path.join(__dirname, './src/*/index.js'))

  Object.keys(entryFiles).map(index => {
    const entryFile = entryFiles[index]
    // '/Users/cpselvis/my-project/src/index/index.js'
    const match = entryFile.match(/src\/(.*)\/index\.js/)
    const pageName = match && match[1]
    console.log(entryFile, path.join(__dirname, `src/${pageName}/index.html`))
    entry[pageName] = entryFile
    htmlWebpackPlugins.push(
      new HtmlWebpackPlugin({
        inlineSource: '.css$',
        template: path.join(__dirname, `src/${pageName}/index.html`),
        filename: `${pageName}.html`,
        chunks: ['vendors', pageName],
        inject: true,
        minify: {
          html5: true,
          collapseWhitespace: true,
          preserveLineBreaks: false,
          minifyCSS: true,
          minifyJS: true,
          removeComments: false
        }
      })
    )
  })

  return {
    entry,
    htmlWebpackPlugins
  }
}
```

## sourceMap

源代码与打包后的代码的映射关系

在 dev 模式中，默认开启，关闭的话 可以在配置⽂件⾥

```
devtool:"none"
```

devtool 的介绍：https://webpack.js.org/configuration/devtool#devtool

eval:速度最快 ,使用 eval 包裹模块代码

source-map:会生成.map 文件

cheap:较快，不⽤管列的报错 和 loader 的 source map

module：第三⽅模块

inline: 将.map 作为 uri 嵌入，不单独生成.map 文件

开发环境推荐：

```js
devtool: 'cheap-module-eval-source-map'
// 线上环境可以不开启：如果要看到⼀些错误信息，推荐；
devtool: 'cheap-module-source-map'
```

## 基础库分离

将 react、react-dom 基础 包通过 cdn 引⼊，不打⼊ bundle 中

使⽤ html-webpackexternals-plugin

```js
new HtmlWebpackExternalsPlugin({
    externals: [
        {
            module: 'react',
            entry: 'https://11.url.cn/now/lib/16.2.0/react.min.js',
            global: 'React',
        },
        {
            module: 'react-dom',
            entry: 'https://11.url.cn/now/lib/16.2.0/react-dom.min.js',
            global: 'ReactDOM',
        },
    ]
}),
```

## SplitChunksPlugin

- 进⾏公共脚本分离

chunks 参数说明：

1. **async** 异步引⼊的库进⾏分离(默认)

2. **initial** 同步引⼊的库进⾏分离

3. **all** 所有引⼊的库进⾏分离(推荐)

```js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'async',
      minSize: 30000, // 公共包最小的大小，[单位是字节]
      maxSize: 0, // 公共包最大的大小，[单位是字节]
      minChunks: 1, // 最小的引用次数
      maxAsyncRequests: 5, // 同时请求js/css的数量
      maxInitialRequests: 3,
      automaticNameDelimiter: '~',
      name: true,
      cacheGroups: {
        // 只有满足上面的条件才会进入下面的判断
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10 // 数字越大，优先级越高
        }
      }
    }
  }
}
```

- 分离基础包

test: 匹配出需要分离的包

```js
module.exports = {
  optimization: {
    splitChunks: {
      cacheGroups: {
        commons: {
          test: /(react|react-dom)/,
          name: 'vendors',
          chunks: 'all'
        }
      }
    }
  }
}
```

- 分离⻚⾯公共⽂件

minChunks: 设置最⼩引⽤次数为 2 次

minuSize: 分离的包体积的⼤⼩

```js
module.exports = {
    optimization: {
        splitChunks: {
            minSize: 0,
            cacheGroups: {
                commons: {
                    name: 'commons',
                    chunks: 'all',
                    minChunks: 2
                }
            }
        }
    }
}
};
```

## tree shaking(摇树优化)

**没⽤到的⽅法会在 uglify 阶段被擦除掉**

**使⽤**：webpack 默认⽀持，mode 为 production 默认支持，需要把 mode:none 不支持

**要求**：必须是 ES6 的语法，CJS 的⽅式不⽀持

- DCE (Dead code elimination)

  代码不会被执⾏，不可到达

  代码只会影响死变量（只写不读）

  代码执⾏的结果不会被⽤到

- 代码擦除： uglify 阶段删除⽆⽤代码

## scope hoisting 原理

将所有模块的代码按照引⽤顺序放在⼀个函数作⽤域⾥，然后适当的重命名⼀ 些变量以防⽌变量名冲突

- **即把使用了一次的代码内联到文件里面，使用多次的任然是一个闭包**

- mode: production 默认开启

- **必须是 ES6 语法，CJS 不⽀持**

```js
// webpack 3需要手动开启
new webpack.optimize.ModuleConcatenationPlugin()
```

## 动态 import

- 安装 babel 插件 ES6：

npm install @babel/plugin-syntax-dynamic-import --save-dev

- ES6：动态 import（⽬前还没有原⽣⽀持，需要 babel 转换）【.babelrc 文件中添加】

```js
{ "plugins": ["@babel/plugin-syntax-dynamic-import"], ... }
```

## ESLint 如何执⾏落地？

- 和 CI/CD 系统集成

安装 husky

```
npm install husky --save-dev
```

增加 npm script，通过 lint-staged 增量检查修改的⽂件

```json
"scripts": {
    "precommit": "lint-staged"
},
"lint-staged": {
    "linters": {
        "*.{js,scss}": ["eslint --fix", "git add"]
    }
},
```

- 和 webpack 集成

使⽤ eslint-loader，构建时检查 JS 规范

## 文件的预加载

在网络空闲的时候进行加载文件

```js
import(/* webpackPrefetch: true */ 'LoginModal')
```

## Hot Module Replacement (HMR:热模块替换)

启动 hmr

```js
devServer: {
    contentBase: "./dist",
    open: true,
    hot:true,
    //即便HMR不生效，浏览器也不自动刷新，就开启hotOnly
    hotOnly:true
},
```

配置文件头部引入 webpack

```js
const webpack = require('webpack')
```

在插件配置处添加：

```js
plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
        template: "src/index.html"
    }),
    new webpack.HotModuleReplacementPlugin()
],
```

- 开启热更新之后，css 会自动生效
- 开启热更新之后，js 不会自动生效

```js
if (module.hot) {
  module.hot.accept('文件地址', function() {
    // 文件更新之后相应的处理
  })
}
```

## Babel 处理 ES6

- @babel/polyfill

以全局变量的方式注入进来的。windows.Promise，会造成全局对象的污染。

```
npm install --save @babel/polyfill
```

按需加载

```js
options: {
  presets: [
    [
      '@babel/preset-env',
      {
        targets: {
          edge: '17',
          firefox: '60',
          chrome: '67',
          safari: '11.1'
        },
        useBuiltIns: 'usage', //按需注入
        corejs: 2
      }
    ]
  ]
}
```

当我们开发的是组件库，工具库这些场景的时候，polyfill 就不适合了，因为 polyfill 是注入到全局变量，window 下 的，会污染全局环境，所以推荐闭包方式：@babel/plugin-transform-runtime

- @babel/plugin-transform-runtime

它不会造成全局污染

```
npm install --save-dev @babel/plugin-transform-runtime
npm install --save @babel/runtime
```

修改.babelrc 文件

```js
"plugins": [
    [
        "@babel/plugin-transform-runtime",
        {
            "absoluteRuntime": false,
            "corejs": 2,
            "helpers": true,
            "regenerator": true,
            "useESModules": false
        }
    ]
]
```

babel-transform-runtime，不会造成全局污染，因此也会不会对类似 Array.prototype.includes() 进行 polyfill。

## tree Shaking

作用：去除没用的代码

webpack2.x 开始支持 tree shaking 概念，顾名思义，"摇树"，**只支持 ES module 的引入方式**

```json
optimization: {
	usedExports: true
}
```

```
//package.json
"sideEffects":false
// 正常对所有模块进行tree shaking 或者 "sideEffects":['*.css','@babel/polyfill']
```

## 代码分割 code Splitting

```json
optimization: {
    splitChunks: {
        chunks: 'async',//对同步，异步，所有的模块有效【默认为异步】
        minSize: 30000,//当模块大于30kb
        maxSize: 0,//如果大于这个数，对模块进行二次分割时使用，不推荐使用,
        minChunks: 1,//打包生成的chunk文件最少有几个chunk引用了这个模块
        maxAsyncRequests: 5,//模块请求5次
        maxInitialRequests: 3,//入口文件同步请求3次
        automaticNameDelimiter: '~',
        name: true,
        cacheGroups: { //当文件符合上面的条件，就会进入这个判断
            vendors: {
                //文件在node_modules中，则创建文件名为vendors
                test: /[\\/]node_modules[\\/]/,
                priority: -10//优先级 数字越大，优先级越高
            },
            default: {
                minChunks: 2,
                priority: -20,
                reuseExistingChunk: true
            }
        }
    }
```

## webpack 官方推荐的编码方式

```js
optimization:{
    //帮我们自动做代码分割
    splitChunks:{
        chunks:"async",//默认是支持异步
    }
}
```

​ 异步加载文件

```yaml
npm install --save-dev @babel/plugin-syntax-dynamic-import
```

```js
document.addEventListener('click', () => {
  // /* webpackPrefetch: true */ 在import中使用，表示网络空闲的时候加载文件
  import(/* webpackPrefetch: true */ './click.js').then(({ default: func }) => {
    //需要用到 npm install --save-dev @babel/plugin-syntax-dynamic-import
    func()
  })
})
```

```
// .babelrc
plugins:[
	"@babel/plugin-syntax-dynamic-import"
]
```

## loader

- file-loader

原理是把打包⼊⼝中识别出的资源模块，移动到输出⽬录，并且返回⼀个地址名称

所以我们什么时候⽤ file-loader 呢？

场景：就是当我们需要模块，仅仅是从源代码挪移到打包⽬录，就可以使⽤ file-loader 来处理， txt，svg，csv，excel，图⽚资源啦等等

```js
module: {
    rules: [
        {
            test: /\.(png|jpe?g|gif)$/,
            //use使⽤⼀个loader可以⽤对象，字符串，两个loader需要⽤数组
            use: {
                loader: "file-loader",
                // options额外的配置，⽐如资源名称
                options: {
                    // placeholder 占位符 [name]⽼资源模块的名称
                    // [ext]⽼资源模块的后缀
                    // https://webpack.js.org/loaders/file-loader#placeholders
                    name: "[name]_[hash].[ext]",
                    //打包后的存放位置
                    outputPath: "images/"
                }
            }
        }
    ]
},
```

- url-loader

可以处理 file-loader 所有的事情，但是遇到 jpg 格式的模块，会把该图⽚转换成 base64 格式字符 串，并打包到 js ⾥。对⼩体积的图⽚⽐较合适，⼤图⽚不合适。

## Plugins

plugin 可以在 webpack 运⾏到某个阶段的时候，帮你做⼀些事情，类似于⽣命周期的概念

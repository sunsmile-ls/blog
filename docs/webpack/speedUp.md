# webpack 构建速度和体积优化策略

## 使用 webpack 内置的 stats

- stats: 构建的统计信息

```js
// package.json 中使用 stats
// 文件build写入到stats.json文件中
"scripts": {
    "build:stats": "webpack --config webpack.prod.js --json > stats.json"
  },
```

- Node.js 中使用

```js
const webpack = require('webpack')
const config = require('./webpack.config.js')('production')

webpack(config, (err, stas) => {
  if (err) {
    return console.error(err)
  }
  if (stats.hasErrors()) {
    return console.error(stats.toString('errors-only'))
  }
  console.log(stats)
})
```

## 速度分析：使用 speed-measure-webpack-plugin

> 可以看到每个 loader 和插件执行耗时

```js
const SpeedMeasurePlugin = require('speed-measure-webpack-plugin')

const smp = new SpeedMeasurePlugin()

// 此方法包含在json对象的最外面
const webpackConfig = smp.wrap({
  entry: entry,
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name]_[chunkhash:8].js'
  },
  mode: 'production',
  plugins: [new MyPlugin(), new MyOtherPlugin()]
})
```

- 速度分析插件作用
  - 分析整个打包总耗时
  - 每个插件和 loader 的耗时情况

## webpack-bundle-analyzer 分析体积

```js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer')
  .BundleAnalyzerPlugin

module.exports = {
  plugins: [new BundleAnalyzerPlugin()]
}
```

## 使用高版本的 webpack 和 Node.js

- V8 带来的优化（for of 替代 forEach、Map 和 Set 替代 Object、includes 替代 indexOf）
- 默认使用更快的 md4 hash 算法
- webpack AST 可以直接从 loader 传递给 AST，减少解析时间
- 使用字符串方法替代正则表达式

## 多进程/多实例构建：资源并行解析可选方案

![1569588449376](.\images\1569588449376.png)

- 使用 HappyPack 解析资源

> 原理：每次 webapck 解析一个模块，HappyPack 会将它及它的依赖分配给 worker 线程中

```js
exports.plugins = [
  // plugins 中添加loader的处理
  new HappyPack({
    id: 'jsx',
    threads: 4,
    loaders: ['babel-loader']
  }),

  new HappyPack({
    id: 'styles',
    threads: 2,
    loaders: ['style-loader', 'css-loader', 'less-loader']
  })
]

exports.module.rules = [
  {
    test: /\.js$/,
    use: 'happypack/loader?id=jsx' // 根据plugins中参数进行替换
  },

  {
    test: /\.less$/,
    use: 'happypack/loader?id=styles'
  }
]
```

- 使用 thread-loader 解析资源

> 原理：每次 webpack 解析一个模块，threadloader 会将它及它的依赖分配给 worker 线程中

```js
rules: [{
	test: /.js$/,
		use: [{
			loader: 'thread-loader',
			options: {
				workers: 3
			}
		},
		'babel-loader',
	]
},
```

## 多进程/多实例：并行压缩

- 方法一：使用 parallel-uglify-plugin 插件

```js
const ParallelUglifyPlugin = require("uglifyjs-webpack-plugin");

module.exports = {
    plugins: [
        new ParallelUglifyPlugin({ // 默认会进行并行压缩
        	uglifyJS:{
        		output: {
                    beautify: false,
                    comments: false,
                },
                {
                	warnings: false,
                	drop_console: true,
                	collapse_vars: true,
                	reduce_vars: true,
            	}
            },
        })
    ]
}
```

- 方法二：uglifyjs-webpack-plugin 开启 parallel 参数

```js
const UglifyJsPlugin = require('uglifyjs-webpack-plugin')

module.exports = {
  plugins: [
    new UglifyJsPlugin({
      uglifyOptions: {
        warnings: false,
        parse: {},
        compress: {},
        mangle: true,
        output: null,
        toplevel: false,
        nameCache: null,
        ie8: false,
        keep_fnames: false
      },
      parallel: true // 多进程多实例压缩
    })
  ]
}
```

- 方法三：terser-webpack-plugin 开启 parallel 参数

```js
const TerserPlugin = require('terser-webpack-plugin')
module.exports = {
  optimization: {
    minimizer: [
      new TerserPlugin({
        parallel: 4
      })
    ]
  }
}
```

## 进一步分包：预编译资源模块

> 思路：将 react、react-dom、redux、react-redux 基础包和业务基础包打包成一个文件

- 方法：使用 DLLPlugin 进行分包，DllReferencePlugin 对 manifest.json 引用

1. **使用 DLLPlugin 进行分包**

```js
const path = require('path')
const webpack = require('webpack')

module.exports = {
  entry: {
    library: ['react', 'react-dom']
  },
  output: {
    filename: '[name]_[chunkhash].dll.js',
    path: path.join(__dirname, 'build/library'),
    library: '[name]'
  },
  plugins: [
    new webpack.DllPlugin({
      name: '[name]_[hash]',
      path: path.join(__dirname, 'build/library/[name].json')
    })
  ]
}
```

2. **使用 DllReferencePlugin 引用 manifest.json**

```
new PurgecssPlugin({
	paths: glob.sync(`${PATHS.src}/**/*`,  { nodir: true }),
})
```

## 缓存

> 目的：提升二次构建速度

- 缓存思路：

  - babel-loader 开启缓存

  ```js
  new HappyPack({
  	loaders: [ 'babel-loader?cacheDirectory=true' ]
  }),
  ```

  - terser-webpack-plugin 开启缓存

  ```js
  const TerserPlugin = require('terser-webpack-plugin')
  module.exports = {
    optimization: {
      minimizer: [
        new TerserPlugin({
          parallel: 4,
          cache: true
        })
      ]
    }
  }
  ```

  - 使用 cache-loader 或者 hard-source-webpack-plugin

  ```js
  const HardSourceWebpackPlugin = require('hard-source-webpack-plugin')

  new HardSourceWebpackPlugin()
  ```

## 缩小构建目标

> 目的：尽可能的少构建模块

- 比如 babel-loader 不解析 node_modules

```js
{
    test: /.js$/,
    loader:'happypack/loader',
    exclude: 'node_modules'
},
```

## 减少文件搜索范围

- 优化 resolve.modules 配置（减少模块搜索层级）

- 优化 resolve.mainFields 配置【指定入口文件】
- 优化 resolve.extensions 配置 【指定解析文件的后缀】
- 合理使用 alias 【指定文件位置】

```js
resolve: {
    alias: {
        'react': path.resolve(__dirname, './node_modules/react/umd/react.production.min.js'),
        'react-dom': path.resolve(__dirname, './node_modules/react-dom/umd/react-dom.production.min.js'),
      },
    extensions: ['.js'],
    mainFields: ['main']
}
```

## 无用的 CSS 如何删除掉？

- PurifyCSS: 遍历代码，识别已经用到的 CSS class
- uncss: HTML 需要通过 jsdom 加载，所有的样式通过 PostCSS 解析，通过 document.querySelector 来识别在 html 文件里面不存在的选择器

> 在 webpack 中如何使用 PurifyCSS?

1. purgecss-webpack-plugin 和 mini-css-extract-plugin 配合使用

```js
const path = require('path')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const PurgecssPlugin = require('purgecss-webpack-plugin')
const glob = require('glob')

const PATHS = {
  src: path.join(__dirname, 'src')
}

module.exports = {
  module: {
    rules: [
      {
        test: /.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name]_[contenthash:8].css'
    }),
    new PurgecssPlugin({
      paths: glob.sync(`${PATHS.src}/**/*`, { nodir: true })
    })
  ]
}
```

## 图片压缩

> 要求：基于 Node 库的 imagemin 或者 tinypng API

> 使用：配置 image-webpack-loader

```js
{
test: /.(png|jpg|gif|jpeg)$/,
    use: [
        {
            loader: 'file-loader',
            options: {
            name: '[name]_[hash:8].[ext]'
        }
	},
    {
        loader: 'image-webpack-loader',
        options: {
            mozjpeg: {
                progressive: true,
                quality: 65
            },
            // optipng.enabled: false will disable optipng
            optipng: {
                enabled: false,
            },
            pngquant: {
                quality: '65-90',
                speed: 4
            },
            gifsicle: {
                interlaced: false,
            },
            // the webp option will enable WEBP
            webp: {
                quality: 75
            }
        }
	}]
},
```

**优点**

- Imagemin 的压缩原理
  - **pngquant**: 是一款 PNG 压缩器，通过将图像转换为具有 alpha 通道（通常比 24/32 位 PNG
    文件小 60-80％）的更高效的 8 位 PNG 格式，可显著减小文件大小。
  - **pngcrush**:其主要目的是通过尝试不同的压缩级别和 PNG 过滤方法来降低 PNG IDAT 数据
    流的大小。
  - **optipng**:其设计灵感来自于 pngcrush。optipng 可将图像文件重新压缩为更小尺寸，而不
    会丢失任何信息。
  - **tinypng**:也是将 24 位 png 文件转化为更小有索引的 8 位图片，同时所有非必要的 metadata
    也会被剥离掉

#### 构建体积优化：动态 Polyfill

![1569646270835](.\images\1569646270835.png)

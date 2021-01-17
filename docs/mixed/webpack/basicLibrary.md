## webpack 打包库和组件

> webpack 除了可以⽤来打包应⽤，也可以⽤来打包 js 库

#### 需求

- 实现⼀个⼤整数加法库的打包

  - 需要打包压缩版和⾮压缩版本
  - ⽀持 AMD/CJS/ESM 模块引⼊

- 目录结构

```
|- /dist
	|- large-number.js
	|- large-number.min.js
|- webpack.config.js
|- package.json
|- index.js
|- /src
	|- index.js
```

- ⽀持的使⽤⽅式

  - ⽀持 ES module

  ```js
  import * as largeNumber from 'large-number'
  // ...
  largeNumber.add('999', '1')
  ```

  - ⽀持 CJS

  ```js
  const largeNumbers = require('large-number')
  // ...
  largeNumber.add('999', '1')
  ```

  - ⽀持 AMD

  ```js
  require(['large-number'], function (large-number) {
  // ...
  largeNumber.add('999', '1');
  });
  ```

#### package 的书写

```json
{
  "name": "large-number",
  "version": "1.0.1",
  "description": "大整数加法打包",
  "main": "index.js",
  "scripts": {
    "build": "webpack",
    "prepublish": "webpack"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "terser-webpack-plugin": "^1.3.0",
    "webpack": "^4.34.0",
    "webpack-cli": "^3.3.4"
  }
}
```

#### 指对 .min 压缩

- 通过 include 设置只压缩 min.js 结尾的⽂件

```js
module.exports = {
  mode: 'none',
  entry: {
    'large-number': './src/index.js',
    'large-number.min': './src/index.js'
  },
  output: {
    filename: '[name].js',
    library: 'largeNumber',
    libraryTarget: 'umd'
  },
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        include: /\.min\.js$/
      })
    ]
  }
}
```

#### 设置⼊⼝⽂件

- package.json 的 main 字段为 index.js

```js
// |- 根目录index.js文件下面
if (process.env.NODE_ENV === 'production') {
  module.exports = require('./dist/large-number.min.js')
} else {
  module.exports = require('./dist/large-number.js')
}
```

#### 大整数加法的代码

```js
export default function add(a, b) {
  let i = a.length - 1
  let j = b.length - 1

  let carry = 0
  let ret = ''
  while (i >= 0 || j >= 0) {
    let x = 0
    let y = 0
    let sum

    if (i >= 0) {
      x = a[i] - '0'
      i--
    }

    if (j >= 0) {
      y = b[j] - '0'
      j--
    }

    sum = x + y + carry

    if (sum >= 10) {
      carry = 1
      sum -= 10
    } else {
      carry = 0
    }
    // 0 + ''
    ret = sum + ret
  }

  if (carry) {
    ret = carry + ret
  }

  return ret
}
```

## 参考

[极客时间 webpack 课程](https://time.geekbang.org/course/detail/190-103056)

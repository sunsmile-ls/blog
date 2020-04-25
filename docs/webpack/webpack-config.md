## 构建配置抽离成 npm 包的意义

- 通用性
  - 业务开发者无需关注构建配置
  - 统一团队构建脚本
- 可维护性
  - 构建配置合理的拆分
  - README 文档、ChangeLog 文档等
- 质量
  - 冒烟测试、单元测试、测试覆盖率
  - 持续集成

## 构建配置管理的可选方案

- **通过多个配置文件管理不同环境的构建**，webpack --config 参数进行控制
- 将**构建配置设计成一个库**，比如：hjs-webpack、Neutrino、webpack-blocks
- 抽成一个**工具进行管理**，比如：create-react-app, kyt, nwb
- 将所有的配置放在一个文件，通过 --env 参数控制分支选择

## 构建配置包设计

- 通过多个配置文件管理不同环境的 webpack 配置

  - 开发环境：webpack.dev.js
  - 生产环境：webpack.prod.js
  - SSR 环境：webpack.ssr.js

- 抽离成一个 npm 包统一管理
  - 规范：Git commit 日志、README、ESLint 规范、Semver 规范
  - 质量：冒烟测试、单元测试、测试覆盖率和 CI

#### 通过 webpack-merge 组合配置

## 功能模块设计

![1569136375061](.\images\1569136375061.png)

## 目录结构设计

- lib 放置源代码
- test 放置测试代码

```js
|- /test
|- /lib
	|- webpack.dev.js
	|- webpack.prod.js
	|- webpack.ssr.js
	|- webpack.base.js
|- README.md
|- CHANGELOG.md
|- .eslinrc.js
|- package.json
|- index.js
```

## 使用 ESLint 规范构建脚本

- 使用 eslint-config-airbnb-base
- eslint --fix 可以自动处理空格

```js
module.exports = {
  parser: 'babel-eslint',
  extends: 'airbnb-base',
  env: {
    browser: true,
    node: true
  }
}
```

## 冒烟测试执行

- 构建是否成功

```js
const path = require('path')
const webpack = require('webpack')
const rimraf = require('rimraf')
const Mocha = require('mocha')

const mocha = new Mocha({
  timeout: '10000ms'
})

process.chdir(path.join(__dirname, 'template'))

rimraf('./dist', () => {
  const prodConfig = require('../../lib/webpack.prod.js')

  webpack(prodConfig, (err, stats) => {
    if (err) {
      console.error(err)
      process.exit(2)
    }
    console.log(
      stats.toString({
        colors: true,
        modules: false,
        children: false
      })
    )

    console.log('Webpack build success, begin run test.')

    mocha.addFile(path.join(__dirname, 'html-test.js'))
    mocha.addFile(path.join(__dirname, 'css-js-test.js'))
    mocha.run()
  })
})
```

- 每次构建完成 build 目录是否有内容输出
  - 是否有 JS、CSS 等静态资源文件
  - 是否有 HTML 文件

```js
// html-test.js
const glob = require('glob-all')

describe('Checking generated html files', () => {
  it('should generate html files', done => {
    const files = glob.sync(['./dist/index.html', './dist/search.html'])

    if (files.length > 0) {
      done()
    } else {
      throw new Error('no html files generated')
    }
  })
})

// css-js-test.js
const glob = require('glob-all')

describe('Checking generated css js files', () => {
  it('should generate css js files', done => {
    const files = glob.sync([
      './dist/index_*.js',
      './dist/index_*.css',
      './dist/search_*.js',
      './dist/search_*.css'
    ])

    if (files.length > 0) {
      done()
    } else {
      throw new Error('no css js files generated')
    }
  })
})
```

## 单元测试与测试覆盖率

#### 编写单元测试用例

- 技术选型：Mocha + Chai
- 测试代码：describe, it, except
  - describe: 描述文件
  - it ： 每一个测试用例
  - except ： 断言

```js
const expect = require('chai').expect
const add = require('../src/add')
describe('use expect: src/add.js', () => {
  it('add(1, 2) === 3', () => {
    expect(add(1, 2).to.equal(3))
  })
})
```

- 测试命令：mocha add.test.js

#### 单元测试接入

1.  安装 mocha + chai

npm i mocha chai -D

2. 新建 test 目录，并增加 xxx.test.js 测试文件

3. 在 package.json 中的 scripts 字段增加 test 命令

   ```js
   "scripts": {
   	"test": "node_modules/mocha/.bin/_mocha”
   },
   ```

4. 执行测试命令
   npm run test

#### 项目中使用

```js
const path = require('path')

process.chdir(path.join(__dirname, 'smoke/template'))

describe('builder-webpack test case', () => {
  require('./unit/webpack-base-test')
})

// ./unit/webpack-base-test.js
const assert = require('assert')

describe('webpack.base.js test case', () => {
  const baseConfig = require('../../lib/webpack.base.js')

  it('entry', () => {
    assert.equal(baseConfig.entry.index, 'template/src/index/index.js')
    assert.equal(baseConfig.entry.search, '/template/src/search/index.js')
  })
})
```

#### 集成覆盖率测试

- 安装 Istanbul

```
npm i istanbul -D
```

- 添加到配置文件中

```js
"scripts": {
    "test": "istanbul cover ./node_modules/.bin/_mocha",
 },
```

## 集成持续集成

- 优点：
  - 快速发现错误
  - 防止分支大幅偏离主干

> 核心措施：代码集成到主干之前，必须通过自动化测试，只要有一个偏离主干，就不能集成

### 接入 Travis CI

1. https://travis-ci.org/ 使用 GitHub 账号登录
2. 在 https://travis-ci.org/account/repositories 为项目开启
3. 项目根目录下新增 .travis.yml

#### travis.yml 文件内容

```yaml
language: node_js

sudo: false

cache:
	apt: true
	directories:
		_ node_modules

node_js: stable #设置相应的版本

install:
	- npm install -D #安装构造器依赖
	- cd ./test/template-project
	- npm install -D #安装模板项目依赖
script:
	- npm test
```

## 发布到 npm

##### 添加用户： npm adduser

##### 升级版本

- 升级补丁版本号：npm version patch
- 升级小版本号：npm version minor
- 升级大版本号：npm version major

##### 发布版本：npm publish

## Git 规范和 Changelog 生成

良好的 Git commit 规范优势：

- ·加快 Code Review 的流程
- 根据 Git Commit 的元数据生成 Changelog
- 后续维护者可以知道 Feature 被修改的原因

#### 技术方案

![1569161992055](.\images\1569161992055.png)

#### 提交格式要求

![1569162043831](.\images\1569162043831.png)

#### 本地开发阶段增加 precommit 钩子

- 安装 husky

  npm install husky --save-dev

```js
"scripts": {
	"commitmsg": "validate-commit-msg",
	"changelog": "conventional-changelog -p
	angular -i CHANGELOG.md -s -r 0"
},
"devDependencies": {
    "validate-commit-msg": "^2.11.1",
    "conventional-changelog-cli": "^1.2.0",
    "husky": "^0.13.1"
}
```

## 开源项目版本信息案例

- 软件的版本通常由三位组成，形如：X.Y.Z

  > 遵守 semver 规范的优势

  - 优势：
    ·避免出现循环依赖
    ·依赖冲突减少

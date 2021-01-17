## webpack 打包原理分析

webpack 在执⾏ npx webpack 进⾏打包后，都⼲了什么事情？

```js
(function(modules) {
    var installedModules = {};
    function __webpack_require__(moduleId) {
        if (installedModules[moduleId]) {
            return installedModules[moduleId].exports;
        }
        var module = (installedModules[moduleId] = {
            i: moduleId,
            l: false,
            exports: {}
        });
        modules[moduleId].call(
            module.exports,
            module,
            module.exports,
            __webpack_require__
        );
        module.l = true;
        return module.exports;
    }
    return __webpack_require__((__webpack_require__.s = "./index.js"));
})({
    "./index.js": function(module, exports) {
        eval(
            '// import a from "./a";\n\nconsole.log("hello word");\n\n\n//#
            sourceURL=webpack:///./index.js?'
        );
    }
});
```

⼤概的意思就是，实现了⼀个**webpack_require** 来实现⾃⼰的模块化，把代码都缓存在 **installedModules**⾥，代码⽂件以对象传递进来，key 是路径，value 是包裹的代码字符串，并且代码内 部的 require，都被替换成了**webpack_require**

## ⾃⼰实现⼀个 bundle.js

- 模块分析：读取⼊⼝⽂件，分析代码

```js
const fs = require('fs')
const babelParser = require('@babel/parser')
const path = require('path')
const traverse = require('@babel/traverse').default
const babel = require('@babel/core')

const moduleAnalyser = entry => {
  const content = fs.readFileSync(entry, 'utf-8')
  // 1. 获取到ast树
  const ast = babelParser.parse(content, {
    sourceType: 'module'
  })

  // 2. ast 经过解析获取到导入的文件路径 比如import a from 'a.js' 中的a.js
  const dependencies = {}
  traverse(ast, {
    ImportDeclaration({ node }) {
      const dirname = path.dirname(entry) // 获取入口文件的目录
      const newFile = './' + path.join(dirname, node.source.value)
      dependencies[node.source.value] = newFile
      // dependencies.push(node.source.value);
    }
  })
  // console.log(dependencies) // { './a.js': './src\\a.js', './b.js': './src\\b.js' }
  // 3. 根据 ast 解析出 code
  const { code } = babel.transformFromAst(ast, null, {
    presets: ['@babel/preset-env']
  })
  return {
    entry,
    dependencies,
    code
  }
}

const entry = fenximokuai('./src/index.js')
console.log(entry)
```

- 根据 dependencies 获取文件

```js
const makeDependenciesToGraph = entry => {
  // 获取入口文件的解析结果
  const entryModule = moduleAnalyser(entry)

  // 解析出所有文件，并且放入到 数组中
  const moduleGraphArr = [entryModule]
  for (let i = 0; i < moduleGraphArr.length; i++) {
    const dependencies = moduleGraphArr[i].dependencies

    if (dependencies) {
      for (let j in dependencies) {
        moduleGraphArr.push(moduleAnalyser(dependencies[j]))
      }
    }
  }
  const graph = {} // 以文件名作为key，放到对象中
  moduleGraphArr.forEach(item => {
    graph[item.entry] = {
      dependencies: item.dependencies,
      code: item.code
    }
  })
  return graph
}
// {"./src/index.js":{"dependencies":{"./a.js":"./src\\a.js","./b.js":"./src\\b.js"},"code":"\"use strict\";\n\nvar _a = _interopRequireDefault(require(\"./a.js\"));\n\nvar _b = _interopRequireDefault(require(\"./b.js\"));\n\nfunction _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { \"default\": obj }; }\n\nconsole.log(_a[\"default\"] + _b[\"default\"]);"},"./src\\a.js":{"dependencies":{},"code":"\"use strict\";\n\nObject.defineProperty(exports, \"__esModule\", {\n  value: true\n});\nexports[\"default\"] = void 0;\nvar a = 10;\nvar _default = a;\nexports[\"default\"] = _default;"},"./src\\b.js":{"dependencies":{},"code":"\"use strict\";\n\nObject.defineProperty(exports, \"__esModule\", {\n  value: true\n});\nexports[\"default\"] = void 0;\nvar b = 10;\nvar _default = b;\nexports[\"default\"] = _default;"}}
```

- 生成代码

```js
const generateCode = entry => {
  const data = JSON.stringify(makeDependenciesToGraph(entry))
  console.log(data)
  return `
	(function(graph){
        function require(module){
          function localRequire(relativePath){
            return require(graph[module].dependencies[relativePath])
          }
          var exports = {};
          (function(require,exports,code){
            eval(code)
          })(localRequire,exports,graph[module].code)
          return exports;
        }
        require('${entry}')
      })(${data});
  `
}
```

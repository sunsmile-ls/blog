#### 查看是否有 ts 声明文件

https://microsoft.github.io/TypeSearch/

为社区贡献声明文件的方法

- https://definitelytyped.org

#### 编写声明文件

- global

```typescript
// global-lib.js
// 全局的js文件
function globalLib(options) {
  console.log(options)
}
globalLib.version = '1.0.0'

globalLib.doSomething = function() {
  console.log('globalLib do something')
}
```

声明文件

```typescript
// global-lib.d.ts
declare function globalLib(options: globalLib.Options): void

declare namespace globalLib {
  const version: string
  function doSomething(): void
  interface Options {
    [key: string]: any
  }
}
```

- cmd

```typescript
// module-lib.js
const version = '1.0.0'

function doSomething() {
  console.log('moduleLib do something')
}

function moduleLib(options) {
  console.log(options)
}

moduleLib.version = version
moduleLib.doSomething = doSomething

module.exports = moduleLib
```

声明文件

```typescript
// module-lib.d.ts
declare function moduleLib(options: Options): void

interface Options {
  // 本身是一个模块，所以不会向外暴露
  [key: string]: any
}

declare namespace moduleLib {
  const version: string
  function doSomething(): void
}

export = moduleLib //兼容写法
```

- umd

```typescript
;(function(root, factory) {
  if (typeof define === 'function' && define.amd) {
    define(factory)
  } else if (typeof module === 'object' && module.exports) {
    module.exports = factory()
  } else {
    root.umdLib = factory()
  }
})(this, function() {
  return {
    version: '1.0.0',
    doSomething() {
      console.log('umdLib do something')
    }
  }
})
```

声明文件的编写

```typescript
declare namespace umdLib {
  const version: string
  function doSomething(): void
}

export as namespace umdLib //如果为umd类库，必须写这一条

export = umdLib
```

#### 给类库添加自定义方法

```typescript
// 模块插件
import m from 'moment'
declare module 'moment' {
  export function myFunction(): void
}
m.myFunction = () => {}

// 全局插件
declare global {
  namespace globalLib {
    function doAnyting(): void
  }
}
globalLib.doAnyting = () => {}
```

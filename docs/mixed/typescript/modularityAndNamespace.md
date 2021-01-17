#### 模块化的兼容写法

```typescript
export = function() {
  // 兼容性导出，这个是顶级导出，这样写之后不能通过其他方式导出
  console.log("I'm default")
}
```

tsconfig.json 中如果*"esModuleInterop"*: true ，则可以利用 es6 导入方式

如果没有开启则为下面的

```typescript
import c4 = require('../es6/d')
```

#### 命名空间

```typescript
/// <reference path="a.ts" />
namespace Shape {
  export function square(x: number) {
    return x * x
  }
}
// 不要和模块混用，最好在全局的情况下使用
console.log(Shape.cricle(2))
console.log(Shape.square(2))

import cricle = Shape.cricle // cricle是es模块 ,定义别名
console.log(cricle(2))
```

- 实现原理

立即执行函数构成闭包

- 要点

1. 局部变量对外不可见
2. 导出成员对外可见
3. 多个文件可共享同名命名空间
4. 依赖关系 ///<reference path =" " />
5. 命名空间的函数支持别名

#### 声明合并（命名空间要在类和函数声明之后）

![image-20200412210159190](.\images\image-20200412210159190.png)

```typescript
interface A {
  x: number
  // y: string;
  foo(bar: number): number // 5
  foo(bar: 'a'): string // 2
}

interface A {
  y: number
  foo(bar: string): string // 3
  foo(bar: string[]): string[] // 4
  foo(bar: 'b'): string // 1
}
```

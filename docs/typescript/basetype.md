## 类型注解

作用： 相当于强类型语言中的类型声明

语法： (变量/函数):type

```js
// 原始类型
let bool: boolean = true
let num: number = 123 // TypeScript 还支持 ECMAScript 2015中引入的二进制和八进制字面量。
let str: string = 'abc' // 可以用双引号（"）或单引号（'），模板字符串，拼接字符串

// 数组的两种声明方式
let arr1: number[] = [1, 2, 3]
let arr2: Array<number | string> = [1, 2, 3, '4']

// 函数
let add = (x: number, y: number) => x + y
let compute: (x: number, y: number) => number
compute = (a, b) => a + b

// 对象
let obj: { x: number, y: number } = { x: 1, y: 2 }
//如果obj后面为Object,则不能用obj.x去访问属性
obj.x = 3

// symbol
let s1: symbol = Symbol()
let s2 = Symbol()

// undefined, null是任何类型的子类型，
// 如果赋值给number,需要tsconfig.json中设置strictNullChecks为false
let un: undefined = undefined
let nu: null = null

// void
let noReturn = () => {}

// never 抛出异常和死循环为never类型
let error = () => {
  throw new Error('error')
}
let endless = () => {
  while (true) {}
}
```

## 元组 Tuple

元组类型允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。 比如，你可以定义一对值分别为 `string` 和 `number` 类型的元组。

```js
let x: [string, number]
x = ['hello', 10] // OK
x = [10, 'hello'] // Error
```

当访问一个已知索引的元素，会得到正确的类型：

```js
console.log(x[0].substr(1)) // OK
console.log(x[1].substr(1)) // Error, 'number' 不存在 'substr' 方法
```

当访问一个越界的元素，会使用联合类型替代：

```js
x[3] = 'world' // OK, 字符串可以赋值给(string | number)类型

console.log(x[5].toString()) // OK, 'string' 和 'number' 都有 toString

x[6] = true // Error, 布尔不是(string | number)类型
```

联合类型是高级主题，我们会在以后的章节里讨论它。

**注意**：自从 TyeScript 3.1 版本之后，访问越界元素会报错，我们不应该再使用该特性。

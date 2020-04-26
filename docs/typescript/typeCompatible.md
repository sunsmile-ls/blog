## ts 类型推断

```javascript
// 根据右边的值类型推断出左边变量的类型
let a = 1
let b = [1, null, 'a']
let c = { x: 1, y: 'a' }

let d = (x = 1) => x + 1

//根据左边的事件类型，推断出右边的参数类型
window.onkeydown = event => {
  // console.log(event.button)
}
```

## 类型兼容

当一个类型 Y 可以赋值给另一个类型 X，我们就可以说类型 X 兼容类型 Y

X 兼容 Y： X(目标类型)=Y（源类型）

**口诀**：

**结构之间兼容：成员少的兼容成员多的**

**函数之间的兼容：参数多的兼容参数少的**

- 接口的兼容性

```javascript
// 接口兼容性  成员少的可以兼容成员多的
interface X {
  a: any;
  b: any;
}
interface Y {
  a: any;
  b: any;
  c: any;
}
let x: X = { a: 1, b: 2 }
let y: Y = { a: 1, b: 2, c: 3 }
x = y
// y = x Y不兼容X
```

- 函数兼容

```javascript
// 函数兼容性
type Handler = (a: number, b: number) => void
function hof(handler: Handler) {
  return handler
}

// 1)参数个数
// 目标类型的参数个数要多于源类型的参数个数   X(目标类型)=Y（源类型）
let handler1 = (a: number) => {}
hof(handler1)
let handler2 = (a: number, b: number, c: number) => {}
// hof(handler2)

// 可选参数和剩余参数
let a = (p1: number, p2: number) => {}
let b = (p1?: number, p2?: number) => {}
let c = (...args: number[]) => {}
a = b // 固定参数可以兼容可选参数和剩余参数
a = c
// b = a // 可选参数不兼容固定参数和剩余参数
// b = c // 如果strictFunctionTypes 为false则兼容
c = a // 剩余参数兼容固定参数和可选参数
c = b

// 2)参数类型
let handler3 = (a: string) => {}
// hof(handler3) // 类型不同不兼容

interface Point3D {
  x: number
  y: number
  z: number
}
interface Point2D {
  x: number
  y: number
}
let p3d = (point: Point3D) => {}
let p2d = (point: Point2D) => {}
p3d = p2d //把对象中的数据看成是单独的参数
// p2d = p23 //成员少的兼容于成员多的

// 3) 返回值类型 成员少的兼容于成员多的
let f = () => ({ name: 'Alice' })
let g = () => ({ name: 'Alice', location: 'Beijing' })
f = g
// g = f

// 函数重载
// 目标函数的参数个数要多于源函数的参数个数
function overload(a: number, b: number): number
function overload(a: string, b: string): string
function overload(a: any, b: any): any {}
// function overload(a: any): any {}
// function overload(a: any, b: any, c: any): any {}
// function overload(a: any, b: any) {}
```

- 枚举类型

```javascript
// 枚举兼容性
enum Fruit {
  Apple,
  Banana
}
enum Color {
  Red,
  Yellow
}
let fruit: Fruit.Apple = 1 // 枚举和数字之间互相兼容
let no: number = Fruit.Apple
// let color: Color.Red = Fruit.Apple // 枚举类型之间不兼容
```

- 类类型

```typescript
// 类兼容性
// 1） 静态成员和构造函数不作为比较
// 2) 如果包含私有成员，则子类和父类之间兼容，其他方式不兼容
class A {
  constructor(p: number, q: number) {}
  id: number = 1
  private name: string = ''
}
class B {
  static s = 1
  constructor(p: number) {}
  id: number = 2
  private name: string = ''
}
class C extends A {}
let aa = new A(1, 2)
let bb = new B(1)
// aa = bb
// bb = aa
let cc = new C(1, 2)
aa = cc
cc = aa
```

- 泛型兼容

```javascript
// 泛型兼容性
interface Empty<T> {
  // value: T
}
// 泛型函数只有在使用的时候可以判断兼容性
let obj1: Empty<number> = {}
let obj2: Empty<string> = {}
obj1 = obj2

let log1 = <T>(x: T): T => {
  console.log('x')
  return x
}
let log2 = <U>(y: U): U => {
  console.log('y')
  return y
}
// 定义相同的两个泛型函数，互相兼容
log1 = log2
```

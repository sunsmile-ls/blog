## 对象类型 interface

```javascript
interface List {
    readonly id: number; // 防止在其他地方修改变量
    name: string;
    // [x: string]: any;
    age?: number; // 接口里的属性不全都是必需的。 有些是只在某些条件下存在，或者根本不存在。
}
interface Result {
    data: List[]
}
function render(result: Result) {
    result.data.forEach((value) => {
        console.log(value.id, value.name)
        if (value.age) {
            console.log(value.age)
        }
        // value.id++
    })
}
let result = {
    data: [
        {id: 1, name: 'A', sex: 'male'}, //sex不在声明中，ts采用了鸭式变型法，可以通过检查
        {id: 2, name: 'B', age: 10}
    ]
}
render(result) // 这样不会检查
```

- 如果 result 的值放入到 render 函数中，会对传入的字段进行校验

```javascript
render({
  data: [
    { id: 1, name: 'A', sex: 'male' }, //会对sex进行校验，ts会报错
    { id: 2, name: 'B', age: 10 }
  ]
})

// 解决方案（1）
render({
  data: [
    { id: 1, name: 'A', sex: 'male' }, //会对sex进行校验，ts会报错
    { id: 2, name: 'B', age: 10 }
  ]
} as Result)

// 解决方案（2）
interface List {
  readonly id: number
  name: string
  [x: string]: any // 添加字符串索引签名，含义为用任意的字符串索引list，会得到任意的字符串
  age?: number
}
```

## 可索引的类型

```javascript
interface StringArray {
  // 相当有声明了一个数组
  // 用任意的number索引StringArray，都会得到一个字符串
  [index: number]: string
}
let chars: StringArray = ['a', 'b']

interface Names {
  [x: string]: string // 用任意的字符串索引Names，得到一个字符串
  [z: number]: number // 这样不兼容，必须为数字索引签名的返回值，必须为字符串索引签名的子类型
}
```

## 函数类型 interface

```javascript
interface Add {
  (x: number, y: number): number;
}
// 使用类型别名定义方法
type Add = (x: number, y: number) => number
// 使用变量定义函数
let add: Add = (a: number, b: number) => a + b
```

## 混合类型

```javascript
// 接口中既有函数又有属性
interface Lib {
  (): void
  version: string
  doSomething(): void
}

function getLib() {
  let lib = (() => {}) as Lib
  lib.version = '1.0.0'
  lib.doSomething = () => {}
  return lib
}
```

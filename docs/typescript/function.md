#### 函数类型

```typescript
// 函数定义
function add1(x: number, y: number) {
  return x + y
}

let add2: (x: number, y: number) => number

type add3 = (x: number, y: number) => number

interface add4 {
  (x: number, y: number): number
}
```

- 函数参数不能少，也不能多

```typescript
add1(1, 2, 3) // 会报错
```

- 参数可选，需要在定义为可选参数

```typescript
function add5(x: number, y?: number) {
  // 可选参数必须位于必选参数之后
  return y ? x + y : x
}
add5(1)
```

- 默认参数

```typescript
function add6(x: number, y = 0, z: number, q = 1) {
  return x + y + z + q
}
add6(1, undefined, 3) // 在必选参数之前，默认参数是不可以省略的，必须传递undefined
```

- 剩余参数

```typescript
function add7(x: number, ...rest: number[]) {
  return x + rest.reduce((pre, cur) => pre + cur)
}
add7(1, 2, 3, 4, 5)
```

- 函数重载

ts 函数重载，需要定义相同名称的一些列函数，最后在类型宽泛的函数中实现重载

!> ts 会从上往下匹配函数，把最容易匹配到的函数放到前面

```typescript
function add8(...rest: number[]): number
function add8(...rest: string[]): string
function add8(...rest: any[]) {
  let first = rest[0]
  if (typeof first === 'number') {
    return rest.reduce((pre, cur) => pre + cur)
  }
  if (typeof first === 'string') {
    return rest.join('')
  }
}
console.log(add8(1, 2))
console.log(add8('a', 'b', 'c'))
```

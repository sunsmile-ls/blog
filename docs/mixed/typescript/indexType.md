## 索引类型

```typescript
let obj = {
  a: 1,
  b: 2,
  c: 3
}

// function getValues(obj: any, keys: string[]) {
//     return keys.map(key => obj[key])
// }
function getValues<T, K extends keyof T>(obj: T, keys: K[]): T[K][] {
  return keys.map(key => obj[key])
}
console.log(getValues(obj, ['a', 'b']))
// console.log(getValues(obj, ['d', 'e']))

// keyof T  索引类型查询操作符
interface Obj {
  a: number
  b: string
}
let key: keyof Obj // key:number|string

// T[K] 索引访问操作符 对象T的属性key所代表的类型
let value: Obj['a'] // 表示Obj的属性a对应的类型

// T extends U // 泛型变量可以继承某个类型，获得某些属性
```

## 映射类型

```typescript
interface Obj {
  a: string
  b: number
}
type ReadonlyObj = Readonly<Obj> // 把所有类型变为只读属性

type PartialObj = Partial<Obj> // 把所有类型变为可选属性

type PickObj = Pick<Obj, 'a' | 'b'> // 对象中的几个属性

type RecordObj = Record<'x' | 'y', Obj> // 变为{x:Obj,y:Obj}
```

## 条件类型

```typescript
// T extends U ? X : Y
// 如果类型T可以赋值给U,则是X类型，否则为Y类型

type TypeName<T> = T extends string
  ? 'string'
  : T extends number
  ? 'number'
  : T extends boolean
  ? 'boolean'
  : T extends undefined
  ? 'undefined'
  : T extends Function
  ? 'function'
  : 'object'
type T1 = TypeName<string>
type T2 = TypeName<string[]>

// (A | B) extends U ? X : Y
// (A extends U ? X : Y) | (B extends U ? X : Y)
type T3 = TypeName<string | string[]>

type Diff<T, U> = T extends U ? never : T
type T4 = Diff<'a' | 'b' | 'c', 'a' | 'e'>
// Diff<"a", "a" | "e"> | Diff<"b", "a" | "e"> | Diff<"c", "a" | "e">
// never | "b" | "c"
// "b" | "c"

type NotNull<T> = Diff<T, null | undefined>
type T5 = NotNull<string | number | undefined | null>

// Exclude<T, U>  //从类型T中过滤掉可以赋值给类型U的类型
// NonNullable<T> // 去掉null和undefined

// Extract<T, U> // 从类型T中选出可以赋值给类型U的类型
type T6 = Extract<'a' | 'b' | 'c', 'a' | 'e'>

// ReturnType<T> // 返回函数的返回值类型
type T8 = ReturnType<() => string>
```

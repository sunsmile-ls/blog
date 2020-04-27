## 枚举

`enum` 类型是对 JavaScript 标准数据类型的一个补充。 像 C# 等其它语言一样，使用枚举类型可以为一组数值赋予友好的名字。

##### 数字枚举

```typescript
// 数字枚举
enum Color {
  Red,
  Green,
  Blue
}
let c: Color = Color.Green
```

默认情况下，从 `0` 开始为元素编号。 你也可以手动的指定成员的数值。 例如，我们将上面的例子改成从 `1` 开始编号：

```typescript
enum Color {
  Red = 1,
  Green,
  Blue
}
let c: Color = Color.Green
```

编译后的 code

```typescript
'use strict'
var Color
;(function(Color) {
  Color[(Color['Red'] = 1)] = 'Red'
  Color[(Color['Green'] = 2)] = 'Green'
  Color[(Color['Blue'] = 3)] = 'Blue'
})(Color || (Color = {}))
```

或者，全部都采用手动赋值：

```typescript
enum Color {
  Red = 1,
  Green = 2,
  Blue = 4
}
let c: Color = Color.Green
```

枚举类型提供的一个便利是你可以由枚举的值得到它的名字。 例如，我们知道数值为 2，但是不确定它映射到 Color 里的哪个名字，我们可以查找相应的名字：

```typescript
enum Color {
  Red = 1,
  Green,
  Blue
}
let colorName: string = Color[2]

console.log(colorName) // 显示'Green'因为上面代码里它的值是2
```

#### 字符串枚举

**字符串枚举不会映射**

```typescript
// 字符串枚举
enum Message {
  Success = '恭喜你，成功了',
  Fail = '抱歉，失败了'
}
```

编译后

```typescript
'use strict'
var Message
;(function(Message) {
  Message['Success'] = '\u606D\u559C\u4F60\uFF0C\u6210\u529F\u4E86'
  Message['Fail'] = '\u62B1\u6B49\uFF0C\u5931\u8D25\u4E86'
})(Message || (Message = {}))
```

##### 异构枚举(不推荐使用)

```typescript
enum Answer {
  N,
  Y = 'Yes'
}
```

枚举成员为只读成员，不可以修改值

```typescript
// 枚举成员
enum Char {
  // const member 常量成员 在编译阶段计算
  a,
  b = Char.a,
  c = 1 + 3,
  // computed member 计算成员 在运行时计算
  d = Math.random(),
  e = '123'.length,
  f = 4 // 这儿会抛错
}
```

**在计算成员之后，不能添加常量成员**

#### 常量枚举

```typescript
const enum Month {
  Jan,
  Feb,
  Mar,
  Apr = Month.Mar + 1
  // May = () => 5
}
```

编译之后不会生成代码，用处：在不需要对象，只需要对象的值的时候可以使用。

```typescript
// 枚举类型
enum E {
  a,
  b
}
enum F {
  a = 0,
  b = 1
}
enum G {
  a = 'apple',
  b = 'banana'
}

let e: E = 3
let f: F = 3
// console.log(e === f) Error不同的枚举成员类型，不能比较

let e1: E.a = 3
let e2: E.b = 3
let e3: E.a = 3
// 不同的枚举成员类型，不能比较
// console.log(e1 === e2)
// console.log(e1 === e3)

let g1: G = G.a
let g2: G.a = G.a
```

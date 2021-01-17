#### 交叉类型和联合类型

```typescript
interface DogInterface {
  run(): void
}
interface CatInterface {
  jump(): void
}
let pet: DogInterface & CatInterface = {
  // 交叉类型是所有类型都实现
  run() {},
  jump() {}
}

// 联合类型，只符合其中一项就可以了
let a: number | string = 1
let b: 'a' | 'b' | 'c'
let c: 1 | 2 | 3

class Dog implements DogInterface {
  run() {}
  eat() {}
}
class Cat implements CatInterface {
  jump() {}
  eat() {}
}
enum Master {
  Boy,
  Girl
}
function getPet(master: Master) {
  // 只能调用两个类型共有的方法
  let pet = master === Master.Boy ? new Dog() : new Cat()
  // pet.run()
  // pet.jump()
  pet.eat()
  return pet
}

interface Square {
  kind: 'square'
  size: number
}
interface Rectangle {
  kind: 'rectangle'
  width: number
  height: number
}
interface Circle {
  kind: 'circle'
  radius: number
}
type Shape = Square | Rectangle | Circle
// 通过共有属性区分类型
function area(s: Shape) {
  switch (s.kind) {
    case 'square':
      return s.size * s.size
    case 'rectangle':
      return s.height * s.width
    case 'circle':
      return Math.PI * s.radius ** 2
    default:
      return ((e: never) => {
        throw new Error(e)
      })(s)
  }
}
console.log(area({ kind: 'circle', radius: 1 }))
```

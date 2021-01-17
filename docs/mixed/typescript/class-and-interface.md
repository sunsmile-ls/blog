#### interface and class

```typescript
// 接口只能约束类的共有成员
// 接口不能约束类的构造函数
interface Human {
  name: string
  eat(): void
}

class Asian implements Human {
  // 类实现接口，需要实现所有接口的成员
  constructor(name: string) {
    this.name = name
  }
  name: string
  eat() {}
  age: number = 0
  sleep() {}
}
```

- interface 可以继承 interface，并且可以继承多个接口

```typescript
interface Man extends Human {
  run(): void
}

interface Child {
  cry(): void
}

interface Boy extends Man, Child {}
```

- interface 可以继承 class

```typescript
class Auto {
  // 相当于类只有类的成员结构，没有类的实现
  state = 1
  protected state2 = 1
}
interface AutoInterface extends Auto {}
class C1 implements AutoInterface {
  // 如果接口继承的类有private、protected，则不能被实现
  state = 1
  // protected state2 = 3
}
class Bus extends Auto implements AutoInterface {}
```

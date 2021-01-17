## 类的声明

```typescript
class Dog {
  constructor(name: string) {
    // 构造函数的参数增加了类型注解
    this.name = name
  }
  public name: string = 'dog' // 成员属性需要加类型注解
  run() {}
}
// 成员属性需要初始化，要么构造函数中初始化，要么在类型属性声明的时候初始化
// 设置为可选属性 public name?: string,可以不初始化
```

- 类成员的属性都是实例属性，类成员的方法，都是原型方法

## 公共，私有与受保护的修饰符

```typescript
class Dog extends Animal {
  constructor(name: string) {
    super()
    this.name = name
    this.pri()
  }
  public name: string = 'dog'
  run() {}
  private pri() {} // 只能被本身调用，不能被子类调用或者实例调用
  protected pro() {} // 只可以在类或者子类中访问，不可以在实例中访问
  readonly legs: number = 4 // 声明只读的属性一定要被初始化
  static food: string = 'bones' // 静态成员只能通过类名去调用
}
let dog = new Dog('wangwang')
// dog.pri()   [Error]
// dog.pro()   [Error]
console.log(Dog.food)
dog.eat()

class Husky extends Dog {
  constructor(name: string, public color: string) {
    super(name)
    this.color = color
    // this.pri() [Error]
    this.pro()
  }
  // color: string  // 可以省去
}
console.log(Husky.food) // 类的静态成员可以被继承
```

- 构造函数加`private`,表示既不可以被实例化，也不可以继承
- 构造函数加`protected`,表示不可以被实例化，只能被继承

## 参数属性

```typescript
class Husky extends Dog {
  // public color: string, 用public修饰，可以省去成员变量color的声明
  constructor(name: string, public color: string) {
    super(name)
    this.color = color
  }
  // color: string
}
```

## 抽象类

```typescript
abstract class Animal {
  eat() {
    // 抽象类可以实现一个具体的方法，子类不需要实现,就可以访问
    console.log('eat')
  }
  abstract sleep(): void // 抽象类不指定方法的具体实现，构成了一个抽象方法
}
// let animal = new Animal()   //抽象类不能被实例化

class Dog extends Animal {
  constructor(public name: string) {
    super()
    this.name = name
    this.pri()
  }
  sleep() {
    // 子类必须实现抽象方法
    console.log('Dog sleep')
  }
}
```

- 多态： 父类定义了方法，子类有不同的实现，在程序运行时，不同的实例，不同的操作

```typescript
class Workflow {
  step1() {
    return this
  }
  step2() {
    return this
  }
}
new Workflow().step1().step2()

class MyFlow extends Workflow {
  next() {
    return this
  }
}
new MyFlow()
  .next()
  .step1()
  .next()
  .step2() // this 可以是父类型，也可以是子类型，保证了方法调用的连贯性
```

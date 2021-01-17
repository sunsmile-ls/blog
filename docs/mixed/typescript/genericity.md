#### 泛型

* 泛型约束方法

```typescript
function log<T>(value: T): T { //把参数类型的声明延迟到使用时再定义
    console.log(value);
    return value;
}
// 下面两个等同于上面的函数
// type Log = <T>(value: T) => T
// let myLog: Log = log

// interface Log<T> { // 泛型约束接口的时候，在声明的时候必须指定类型
//     (value: T): T
// }
// let myLog: Log<number> = log
// myLog(1)
log<string[]>(['a', ',b', 'c'])
log(['a', ',b', 'c'])
```

* 泛型约束类

```typescript
class Log<T> { // 泛型不可以约束类的静态成员
    run(value: T) {
        console.log(value)
        return value
    }
}
let log3 = new Log<number>()  // 在实例化的时候需要指定类型
log3.run(1)
let log4 = new Log() // 不指定类型，就可以传递任何参数
log4.run({ a: 1 })
```

* 泛型约束

```typescript
interface Length {
  length: number
}
function logAdvance<T extends Length>(value: T): T {
  //因为这儿使用了length的属性所以需要约束一下泛型<T extends Length>
  console.log(value, value.length) 
  return value
}
logAdvance([1])
logAdvance('123')
logAdvance({ length: 3 })
```


## 如何实现一个 call 函数

```javascript
Function.prototype.myCall = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('error')
  }
  // 给函数绑定this,如果不传递则为window
  var context = context || window
  // 给 context 添加一个属性
  context.fn = this
  // 将 context 后面的参数取出来
  var args = [...arguments].slice(1)
  var result = context.fn(...args)
  // 删除 fn
  delete context.fn
  return result
}
```

## 如何实现一个 apply 函数

```javascript
Function.prototype.myApply = function (context) {
  var context = context || window
  context.fn = this

  var result
  // 需要判断是否存储第二个参数
  // 如果存在，就将第二个参数展开
  if (arguments[1]) {
    result = context.fn(...arguments[1])
  } else {
    result = context.fn()
  }

  delete context.fn
  return result
}
```

## 如何实现一个 bind 函数

```javascript
Function.prototype.myBind = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  var _this = this
  var args = [...arguments].slice(1)
  // 返回一个函数
  return function F() {
    // 因为返回了一个函数，我们可以 new F()，所以需要判断
    if (this instanceof F) {
      return new _this(...args, ...arguments)
    }
    return _this.apply(context, args.concat(...arguments))
  }
}
```

因为 bind 返回的是一个函数，对于函数有两种调用方式，一种是直接调用，一种是通过 new 方式

1. 直接调用：apply 的方式实现，`f.bind(obj,1)(2)`,所以利用了`args.concat(...arguments)`
2. `new`方式：对于`new`来说，不会被任何情况改变 this,所以对于这种情况需要忽略传入的 this

## 实现一个 new 函数

```javascript
/**
 * 新生成了一个对象
 * 链接到原型
 * 绑定 this
 * 返回新对象
 */
function create() {
  // 创建一个空对象
  const obj = {}
  // 获取构造函数
  const Con = [].shift.call(arguments)
  // 链接到原型
  obj.__proto__ = Con.prototype
  // 绑定this
  const result = Con.apply(obj, arguments)
  return result instanceof Object ? result : obj
}
```

## 实现一个 instanceof 函数

instanceof 是通过原型链是否可以正确的找到类型的 prototype

```javascript
function myInstanceOf(left, right) {
  let prototype = right.prototype

  left = left.__proto__
  while (true) {
    if (left === null || left === undefined) return false
    if (prototype === left) return true
    left = left.__proto__
  }
}
```

## 简洁版的 promise

```javascript
// 三个常量用于表示状态
const PENDING = 'pending'
const RESOLVED = 'resolved'
const REJECTED = 'rejected'

function MyPromise(fn) {
  const that = this
  this.state = PENDING

  // value 变量用于保存 resolve 或者 reject 中传入的值
  this.value = null

  // 用于保存 then 中的回调，因为当执行完 Promise 时状态可能还是等待中，这时候应该把 then 中的回调保存起来用于状态改变时使用
  that.resolvedCallbacks = []
  that.rejectedCallbacks = []

  function resolve(value) {
    // 首先两个函数都得判断当前状态是否为等待中
    if (that.state === PENDING) {
      that.state = RESOLVED
      that.value = value

      // 遍历回调数组并执行
      that.resolvedCallbacks.map((cb) => cb(that.value))
    }
  }
  function reject(value) {
    if (that.state === PENDING) {
      that.state = REJECTED
      that.value = value
      that.rejectedCallbacks.map((cb) => cb(that.value))
    }
  }

  // 完成以上两个函数以后，我们就该实现如何执行 Promise 中传入的函数了
  try {
    fn(resolve, reject)
  } catch (e) {
    reject(e)
  }
}

// 最后我们来实现较为复杂的 then 函数
MyPromise.prototype.then = function (onFulfilled, onRejected) {
  const that = this

  // 判断两个参数是否为函数类型，因为这两个参数是可选参数
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : (v) => v
  onRejected = typeof onRejected === 'function' ? onRejected : (e) => throw e

  // 当状态不是等待态时，就去执行相对应的函数。如果状态是等待态的话，就往回调函数中 push 函数
  if (this.state === PENDING) {
    this.resolvedCallbacks.push(onFulfilled)
    this.rejectedCallbacks.push(onRejected)
  }
  if (this.state === RESOLVED) {
    onFulfilled(that.value)
  }
  if (this.state === REJECTED) {
    onRejected(that.value)
  }
}
```

## 防抖

```javascript
/**
 * 防抖函数，返回函数连续调用时，空闲时间必须大于或等于 wait，func 才会执行
 *
 * @param  {function} func        回调函数
 * @param  {number}   wait        表示时间窗口的间隔
 * @param  {boolean}  immediate   设置为ture时，是否立即调用函数
 * @return {function}             返回客户调用函数
 */
function debounce(func, wait = 5, immediate = true) {
  let timer, context, args
  const later = () =>
    setTimeout(() => {
      // 延时队列执行完毕，清空缓存中序号
      timer = null
      // 延时执行的情况下，函数会在延迟函数中执行
      // 使用到之前缓存的参数和上下文
      if (!immediate) {
        func.apply(context, args)
        context = args = null
      }
    }, wait)
  // 每次调用的函数
  return function (...params) {
    if (!timer) {
      timer = later()
      if (immediate) {
        func.apply(this, params)
      } else {
        context = this
        args = params
      }
    } else {
      // 如果已有延迟执行函数（later），调用的时候清除原来的并重新设定一个
      // 这样做延迟函数会重新计时
      clearTimeout(timer)
      timer = later()
    }
  }
}
```

## 节流

```javascript
// 获取当前时间戳
function now() {
  return +Date.now()
}

/**
 * underscore 节流函数，返回函数连续调用时，func 执行频率限定为 次 / wait
 *
 * @param  {function}   func      回调函数
 * @param  {number}     wait      表示时间窗口的间隔
 * @param  {object}     options   如果想忽略开始函数的的调用，传入{leading: false}。
 *                                如果想忽略结尾函数的调用，传入{trailing: false}
 *                                两者不能共存，否则函数不能执行
 * @return {function}             返回客户调用函数
 */
function throttle(func, wait, options) {
  var context, args, result
  var timeout = null
  // 之前的时间戳
  var previous = 0
  // 如果 options 没传则设为空对象
  if (!options) options = {}
  // 定时器回调函数
  var later = function () {
    // 如果设置了 leading，就将 previous 设为 0
    // 用于下面函数的第一个 if 判断
    previous =
      options.loading === false ? 0 : Date.now() || new Date().getTime()
    // 置空一是为了防止内存泄漏，二是为了下面的定时器判断
    timeout = null
    result = func.apply(context, args)
    if (!timeout) context = args = null
  }
  return function () {
    // 获得当前时间戳
    var now = Date.now() || new Date().getTime()
    // 首次进入前者肯定为 true
    // 如果需要第一次不执行函数
    // 就将上次时间戳设为当前的
    // 这样在接下来计算 remaining 的值时会大于0
    if (!previous && options.leading === false) previous = now
    // 计算剩余时间
    var remaining = wait - (now - previous)
    context = this
    args = arguments
    // 如果当前调用已经大于上次调用时间 + wait
    // 或者用户手动调了时间
    // 如果设置了 trailing，只会进入这个条件
    // 如果没有设置 leading，那么第一次会进入这个条件
    // 还有一点，你可能会觉得开启了定时器那么应该不会进入这个 if 条件了
    // 其实还是会进入的，因为定时器的延时
    // 并不是准确的时间，很可能你设置了2秒
    // 但是他需要2.2秒才触发，这时候就会进入这个条件
    if (remaining <= 0 || remaining > wait) {
      // 如果存在定时器就清理掉否则会调用二次回调
      if (timeout) {
        clearTimeout(timeout)
        timeout = null
      }
      previous = now
      result = func.apply(context, args)
      if (!timeout) context = args = null
    } else if (!timeout && options.trailing !== false) {
      // 判断是否设置了定时器和 trailing
      // 没有的话就开启一个定时器
      // 并且不能不能同时设置 leading 和 trailing
      timeout = setTimeout(later, remaining)
    }
    return result
  }
}
```

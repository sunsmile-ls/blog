- 初始化配置文件

```cmd
 npx jest --init
```

- 生成测试报告【同时会生成 coverage 页面】

```cmd
npx jest --coverage
```

## jest 导入 es 模块

需要安装 babel

```cmd
npm i @babel/core@7.4.5 @babel/preset-env@7.4.5 -D
```

创建`.babelrc`

```javascript
// 根据当前的node环境对 es6 模块导入进行转换
{
 "presets":[
   ["@babel/preset-env",{
     "targets":{
       "node": "current"
     }
   }]
 ]
}
```

## 匹配器

[expect](https://jestjs.io/docs/zh-Hans/expect)

## 命令行工具

jest 使用 watch 需要搭配 git 使用

在 watch 模式下，根据控制台提示进行操作，设置不同的模式

## 异步代码测试

1. 回调类型的异步需要加 done

```javascript
import axios from 'axios'

export const fetchData = (fn) => {
  axios.get('xxxxx.json').then((res) => {
    fn(res.data)
  })
}

test('test fetch 返回结果为 { success: true}', (done) => {
  fetchData((data) => {
    expect(data).toEqual({
      success: true,
    })
    done()
  })
})
```

2. 返回 promise

```javascript
import axios from 'axios'

export const fetchData = () => {
  return axios.get('xxxxx.json')
}

// 需要把promise 返回一下， 否则测试不生效
test('test fetch 返回结果为 { success: true}', () => {
  return fetchData().then((res) => {
    expect(res.data).toEqual({
      success: true,
    })
  })
})

test('test fetch 返回结果为 404', () => {
  // 处理测试错误返回的话，需要添加以下一句，意思为 expect 必须执行一次
  expect.assertions(1)
  return fetchData().catch((e) => {
    expect(e.toString().indexOf('404') > -1)).toBe(true)
  })
})
```

3. 通过 resolve 和 reject 测试 【下面两个 `return` 可以变为 `await` 】

```javascript
test('test fetch 返回结果为 { success: true}', () => {
  // 取数据并且拿到所有数据， 判断 是不是有 检查对象
  return expect(fetchData()).resolves.toMatchObject({
    data: {
      success: true,
    },
  })
})

test('test fetch 返回结果为 404', () => {
  // 取数据并且拿到所有数据， 判断是不是抛错
  return expect(fetchData()).rejects.toThrow()
})
```

4. 通过 async 和 await 测试

```javascript
test('test fetch 返回结果为 { success: true}', async () => {
  const res = await fetchData()
  return expect(res.data).toEqual({
    success: true,
  })
})

test('test fetch 返回结果为 404', async () => {
  expect.assertions(1)
  try {
    await fetchData()
  } catch (e) {
    expect(e.toString()).toEqual('Error: Request failed width status code 404')
  }
})
```

## jest 中的钩子函数

1. `beforeAll(()=>{})` 在所有的测试执行之前执行
2. `afterAll(()=>{})` 在所有的测试执行之后执行
3. `beforeEach(()=>{})` 在每个测试执行之前执行
4. `afterEach(()=>{})` 在每个测试执行之后执行

> 全局的钩子函数 对 describe 任然有效， 全局的 before 执行的早， 全局的 after 执行的较后

`test.only` 表示只执行此测试用例，其他的都会忽略

!> 代码不要直接放到 describe 中， 需要放到上面的 `before...`、`after...`, 防止有坑

## jest 中的 mock

```javascript
test('测试 test', () => {
  const func = jest.fn()
  expect(func).toBeCalled()
})
```

`func.mock` 有以下几个返回值

- `calls` 值为 `[['sunsmile']]` 表示被调用了几次,以及每次传入的值为什么

```javascript
expect(func.mock.calls[0]).toEqual(['sunsmile']) // 表示第一次调用的参数是 abc
expect(func).toBeCalledWith('abc') // 表示每次调用的参数是 abc
```

- `instances` 表示 this 的指向
- `invocationCallorder` 值为 `[1,2,3,4]`， 表示 mock 函数的调用顺序
- `results` 返回值为 `[{type:'return',value: 'sunsmile'}]`， 表示方法的返回值

```javascript
// 可以通过此方法设置返回值
func.mockReturnValueOnce('sunsmile').mockReturnValueOnce('daniel')
// 上面这句还可以写成
func.mockImplementationOnce(() => {
  return 'sunsmile'
})

func.mockImplementation(() => {
  return this
})
// 上面这个方法等价于下面的
func.mockReturnThis()
```

> mock 函数的作用

1. 捕获函数的调用和返回结果， this 和 调用顺序
2. 可以自由的设置返回结果
3. 改变函数的内部实现

```javascript
// a.js
export const getData = () => {
  return axios.get('/api').then((res) => res.data)
}

// a.test.js
import axios from 'axios'
jest.mock('axios')
test.only('测试 getData', async () => {
  axios.get.mockResolvedValue({ data: 'hello' })
  // axios.get.mockResolvedValueOnce({ data: 'hello' }) // 表示只返回一次值
  await getData().then((data) => {
    expect(data).toBe('hello')
  })
})
```

- 创建 mock 文件

```javascript
// demo.js
export const fetchData = () => {
  return axios.get('/api').then((res) => res.data)
}

// __mocks__/demo.js
// 表示用 mocks 下面的demo.js 会替换 demo.js
export const fetchData = () => {
  return new Promise((resolved, reject) => {
    resolved(123)
  })
}
// demo.test.js

// 告诉 jest 模拟 demo
jest.mock('./demo') // 可以用 jest.config.js 文件中 automock 设置为 ture 代替

// jest.unmock("./demo") // 取消 demo 的模拟
// 引入的是 __mocks__ 目录下面的 demo.js 文件
import { fetchData } from './demo.js'

test('fetchData test', () => {
  return fetchData().then((data) => {
    expect(data).toEqual(123)
  })
})

// 测试 demo 中的同步代码
const { syncFunc } = jest.requireActual('demo')
```

- 测试 timer

```javascript
// timer.js
export default (callback) => {
  setTimeout(() => {
    callback()
  })
}

// timer.test.js
import timer from './timer'
jest.useFakeTimers() //模拟 timer ，不需要等待特别长的时间
test('timer test', () => {
  const fn = jest.fn()
  timer(fn)
  // jest.runAllTimers() // 表示运行所有 timer
  // jest.runOnlyPendingTimers() // 表示只运行 pending 的 timer
  jest.advanceTimersByTime(3000) // 表示时间快进三秒
  expect(fn).toHaveBeenCalledTimers(1) // 表示timer 被调用了一次
})
```

## snapshot

使用 snapshot 生成快照

```javascript
// config.js
export const generateConfig = () => {
  return {
    server: 'http://localhost',
    port: 8000,
    time: new Date(),
  }
}

// config.test.js
import { generateConfig } from './config'

test('test generateConfig 函数', () => {
  // 第一次会生成快照，后面如果改变文件，生成新的快照，需要确认
  // 如果确认修改 按 u
  // 如果多个文件冲突， 需要按 i 进行 单个文件的确认， 如果确认修改则  按 u
  expect(generateConfig()).toMatchSnapshot({
    time: expect.any(Date), // 加入此行的意思是不需要和上一次的快照相等
  })
})
```

- 行内的 snapshot

1. 安装

```shell
npm i prettier@1.18.2 --save
```

2. 运行

```javascript
test('test generateConfig 函数', () => {
  expect(generateConfig()).toMatchInlineSnapshot() //生成的文件会放到 行内
})
```

## ES6 进行测试

jest.mock 发现文件声明式一个类， 会自动把类的构造函数和方法变成 jest.fn()

```javascript
class Utils {
  a(){

  },
}
// 会变为 const Util = jest.fn()  Util.a = jest.fn()
```

> jest 在 node 环境中自己模拟了一套 dom 的 api, 所以可以通过 document.getElementById() 获取元素

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

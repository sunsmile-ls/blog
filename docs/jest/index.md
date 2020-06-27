先看一段代码

```js
function expect(result) {
  return {
    toBe: function (actual) {
      console.log(actual, result)
      if (actual !== result) {
        throw new Error('预期值和期望值不一样')
      }
    },
  }
}

function test(desc, fn) {
  try {
    fn()
    console.log(`${desc} 通过了测试`)
  } catch (error) {
    console.log(`${desc} 没有通过 ${error}`)
  }
}

test('测试 2 + 4', () => {
  expect(add(2, 4)).toBe(6)
})

test('测试 4 - 2', () => {
  expect(minus(4, 2)).toBe(2)
})
```

通过上面的代码可以看出来 `expect` 函数，已经可以断言了，但是如果断言比较多的话，不能分清楚各自断言的方法，所以加了第二个`test`主要是表名意思的。

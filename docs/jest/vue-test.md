## Test Driven Development (TDD)

开发步骤：

1. 编写测试用例
2. 运行测试，测试用例无法通过测试
3. 编写代码，使测试用例通过测试
4. 优化代码，完成开发
5. 重复以上步骤

优势：

1. 长期减少回归 bug
2. 代码质量更好（组织，可维护性）
3. 测试覆盖率
4. 错误测试代码不容易出现

## vuex 的测试

```javascript
// store.js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)
const store = new Vuex.Store({
  state: {
    inputValue: '',
  },
  mutations: {
    changeInputValue(state, payload) {
      state.inputValue = payload
    },
  },
})

export default store

// store.test.js
import store from 'xxxx/store.js'
it('当 store 接受到 changeInputValue 的 commit 时， inputValue 发生变化', () => {
  const value = '123'
  store.commit('changeInputValue', value)
  expect(store.state.inputValue).toBe(value)
})
```

## vue 中获取前端 ajax

> 场景假设。在页面加载成功之后等待 5 秒，获取用户信息

```javascript
import axios from 'axios'

export default {
  mounted: {
    /**
      {
        success: true,
        data: [{
          username: 'sunsmile'
          role: 2
        }]
      }
    */
    setTimeout(()=>{
      axios.get('/getUserInfo').then(res=>{
          this.user = res.data
      })
    }, 5000) // jest 默认 最大延迟是五秒
  },
}

```

```javascript
// __mocks__/axios.js
const userInfo = {
  success: true,
  data: [{
    username: 'sunsmile'
    role: 2
  }]
}
export default {
  get(url) {
    if (url === 'getUserInfo') {
      return new Promise((resolve) => {
        resolve(userInfo)
      })
    }
  },
}

// test.js
beforeEach(()=>{
  jest.useFakeTimers() // 使用 jest 模拟原生的 timer
})
it(`
  1. 用户进入页面的时候，请求远程数据
  2. 应该展示相应的名字
  `, (done)=>{
    const wrapper =  mount(Header, {store})

    jest.runAllTimers() // 表示运行所有 timer
    wrapper.vm.$nextTick(()=>{
      const userContainer = findTestWrapper(wrapper, 'user-name')
      expect(userContainer.at(0).text).toBe('sunsmile')
      done()
    })
  })
```

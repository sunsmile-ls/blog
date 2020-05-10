## vuex

!> 本教程是根据 [vuex-4.0.0-beta.1](https://github.com/vuejs/vuex/tree/v4.0.0-beta.1)解读，并且会长期维护

> 我分析的方法是明确入口，分析出口，然后确认出口的方法

`vuex`借鉴了 `Flux`、`Redux` 。`Vuex` 是专门为 `Vue.js` 设计的状态管理库。

## 为什么要用 vuex?

- 多个视图依赖于同一状态。**[多层嵌套状态传递繁琐、兄弟组件状态无能为力]**
- 来自不同视图的行为需要变更同一状态。**[通过事件，props 传递状态会导致代码难以维护]**
- `Vuex` 也集成到 `Vue` 的官方调试工具 `devtools extension`，提供了诸如零配置的 `time-travel` 调试、状态快照导入导出等高级调试功能

## vuex 架构图

![vuex](./images/vuex.png)

## vuex 的基本使用

`vuex` 为了和 `Vue 3` 初始化过程保持一致，Vuex 的安装过程也已更改。

```javascript
// store.js
import { createStore } from 'vuex'

const debug = process.env.NODE_ENV !== 'production'

export default createStore({
  modules: {
    cart,
    products
  },
  strict: debug,
  plugins: debug ? [createLogger()] : []
})
```

```javascript
// main.js
import { createApp } from 'vue'
import store from './store'
import App from './APP.vue'

const app = createApp(App)

app.use(store)

app.mount('#app')
```

## 源码部分

因为创建 `store` 是通过调用 `createStore api` 创建的，返回一份 `Store` 的实例, 接下来我们来分析一下 `Store` 的构造函数

```javascript
constructor (options = {}) {
    const {
      //  应用在 store 中的插件，接收store本身
      plugins = [],
      // 是否开启严格模式，如果为 true 的话，只能在 mutation 中修改 store，外部修改会报错
      strict = false
    } = options

    // store internal state
    // 判断是不是在 mutation 中修改的 state，在 mutation 中修改会先把它改为 true
    this._committing = false
    // 存放 action
    this._actions = Object.create(null)
    this._actionSubscribers = []
    // 存放 mutation
    this._mutations = Object.create(null)
    // 存放 getter
    this._wrappedGetters = Object.create(null)
    // 收集 module [递归收集 子module 到_children 中]
    this._modules = new ModuleCollection(options)
    // 根据 nameSpace 存放 module
    this._modulesNamespaceMap = Object.create(null)
    // 存放订阅者
    this._subscribers = []
    this._makeLocalGettersCache = Object.create(null)

    // bind commit and dispatch to self
    const store = this
    const { dispatch, commit } = this
    // 绑定 dispatch 和 commit 中的 this 为 store
    this.dispatch = function boundDispatch (type, payload) {
      return dispatch.call(store, type, payload)
    }
    this.commit = function boundCommit (type, payload, options) {
      return commit.call(store, type, payload, options)
    }

    // strict mode
    // 严格模式
    this.strict = strict

    const state = this._modules.root.state

    // init root module.
    // this also recursively registers all sub-modules
    // and collects all module getters inside this._wrappedGetters
    // 初始化根 module，递归的注册所有的子 moudle,收集所有的 getters 到 _wrappedGetters
    installModule(this, state, [], this._modules.root)

    // initialize the store state, which is responsible for the reactivity
    // (also registers _wrappedGetters as computed properties)
    // 初始化 store 的状态， 并且吧 _wrappedGetters 注册为计算属性
    resetStoreState(this, state)

    // 应用 plugin 到 store 上面
    plugins.forEach(plugin => plugin(this))

    // devTools 默认为开启状态
    const useDevtools = options.devtools !== undefined ? options.devtools : /* Vue.config.devtools */ true
    if (useDevtools) {
      devtoolPlugin(this)
    }
  }
```

接下来我们来分析一下 `ModuleCollection`入参是上面的例子，子 module 会存放在`this.root._children`中，输出图为：
![moduleCollection](./images/moduleCollection.png)

## 分析 installModule

上文提到了`installModule(this, state, [], this._modules.root)`，其含义为**初始化根 module，递归的注册所有的子 moudle,收集所有的 getters 到 \_wrappedGetters**。

```javascript
function installModule(store, rootState, path, module, hot) {
  // 如果 path 是 [] 则为根根 module
  const isRoot = !path.length
  // 获取命名空间
  const namespace = store._modules.getNamespace(path)

  // register in namespace map
  // 注册命名空间到 store._modulesNamespaceMap 中
  if (module.namespaced) {
    if (store._modulesNamespaceMap[namespace] && __DEV__) {
      console.error(
        `[vuex] duplicate namespace ${namespace} for the namespaced module ${path.join(
          '/'
        )}`
      )
    }
    store._modulesNamespaceMap[namespace] = module
  }

  // set state
  if (!isRoot && !hot) {
    const parentState = getNestedState(rootState, path.slice(0, -1))
    const moduleName = path[path.length - 1]
    store._withCommit(() => {
      //
      parentState[moduleName] = module.state
    })
  }

  const local = (module.context = makeLocalContext(store, namespace, path))

  // 注册 mutation
  module.forEachMutation((mutation, key) => {
    const namespacedType = namespace + key
    registerMutation(store, namespacedType, mutation, local)
  })

  // 注册 action
  module.forEachAction((action, key) => {
    const type = action.root ? key : namespace + key
    const handler = action.handler || action
    registerAction(store, type, handler, local)
  })

  // 注册Getter
  module.forEachGetter((getter, key) => {
    const namespacedType = namespace + key
    registerGetter(store, namespacedType, getter, local)
  })

  // 递归注册 Module
  module.forEachChild((child, key) => {
    installModule(store, rootState, path.concat(key), child, hot)
  })
}
```

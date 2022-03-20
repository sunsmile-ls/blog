## 路由总结

前端路由：

- spa 应用

后端路由：

- 动态路由：

  - 服务器用动态得到的数据，利用相应的模板对页面进行渲染，然后返回渲染完的页面

- 静态路由：

  - 根据 URL 读取服务器资源。

## 前后端路由的优缺点

- 后端路由：

​ 优点：**安全性好，SEO 好**，缺点：**加大服务器的压力，不利于用户体验，代码冗合**

- 前端路由：

优点：**前端路由在访问一个新页面的时候仅仅是变换了一下路径而已，没有了网络延迟，对于用户体验来说会有相当大的提升** 。<br/>
缺点：**使用浏览器的前进，后退键的时候会重新发送请求，没有合理地利用缓存，同样的不利于 seo**

## 前端路由实现方案

- **hash**

- **history Api**

## hash 的方案实现一个路由切换

```html
//首先我们要有个html
<ul>
  <li><a href="#luyou1">路由1</a></li>
  <li><a href="#luyou2">路由2</a></li>
  <li><a href="#luyou3">路由3</a></li>
</ul>
<div id="luyouid"></div>
```

```javascript
//ts逻辑
class router {
  //存贮当前路由
  hashStr: String
  constructor(hash: String) {
    //初始化赋值
    this.hashStr = hash
    //初始化
    this.watchHash()
    //绑定监听事件,由于this被换了，必须用bind绑定
    this.watch = this.watchHash.bind(this)
    window.addEventListener('hashchange', this.watch)
  }
  //监听方法
  watchHash() {
    let hash: String = window.location.hash.slice(1)
    this.hashStr = hash
    console.log(this.hashStr)
    if (this.hashStr) {
      if (this.hashStr == 'luyou1') {
        document.querySelector('#luyouid').innerHTML = 'luyou1'
      } else if (this.hashStr == 'luyou2') {
        document.querySelector('#luyouid').innerHTML = 'luyou2'
      } else {
        document.querySelector('#luyouid').innerHTML = 'default'
      }
    }
  }
}
```

## history Api 的原理

h5 中新增加的 `history.pushState()`和`history.repalceState()` API 可以在不刷新页面的情况下，操作浏览器历史记录。

唯一不同的是，前者是新增一个历史记录，后者是直接替换当前的历史记录

## 监听历史记录

用户点击浏览器后退和前进按钮时，或者使用 js 调用 back、forward、go 方法时才会触发时会触发 `popState` 事件

```javascript
window.addEventListener('popstate', function(event) {})
```

!> 调用`pushState` 方法或 `replaceState` 方法，并不会触发该事件

## 监听 pushState 和 replaceState 的变化

我们可以创建 2 个全新的事件，事件名为 pushState 和 replaceState，我们就可以在全局监听

```javascript
//创建全局事件
var _wr = function(type) {
  var orig = history[type]
  return function() {
    var rv = orig.apply(this, arguments)
    var e = new Event(type)
    e.arguments = arguments
    window.dispatchEvent(e)
    return rv
  }
}
//重写方法
history.pushState = _wr('pushState')
history.replaceState = _wr('replaceState')
//实现监听
window.addEventListener('replaceState', function(e) {
  console.log('THEY DID IT AGAIN! replaceState')
})
window.addEventListener('pushState', function(e) {
  console.log('THEY DID IT AGAIN! pushState')
})
```

## vue-router 源码

1. 实现了用 Router 类去管理路由

```javascript
// router/index.js
import {ref,inject} from 'vue'
import RouterLink from './RouterLink.vue'
import RouterView from './RouterView.vue'

const ROUTER_KEY = '__router__'

// 创建Router对象
function createRouter(options){
    return new Router(options)
}

// 在子组件中获取路由实例
function useRouter(){
    return inject(ROUTER_KEY)
}

function createWebHashHistory(){
    function bindEvents(fn){
        window.addEventListener('hashchange',fn)
    }
    return {
        bindEvents,
        url:window.location.hash.slice(1) || '/'
    }
}

class Router{
    constructor(options){
      	// 在创建Router实例之前，需要先调用createHash
        this.history = options.history
        this.routes = options.routes
        this.current = ref(this.history.url)

        this.history.bindEvents(()=>{
            this.current.value = window.location.hash.slice(1)
        })
    }
  	// 注册到vue实例上
    install(app){
        app.provide(ROUTER_KEY,this)
      	app.component("router-link",RouterLink)
      	app.component("router-view",RouterView)
    }
}

export {createRouter,createWebHashHistory,useRouter}
```

2. 创建`RouterLink`和`RouterView`

```vue
<template>
    <component :is="comp"></component>
</template>
<script setup>

import {computed } from 'vue'
import { useRouter } from '../router/index'

let router = useRouter()

const comp = computed(()=>{
    const route = router.routes.find(
        (route) => route.path === router.current.value
    )
    return route?route.component : null
})
</script>
```

```vue
<template>
    <a :href="'#'+props.to">
        <slot />
    </a>
</template>

<script setup>
import {defineProps} from 'vue'
let props = defineProps({
    to:{type:String,required:true}
})

</script>
```

参考：

[如何监听 URL 的变化](https://juejin.im/post/5c2708cd6fb9a049f06a5744)

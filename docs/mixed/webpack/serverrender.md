## 服务端渲染 (SSR) 是什么？

- 客户端渲染:

  HTML + CSS + JS + Data -> 渲染后的 HTML（客户端渲染是顺序执行的，先加载 html 在进行 css,js,data 等信息的加载）

#### 服务端渲染的优势：

- 服务端渲染 (SSR) 的核⼼是减少请求

- 减少白屏时间
- 对于 SEO 友好

#### SSR 代码实现思路

###### 服务端

- 使用 react-dom/server 的 renderToString 方法将 React 组建渲染成字符串

- 服务端路由返回对应的模板

###### 客户端

- 打包出针对服务端的组建

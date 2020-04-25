####  如何⾃⼰编写⼀个Plugins 

 Plugin: 开始打包，在某个时刻，帮助我们处理⼀些什么事情的机制 

plugin要⽐loader稍微复杂⼀些，在webpack的源码中，⽤plugin的机制还是占有⾮常⼤的场景，可以 说plugin是webpack的灵魂 

* 设计模式 
  * 事件驱动 
  * 发布订阅

 **plugin是⼀个类，⾥⾯包含⼀个apply函数，接受⼀个参数，compiler**  

> 任务：webpack 在文件
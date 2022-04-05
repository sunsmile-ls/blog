## 回顾

通过上一讲，我们可以利用 webgl 绘制出不动的点，缺乏扩展性。本讲我们用 javascript 控制一个点的位置。

## attribute

`javascript` 通过修改 attribute 变量来向顶点着色器传递和顶点相关的数据。

## javascript 修改 attribute

1. 在顶点着色器中声明 attribute 变量

```html
<script id="vertexShader" type="x-shader/x-vertex">
	attribute vec4 a_Position;
	void main(){
	    gl_Position = a_Position;
	    gl_PointSize = 50.0;
	}
</script>
```

- `attribute`: 为存储限定符，类似于 es6 中的`export`
- `vec4`: 变量类型，表示 4 维矢量对象
- `a_Position`: 变量名，指向实际数据的存储位置的指针变量

2. 在 js 中获取 attribute 变量

```js
// gl.getAttribLocation 获取着色器中 attribute 变量方法
const a_Position = gl.getAttribLocation(gl.program, 'a_Position');
```

- js 获取着色器暴露的变量需要通过`程序对象`翻译

!> 翻译的意思就是 gl 上下文对象对 program 程序对象说，你去顶点着色器里找一个名叫'a_Position' 的 attribute 变量

3.  修改 attribute 变量

```js
// gl.vertexAttrib3f() 是改变变量值的方法
// a_Position 为着色器变量
// 顶点的x、y、z位置
gl.vertexAttrib3f(a_Position, 0.0, 0.5, 0.0);
```

#### 扩展

1. **vertexAttrib3f()的同族函数**

一系列修改着色器中的 `attribute` 变量的方法。

```js
gl.vertexAttrib1f(location, v0);
gl.vertexAttrib2f(location, v0, v1);
gl.vertexAttrib3f(location, v0, v1, v2);
gl.vertexAttrib4f(location, v0, v1, v2, v3);
```

他们都可以改变 `attribute` 变量的前 n 个值，比如 vertexAttrib1f() 方法自定一个矢量对象的 v0 值，v1、v2 则默认为 0.0，v3 默认为 1.0，其数值类型为 float 浮点型。

2. **函数的命名规律**

GLSL ES 里函数的命名结构是：<基础函数名><参数个数><参数类型>

以 vertexAttrib3f(location,v0,v1,v2,v3) 为例：

- vertexAttrib：基础函数名
- 3：参数个数，这里的参数个数是要传给变量的参数个数，而不是当前函数的参数个数
- f：参数类型，f 代表 float 浮点类型，除此之外还有 i 代表整型，v 代表数字……

## 鼠标控制点的生成

鼠标控制点的生成，首先面临的问题就是鼠标位置如何转换为 webgl 的坐标

![css](./images/image-20210228111229154.png)

1. 计算坐标的 css 位置

```javascript
canvas.addEventListener('click', function (event) {
	const { clientX, clientY } = event;
	const { left, top } = canvas.getBoundingClientRect();
	const [cssX, cssY] = [clientX - left, clientY - top];
});
```

2. 点击位置基于 webgl 原点的位置

```javascript
// 求画布中心位置
const [halfWidth, halfHeight] = [width / 2, height / 2];
// 鼠标基于画布中心的位置
const [xBaseCenter, yBaseCenter] = [cssX - halfWidth, cssY - halfHeight];
```

3. 解决 y 方向差异

```javascript
// webgl 里的y 轴和canvas 2d 里的y轴相反
const yBaseCenterTop = -yBaseCenter;
```

4. 把 x,y 数据变为 `[-1,1]` 之间的数据

```javascript
const [x, y] = [xBaseCenter / halfWidth, yBaseCenterTop / halfHeight];
```

## 同步绘制

!> `gl.drawArrays()` 方法只会同步绘图，走完了 js 主线程后，再次绘图时，就会从头再来。也就说，异步执行的 `drawArrays()` 方法会把画布上的图像都刷掉。

颜色缓冲区中存储的图像，只在当前线程有效。比如我们先在 js 主线程中绘图，主线程结束后，会再去执行信息队列里的异步线程。在执行异步线程时，颜色缓冲区就会被 webgl 系统重置

- 通过保存数组把一开始的那两个顶点存起来，再次绘制的时候，重新绘制前面的绘制。

## 全部代码

```javascript
import { initShaders } from '../jsm/Utils.js';

const canvas = document.getElementById('canvas');
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;
const gl = canvas.getContext('webgl');
const vsSource = document.getElementById('vertexShader').innerText;
const fsSource = document.getElementById('fragmentShader').innerText;
initShaders(gl, vsSource, fsSource);
const a_Position = gl.getAttribLocation(gl.program, 'a_Position');
gl.clearColor(0.0, 0.0, 0.0, 1.0);
gl.clear(gl.COLOR_BUFFER_BIT);

const g_points = [];
canvas.addEventListener('click', function (event) {
	const { clientX, clientY } = event;
	const { left, top, width, height } = canvas.getBoundingClientRect();
	const [cssX, cssY] = [clientX - left, clientY - top];
	const [halfWidth, halfHeight] = [width / 2, height / 2];
	const [xBaseCenter, yBaseCenter] = [cssX - halfWidth, cssY - halfHeight];
	const yBaseCenterTop = -yBaseCenter;
	const [x, y] = [xBaseCenter / halfWidth, yBaseCenterTop / halfHeight];
	g_points.push({ x, y });
	gl.clear(gl.COLOR_BUFFER_BIT);
	g_points.forEach(({ x, y }) => {
		gl.vertexAttrib2f(a_Position, x, y);
		gl.drawArrays(gl.POINTS, 0, 1);
	});
});
```


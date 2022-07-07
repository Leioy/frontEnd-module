---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
# use UnoCSS (experimental)
css: unocss
---

# 前端模块化


---

# 模块化进程

- 通过函数隐藏内部变量 -> namespace:通过对象封装 -> iife
- AMD和CMD规范(requireJs和seaJs)
- commonJS
- ESModule

<br>

```js
// AMD 规范
define(['Module1', 'Module2'], function (module1, module2) {
    var result1 = module1.exec();
    var result2 = module2.exec();
    return {
      result1: result1,
      result2: result2
    }
});
require(['math'], function (math) {
　 math.sqrt(15)
});

```

---

```js
// CMD 规范
define(function (requie, exports, module) {
    //依赖就近书写
    var module1 = require('Module1');
    var result1 = module1.exec();
    module.exports = {
      result1: result1,
    }
});
```

### AMD和CMD的区别

在依赖的处理上
  - AMD推崇依赖前置，即通过依赖数组的方式提前声明当前模块的依赖
  - CMD推崇依赖就近，在编程需要用到的时候通过调用require方法动态引入

在本模块的对外输出上
  - AMD推崇通过返回值的方式对外输出
  - CMD推崇通过给module.exports赋值的方式对外输出
---

### commonJS

随着NodeJs的发布，以及webpack, browserify等构建工具的诞生，给前端带来了另一种模块组织的方式。
commonJS通过`module.exports=value`或者`exports.xxx=value`来向外部暴露模块，通过require的形式动态加载模块
所以commonJs无法支持tree-shaking

```js
// module.js
 const x = 1
 const add = (a,b) => a+b
 exports.x= x
 exports.add = add

```
```js
// index.js
const module = require('./module')
console.log(module.x) // 1
console.log(module.add(1,2)) //3
```

### ES6模块化

ES6 模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。通过export和import来导出和导入模块。

---

# 自制打包器
早期的浏览器不支持ES6的模块化，所以需要打包成浏览器可执行的代码。

## babel
<br>

### babel原理

1. parse: 把代码转成AST
2. traverse: 遍历AST进行修改
3. transform: 把AST变成输出的代码

示例：见let-to-var分支的代码

### 自动转es5的代码

babel内置了一系列的插件和presets来帮助我们自动生成目标环境的代码

示例： to-es5的代码

---

### 收集依赖

收集依赖是为了后面的打包做准备
示例： deps分支的代码

### 打包

### 打包成一个什么样的文件
```js
var depRelation= [
  {key:'index.js',deps:['a.js','b.js'],code:function...},
  {key:'a.js',deps:['b.js'],code:function...},
  {key:'b.js',deps:['a.js'],code:function...},
]

execute(depRelation[0].key) // 执行入口文件

function execute(key){
  var item = depRelation.find(i => i.key === key)
  item.code() // 执行item的代码，所以code需要是个函数
}
```
示例：运行bundler分支的代码，然后查看输出

---

### 打包css

目前的打包器只能打包js，不能打包css，如何能够打包css？
把css变成js的代码，这样就可以打包css了

示例：见bundler-css的代码

---

# webpack原理

webpack主要的功能是将各种类型的资源，包括图片、js、css等，转译、组合、拼接生成js格式的bundle文件，这个过程的核心是内容转换+资源合并。
包含了三个阶段：
1. 初始化阶段
    1. 初始化参数：读取配置文件、shell中参数，与默认配置结合得出最终的参数
    2. 创建编译器对象：用上一步得到的参数创建compiler对象
    3. 初始化编译环境：注入内置插件、注册各种模块工厂、初始化rule集合
    4. 开始编译：执行compier的run方法
    5. 确定入口：根据entry找出所有的入口文件，转换为dependence对象
2. 构建阶段
    1. 编译模块(make)，根据denpendece创建模块，调用loader转换为标准的js内容，创建AST,根据AST找出该模块的依赖，递归这个步骤直到所有依赖都进行了处理
    2. 完成模块编译，得到每个模块的编译后的内容以及依赖关系图
3. 生成阶段
    1. 输出资源(seal)，根据入口和模块的依赖关系图，组装成一个个包含多个模块的chunk，每个chunk转换成单独的文件加入到输出列表
    2. 写入文件系统(emitAssets)：把文件内容写入到文件系统


---

![webapck.png](https://s2.loli.net/2022/07/06/qstZUV5xkzEfG7y.png)

---

# 其他构建工具

## Parcel 

一个`0`配置的打包工具，开箱即用，同时默认使用Worker进程提升构建速度。在2.0上使用Rust改写了js/css的transformer，进一步提升了构建效率

### 优点

零配置，告别繁琐的工程化配置，能够满足大多数场景。在 JS 和 CSS 的转译上使用了 Rust ，效率上会有所提升。

### 缺点

扩展性不强，几乎没有类似 Webpack 的那种开放性插件特性，因此如果遇到 Parcel 现阶段无法实现或有 Bug 的东西，用户无能为力，只能等 Parcel 去补齐。

---

## Rollup

Rollup 是当前流行的库打包器，它比 Webpack 晚几年出现，也是在 ESM 之后出现的，主打的特点是能够支持并且提倡开发者使用 ESM 模块语法进行开发。Rollup 推崇 ESM 模块标准开发，这个特点也是借助浏览器对 ESM 的支持，Rollup 打包的产物对比 Webpack 会干净很多。
总体而言Rollup是非常优秀的打包工具，产物精简、丰富的插件系统，不过需要支持esm标准的浏览器，所以在打包库方面推荐使用Rollup，反正最终产物也是当成依赖引入

## Snowpack

主打的是 Unbundle，极速的开发体验，在生产环境也同样能依赖 Rollup 打包出产物。推荐Vue团队推出的更成熟的Vite


## Esbuild

使用 Go 语言编写的打包工具，特点就是快,提供两类API: Transform和Build，Transform是用于转换源代码，Build是用于打包产物。
但缺点也很明显，社区生态不够丰富、不能完全转译到ES5的代码等。在业务场景下许多功能是无法覆盖到，比较适合做库的打包

---

## SWC

它并不是一个打包工具，是基于Rust的Compiler工具，后面也许有做Bundle的规划。可以把它看做是babel的替代品，
得益于Rust语言的高效，它的transform效率最高是babel的70倍

## Vite

与 snowpack 类似，他开发阶段采用 unbundle 模式，并且使用 esbuild 做依赖预构建（snowpack 是用的 rollup），生产阶段利用 rollup 做构建。
前面说到Rollup不支持低版本的浏览器，Vite提供了[@vitejs/plugin-legacy]来做低版本的兼容。Vite社区已经很丰富，几乎可以覆盖所有的业务场景。未来很有可能取代Webpack在WebApp方面的地位

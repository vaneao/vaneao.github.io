---
title: webpack 相关面试题
categories:
- 前端
tags:
- webpack
cover: https://tva2.sinaimg.cn/large/00897VxHly8gpspnva4yuj315i0epdkt.jpg
---
文章参考自[当面试官问Webpack的时候他想知道什么](http://mp.weixin.qq.com/s?__biz=MzA4ODUzNTE2Nw==&mid=2451055139&idx=1&sn=82f0d04cda71d288f86ac133c1aa2e59&chksm=87c43b33b0b3b2254e1a39611a8b65c17e5a07497c5551c34a43512c533675866f7b65c0840d&mpshare=1&scene=24&srcid=0420IoE5iC1pnbHdJlLJhBph&sharer_sharetime=1618851250375&sharer_shareid=9696fea7a5d51803a1503d2efcaa547e#rd)，[「吐血整理」再来一打Webpack面试题](https://juejin.cn/post/6844904094281236487#heading-0)，感谢。
## 你知道 webpack 的作用是什么吗？
从官网上的描述我们其实不难理解，`webpack` 的作用其实有以下几点：
- 模块打包。可以将不同模块的文件打包整合在一起，并且保证它们之间的引用正确，执行有序。利用打包我们就可以在开发的时候根据我们自己的业务自由划分文件模块，保证项目结构的清晰和可读性。
- 编译兼容。在前端的“上古时期”，手写一堆浏览器兼容代码一直是令前端工程师头皮发麻的事情，而在今天这个问题被大大的弱化了，通过`webpack`的`Loader`机制，不仅仅可以帮助我们对代码做`polyfill`，还可以编译转换诸如`.less, .vue, .jsx`这类在浏览器无法识别的格式文件，让我们在开发的时候可以使用新特性和新语法做开发，提高开发效率。
- 能力扩展。通过`webpack`的`Plugin`机制，我们在实现模块化打包和编译兼容的基础上，可以进一步实现诸如按需加载，代码压缩等一系列功能，帮助我们进一步提高自动化程度，工程效率以及打包输出的质量。

---
---

## 说一下模块打包运行原理
`Webpack` 实际上为每个模块创造了一个可以导出和导入的环境，本质上并没有修改代码的执行逻辑，代码执行顺序与模块加载顺序也完全一致。

首先我们应该简单了解一下`webpack`的整个打包流程：

1. 读取`webpack`的配置参数；
2. 启动`webpack`，创建`Compiler`对象并开始解析项目；
3. 从入口文件（`entry`）开始解析，并且找到其导入的依赖模块，递归遍历分析，形成依赖关系树；
4. 对不同文件类型的依赖模块文件使用对应的`Loader`进行编译，最终转为`Javascript`文件；
5. 整个过程中`webpack`会通过发布订阅模式，向外抛出一些`hooks`，而`webpack`的插件即可通过监听这些关键的事件节点，执行插件任务进而达到干预输出结果的目的。

其中文件的解析与构建是一个比较复杂的过程，在`webpack`源码中主要依赖于`compiler`和`compilation`两个核心对象实现。

`compiler`对象是一个全局单例，他负责把控整个`webpack`打包的构建流程。
`compilation`对象是每一次构建的上下文对象，它包含了当次构建所需要的所有信息，每次热更新和重新构建，`compiler`都会重新生成一个新的`compilation`对象，负责此次更新的构建过程。

而每个模块间的依赖关系，则依赖于`AST`语法树。每个模块文件在通过`Loader`解析完成之后，会通过`acorn`库生成模块代码的`AST`语法树，通过语法树就可以分析这个模块是否还有依赖的模块，进而继续循环执行下一个模块的编译解析。

最终`Webpack`打包出来的`bundle`文件是一个`IIFE`的执行函数。
```js
// webpack 5 打包的bundle文件内容

(() => { // webpackBootstrap
    var __webpack_modules__ = ({
        'file-A-path': ((modules) => { // ... })
        'index-file-path': ((__unused_webpack_module, __unused_webpack_exports, __webpack_require__) => { // ... })
    })
    
    // The module cache
    var __webpack_module_cache__ = {};
    
    // The require function
    function __webpack_require__(moduleId) {
        // Check if module is in cache
        var cachedModule = __webpack_module_cache__[moduleId];
        if (cachedModule !== undefined) {
                return cachedModule.exports;
        }
        // Create a new module (and put it into the cache)
        var module = __webpack_module_cache__[moduleId] = {
                // no module.id needed
                // no module.loaded needed
                exports: {}
        };

        // Execute the module function
        __webpack_modules__[moduleId](module, module.exports, __webpack_require__);

        // Return the exports of the module
        return module.exports;
    }
    
    // startup
    // Load entry module and return exports
    // This entry module can't be inlined because the eval devtool is used.
    var __webpack_exports__ = __webpack_require__("./src/index.js");
})
```
和`webpack4`相比，`webpack5`打包出来的`bundle`做了相当的精简。在上面的打包`demo`中，整个立即执行函数里边只有三个变量和一个函数方法，`__webpack_modules__`存放了编译后的各个文件模块的`JS`内容，`__webpack_module_cache__`用来做模块缓存，`__webpack_require__`是`Webpack`内部实现的一套依赖引入函数。最后一句则是代码运行的起点，从入口文件开始，启动整个项目。

其中值得一提的是`__webpack_require__`模块引入函数，我们在模块化开发的时候，通常会使用`ES Module`或者`CommonJS`规范导出/引入依赖模块，`webpack`打包编译的时候，会统一替换成自己的`__webpack_require__`来实现模块的引入和导出，从而实现模块缓存机制，以及抹平不同模块规范之间的一些差异性。

---
---

## 你知道sourceMap是什么吗？
提到`sourceMap`，很多小伙伴可能会立刻想到`Webpack`配置里边的`devtool`参数，以及对应的`eval`，`eval-cheap-source-map`等等可选值以及它们的含义。除了知道不同参数之间的区别以及性能上的差异外，我们也可以一起了解一下`sourceMap`的实现方式。

`sourceMap`是一项将编译、打包、压缩后的代码映射回源代码的技术，由于打包压缩后的代码并没有阅读性可言，一旦在开发中报错或者遇到问题，直接在混淆代码中`debug`问题会带来非常糟糕的体验，`sourceMap`可以帮助我们快速定位到源代码的位置，提高我们的开发效率。`sourceMap`其实并不是`Webpack`特有的功能，而是`Webpack`支持`sourceMap`，像`JQuery`也支持`souceMap`。

既然是一种源码的映射，那必然就需要有一份映射的文件，来标记混淆代码里对应的源码的位置，通常这份映射文件以`.map`结尾，里边的数据结构大概长这样：
```js
{
  "version" : 3,                          // Source Map版本
  "file": "out.js",                       // 输出文件（可选）
  "sourceRoot": "",                       // 源文件根目录（可选）
  "sources": ["foo.js", "bar.js"],        // 源文件列表
  "sourcesContent": [null, null],         // 源内容列表（可选，和源文件列表顺序一致）
  "names": ["src", "maps", "are", "fun"], // mappings使用的符号名称列表
  "mappings": "A,AAAB;;ABCDE;"            // 带有编码映射数据的字符串
}
```
其中`mappings`数据有如下规则：
- 生成文件中的一行的每个组用“;”分隔；
- 每一段用“,”分隔；
- 每个段由1、4或5个可变长度字段组成；

有了这份映射文件，我们只需要在我们的压缩代码的最末端加上这句注释，即可让`sourceMap`生效：
```js
//# sourceURL=/path/to/file.js.map
```
有了这段注释后，浏览器就会通过`sourceURL`去获取这份映射文件，通过解释器解析后，实现源码和混淆代码之间的映射。因此`sourceMap`其实也是一项需要浏览器支持的技术。

如果我们仔细查看`webpack`打包出来的`bundle`文件，就可以发现在默认的`development`开发模式下，每个`_webpack_modules__`文件模块的代码最末端，都会加上`//# sourceURL=webpack://file-path?`，从而实现对`sourceMap`的支持。

`sourceMap`映射表的生成有一套较为复杂的规则，有兴趣的小伙伴可以看看以下文章，帮助理解`sourceMap`的原理实现：

[Source Map的原理探究](https://blog.fundebug.com/2018/10/12/understanding_frontend_source_map/)

[Source Maps under the hood – VLQ, Base64 and Yoda](https://docs.microsoft.com/zh-cn/archive/blogs/davidni/source-maps-under-the-hood-vlq-base64-and-yoda)

---
---

## 是否写过Loader？简单描述一下编写loader的思路？
从上面的打包代码我们其实可以知道，`Webpack`最后打包出来的成果是一份`Javascript`代码，实际上在`Webpack`内部默认也只能够处理`JS`模块代码，在打包过程中，会默认把所有遇到的文件都当作 `JavaScript`代码进行解析，因此当项目存在非`JS`类型文件时，我们需要先对其进行必要的转换，才能继续执行打包任务，这也是`Loader`机制存在的意义。

`Loader`的配置使用我们应该已经非常的熟悉：
```js
// webpack.config.js
module.exports = {
  // ...other config
  module: {
    rules: [
      {
        test: /^your-regExp$/,
        use: [
          {
             loader: 'loader-name-A',
          }, 
          {
             loader: 'loader-name-B',
          }
        ]
      },
    ]
  }
}
```
通过配置可以看出，针对每个文件类型，`loader`是支持以数组的形式配置多个的，因此当`Webpack`在转换该文件类型的时候，会按顺序链式调用每一个`loader`，前一个`loader`返回的内容会作为下一个`loader`的入参。因此`loader`的开发需要遵循一些规范，比如返回值必须是标准的`JS`代码字符串，以保证下一个`loader`能够正常工作，同时在开发上需要严格遵循“单一职责”，只关心`loader`的输出以及对应的输出。

`loader`函数中的`this`上下文由`webpack`提供，可以通过`this`对象提供的相关属性，获取当前`loader`需要的各种信息数据，事实上，这个`this`指向了一个叫`loaderContext的loader-runner`特有对象。有兴趣的小伙伴可以自行阅读源码。
```js
module.exports = function(source) {
    const content = doSomeThing2JsString(source);
    
    // 如果 loader 配置了 options 对象，那么this.query将指向 options
    const options = this.query;
    
    // 可以用作解析其他模块路径的上下文
    console.log('this.context');
    
    /*
     * this.callback 参数：
     * error：Error | null，当 loader 出错时向外抛出一个 error
     * content：String | Buffer，经过 loader 编译后需要导出的内容
     * sourceMap：为方便调试生成的编译后内容的 source map
     * ast：本次编译生成的 AST 静态语法树，之后执行的 loader 可以直接使用这个 AST，进而省去重复生成 AST 的过程
     */
    this.callback(null, content);
    // or return content;
}
```
更详细的开发文档可以直接查看官网的 [Loader API](https://www.webpackjs.com/api/loaders/)。

---
---

## 是否写过Plugin？简单描述一下编写plugin的思路？
如果说`Loader`负责文件转换，那么`Plugin`便是负责功能扩展。`Loader`和`Plugin`作为`Webpack`的两个重要组成部分，承担着两部分不同的职责。

上文已经说过，`webpack`基于发布订阅模式，在运行的生命周期中会广播出许多事件，插件通过监听这些事件，就可以在特定的阶段执行自己的插件任务，从而实现自己想要的功能。

既然基于发布订阅模式，那么知道`Webpack`到底提供了哪些事件钩子供插件开发者使用是非常重要的，上文提到过`compiler`和`compilation`是`Webpack`两个非常核心的对象，其中`compiler`暴露了和 `Webpack`整个生命周期相关的钩子（`compiler-hooks`），而`compilation`则暴露了与模块和依赖有关的粒度更小的事件钩子（`Compilation Hooks`）。

`Webpack`的事件机制基于`webpack`自己实现的一套`Tapable`事件流方案（[github](https://github.com/webpack/tapable)）
```js
// Tapable的简单使用
const { SyncHook } = require("tapable");

class Car {
    constructor() {
        // 在this.hooks中定义所有的钩子事件
        this.hooks = {
            accelerate: new SyncHook(["newSpeed"]),
            brake: new SyncHook(),
            calculateRoutes: new AsyncParallelHook(["source", "target", "routesList"])
        };
    }

    /* ... */
}

const myCar = new Car();
// 通过调用tap方法即可增加一个消费者，订阅对应的钩子事件了
myCar.hooks.brake.tap("WarningLampPlugin", () => warningLamp.on());
```
`Plugin`的开发和开发`Loader`一样，需要遵循一些开发上的规范和原则：
- 插件必须是一个函数或者是一个包含 `apply` 方法的对象，这样才能访问`compiler`实例；
- 传给每个插件的 `compiler` 和 `compilation` 对象都是同一个引用，若在一个插件中修改了它们身上的属性，会影响后面的插件;
- 异步的事件需要在插件处理完任务时调用回调函数通知 `Webpack` 进入下一个流程，不然会卡住;

了解了以上这些内容，想要开发一个 `Webpack Plugin`，其实也并不困难。
```js
class MyPlugin {
  apply (compiler) {
    // 找到合适的事件钩子，实现自己的插件功能
    compiler.hooks.emit.tap('MyPlugin', compilation => {
        // compilation: 当前打包构建流程的上下文
        console.log(compilation);
        
        // do something...
    })
  }
}
```
更详细的开发文档可以直接查看官网的 [Plugin API](https://www.webpackjs.com/api/plugins/)。

---
---

## 有哪些常见的Loader？你用过哪些Loader？
- `raw-loader`：加载文件原始内容（`utf-8`）；
- `file-loader`：把文件输出到一个文件夹中，在代码中通过相对 `URL` 去引用输出的文件 (处理图片和字体)；
- `url-loader`：与 `file-loader` 类似，区别是用户可以设置一个阈值，大于阈值会交给`file-loader` 处理，小于阈值时返回文件 `base64` 形式编码 (处理图片和字体)；
- `source-map-loader`：加载额外的 `Source Map` 文件，以方便断点调试；
- `svg-inline-loader`：将压缩后的 `SVG` 内容注入代码中；
- `image-loader`：加载并且压缩图片文件；
- `json-loader`：加载 `JSON` 文件（默认包含）；
- `handlebars-loader`: 将 `Handlebars` 模版编译成函数并返回；
- `babel-loader`：把 `ES6` 转换成 `ES5`；
- `ts-loader`: 将 `TypeScript` 转换成 `JavaScript`；
- `awesome-typescript-loader`：将 `TypeScript` 转换成 `JavaScript`，性能优于 `ts-loader`；
- `sass-loader`：将`SCSS/SASS`代码转换成`CSS`；
- `css-loader`：加载 `CSS`，支持模块化、压缩、文件导入等特性；
- `style-loader`：把 `CSS` 代码注入到 `JavaScript` 中，通过 `DOM` 操作去加载 `CSS`；
- `postcss-loader`：扩展 `CSS` 语法，使用下一代 `CSS`，可以配合 `autoprefixer` 插件自动补齐 `CSS3` 前缀；
- `eslint-loader`：通过 `ESLint` 检查 `JavaScript` 代码；
- `tslint-loader`：通过 `TSLint` 检查 `TypeScript` 代码；
- `mocha-loader`：加载 `Mocha` 测试用例的代码；
- `coverjs-loader`：计算测试的覆盖率；
- `vue-loader`：加载 `Vue.js` 单文件组件；
- `i18n-loader`: 国际化；
- `cache-loader`: 可以在一些性能开销较大的 `Loader` 之前添加，目的是将结果缓存到磁盘里；

更多 [Loader](https://webpack.docschina.org/loaders/) 请参考官网。

---
---

## 有哪些常见的Plugin？你用过哪些Plugin？
- `define-plugin`：定义环境变量 (`Webpack4` 之后指定 `mode` 会自动配置)；
- `ignore-plugin`：忽略部分文件；
- `html-webpack-plugin`：简化 `HTML` 文件创建 (依赖于 `html-loader`)；
- `web-webpack-plugin`：可方便地为单页应用输出 `HTML`，比 `html-webpack-plugin` 好用；
- `uglifyjs-webpack-plugin`：不支持 `ES6` 压缩 (`Webpack4` 以前)；
- `terser-webpack-plugin`: 支持压缩 `ES6 (Webpack4)`；
- `webpack-parallel-uglify-plugin`: 多进程执行代码压缩，提升构建速度；
- `mini-css-extract-plugin`: 分离样式文件，`CSS` 提取为独立文件，支持按需加载 (替代`extract-text-webpack-plugin`)；
- `serviceworker-webpack-plugin`：为网页应用增加离线缓存功能；
- `clean-webpack-plugin`: 目录清理；
- `ModuleConcatenationPlugin`: 开启 `Scope Hoisting`；
- `speed-measure-webpack-plugin`: 可以看到每个 `Loader` 和 `Plugin` 执行耗时 (整个打包耗时、每个 `Plugin` 和 `Loader` 耗时)；
- `webpack-bundle-analyzer`: 可视化 `Webpack` 输出文件的体积 (业务组件、依赖第三方模块)；

更多 [Plugin](https://webpack.docschina.org/plugins/) 请参考官网。

---
---

## Loader和Plugin的区别？
- `Loader` 本质就是一个函数，在该函数中对接收到的内容进行转换，返回转换后的结果。
因为 `Webpack` 只认识 `JavaScript`，所以 `Loader` 就成了翻译官，对其他类型的资源进行转译的预处理工作。

- `Plugin` 就是插件，基于事件流框架 `Tapable`，插件可以扩展 `Webpack` 的功能，在 `Webpack` 运行的生命周期中会广播出许多事件，`Plugin` 可以监听这些事件，在合适的时机通过 `Webpack` 提供的 `API` 改变输出结果。

- `Loader` 在 `module.rules` 中配置，作为模块的解析规则，类型为数组。每一项都是一个 `Object`，内部包含了 `test`(类型文件)、`loader`、`options` (参数)等属性。
`Plugin` 在 `plugins` 中单独配置，类型为数组，每一项是一个 `Plugin` 的实例，参数都通过构造函数传入。

---
---

## Webpack构建流程
Webpack 的运行流程是一个串行的过程，从启动到结束会依次执行以下流程：
- `初始化参数`：从配置文件和 `Shell` 语句中读取与合并参数，得出最终的参数。
- `开始编译`：用上一步得到的参数初始化 `Compiler` 对象，加载所有配置的插件，执行对象的 `run` 方法开始执行编译。
- `确定入口`：根据配置中的 `entry` 找出所有的入口文件。
- `编译模块`：从入口文件出发，调用所有配置的 `Loader` 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理。
- `完成模块编译`：在经过第 4 步使用 `Loader` 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系。
- `输出资源`：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 `Chunk`，再把每个 `Chunk` 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会。
- `输出完成`：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统。

在以上过程中，`Webpack` 会在特定的时间点广播出特定的事件，插件在监听到感兴趣的事件后会执行特定的逻辑，并且插件可以调用 `Webpack` 提供的 `API` 改变 `Webpack` 的运行结果。

简单说：

- `初始化`：启动构建，读取与合并配置参数，加载 `Plugin`，实例化 `Compiler`。
- `编译`：从 `Entry` 出发，针对每个 `Module` 串行调用对应的 `Loader` 去翻译文件的内容，再找到该 `Module` 依赖的 `Module`，递归地进行编译处理。
- `输出`：将编译后的 `Module` 组合成 `Chunk`，将 `Chunk` 转换成文件，输出到文件系统中。

对源码感兴趣的同学可以移步我的另一篇专栏从源码[窥探Webpack4.x原理](https://juejin.cn/post/6844904046294204429)。

---
---

## 使用webpack开发时，你用过哪些可以提高效率的插件？
- `webpack-dashboard`：可以更友好的展示相关打包信息。
- `webpack-merge`：提取公共配置，减少重复配置代码。
- `speed-measure-webpack-plugin`：简称 `SMP`，分析出 `Webpack` 打包过程中 `Loader` 和 `Plugin` 的耗时，有助于找到构建过程中的性能瓶颈。
- `size-plugin`：监控资源体积变化，尽早发现问题。
- `HotModuleReplacementPlugin`：模块热替换。

## 文件监听原理？
在发现源码发生变化时，自动重新构建出新的输出文件。

`Webpack`开启监听模式，有两种方式：

- 启动 `webpack` 命令时，带上 `--watch` 参数。
- 在配置 `webpack.config.js` 中设置 `watch:true`。

缺点：每次需要手动刷新浏览器。

原理：轮询判断文件的最后编辑时间是否变化，如果某个文件发生了变化，并不会立刻告诉监听者，而是先缓存起来，等 `aggregateTimeout` 后再执行。
```js
module.export = {    
    // 默认false,也就是不开启    
    watch: true,    
    // 只有开启监听模式时，watchOptions才有意义    
    watchOptions: {        
        // 默认为空，不监听的文件或者文件夹，支持正则匹配        
        ignored: /node_modules/,        
        // 监听到变化发生后会等300ms再去执行，默认300ms 
        aggregateTimeout:300,        
        // 判断文件是否发生变化是通过不停询问系统指定文件有没有变化实现的，默认每秒问1000次        
        poll:1000    
    }
}
```

---
---

## Webpack 热更新原理
`Webpack` 的热更新又称热替换（`Hot Module Replacement`），缩写为 `HMR`。 这个机制可以做到不用刷新浏览器而将新变更的模块替换掉旧的模块。

`HMR` 的核心就是客户端从服务端拉去更新后的文件，准确的说是 `chunk diff` （`chunk` 需要更新的部分），实际上 `WDS` 与浏览器之间维护了一个 `Websocket`，当本地资源发生变化时，`WDS` 会向浏览器推送更新，并带上构建时的 `hash`，让客户端与上一次资源进行对比。客户端对比出差异后会向 `WDS` 发起 `Ajax` 请求来获取更改内容（文件列表、`hash`），这样客户端就可以再借助这些信息继续向 `WDS` 发起 `jsonp` 请求获取该 `chunk` 的增量更新。

后续的部分(拿到增量更新之后如何处理？哪些状态该保留？哪些又需要更新？)由   `HotModulePlugin` 来完成，提供了相关 `API` 以供开发者针对自身场景进行处理，像 `react-hot-loader` 和 `vue-loader` 都是借助这些 `API` 实现 `HMR`。

细节请参考 [Webpack HMR 原理解析](https://zhuanlan.zhihu.com/p/30669007)。

---
---

## 如何对bundle体积进行监控和分析？
`VSCode` 中有一个插件 `Import Cost` 可以帮助我们对引入模块的大小进行实时监测，还可以使用 `webpack-bundle-analyzer` 生成 `bundle` 的模块组成图，显示所占体积。

`bundlesize` 工具包可以进行自动化资源体积监控。

---
---

## 文件指纹是什么？怎么用？
文件指纹是打包后输出的文件名的后缀。

- `Hash`：和整个项目的构建相关，只要项目文件有修改，整个项目构建的 `hash` 值就会更改。
- `Chunkhash`：和 `Webpack` 打包的 `chunk` 有关，不同的 `entry` 会生出不同的 `chunkhash`。
- `Contenthash`：根据文件内容来定义 `hash`，文件内容不变，则 `contenthash` 不变。

### JS的文件指纹设置
设置 `output` 的 `filename`，用 `chunkhash`。
```js
module.exports = {    
    entry: {        
        app: './scr/app.js',        
        search: './src/search.js'    
    },   
    output: {        
        filename: '[name][chunkhash:8].js',        
        path:__dirname + '/dist'    
    }
}
```
### CSS的文件指纹设置
设置 `MiniCssExtractPlugin` 的 `filename`，使用 `contenthash`。
```js
module.exports = {    
    entry: {        
        app: './scr/app.js',        
        search: './src/search.js'    
    },   
    output: {        
        filename: '[name][chunkhash:8].js',        
        path:__dirname + '/dist'    
    },    
    plugins:[        
        new MiniCssExtractPlugin({            
            filename: `[name][contenthash:8].css`        
        })    
    ]
}
```
### 图片的文件指纹设置
设置 `file-loader` 的 `name`，使用 `hash`。

占位符名称及含义:
- `ext` 资源后缀名
- `name` 文件名称
- `path` 文件的相对路径
- `folder` 文件所在的文件夹
- `contenthash` 文件的内容 `hash`，默认是 `md5` 生成
- `hash` 文件内容的 `hash`，默认是 `md5` 生成
- `emoji` 一个随机的指代文件内容的 `emoji`
```js
const path = require('path');
module.exports = {   
    entry: './src/index.js',    
    output: {        
        filename:'bundle.js',        
        path:path.resolve(__dirname, 'dist')    
    },    
    module:{        
        rules:[{            
            test:/\.(png|svg|jpg|gif)$/,            
            use:[{                
                loader:'file-loader',                
                options:{                    
                    name:'img/[name][hash:8].[ext]'      
                }            
            }]        
        }]    
    }
}
```

---
---

## 在实际工程中，配置文件上百行乃是常事，如何保证各个loader按照预想方式工作？
可以使用 `enforce` 强制执行 `loader` 的作用顺序，`pre` 代表在所有正常 `loader` 之前执行，`post` 是所有 `loader` 之后执行。(`inline` 官方不推荐使用)

---
---

## 如何优化 Webpack 的构建速度？
- 使用 `高版本` 的 `Webpack` 和 `Node.js`。
- 多进程/多实例构建：HappyPack(不维护了)、`thread-loader`。
- 压缩代码
    - 多进程并行压缩：
        - webpack-paralle-uglify-plugin
        - uglifyjs-webpack-plugin 开启 parallel 参数 (不支持ES6)
        - terser-webpack-plugin 开启 parallel 参数
    - 通过 `mini-css-extract-plugin` 提取 `Chunk` 中的 `CSS` 代码到单独文件，通过 `css-loader` 的 `minimize` 选项开启 `cssnano` 压缩 `CSS`。
- 图片压缩
    - 使用基于 `Node` 库的 `imagemin`（很多定制选项、可以处理多种图片格式）。
    - 配置 `image-webpack-loader`。
- 缩小打包作用域
    - `exclude/include` (确定 `loader` 规则范围)
    - `resolve.modules` 指明第三方模块的绝对路径 (减少不必要的查找)
    - `resolve.mainFields` 只采用 `main` 字段作为入口文件描述字段 (减少搜索步骤，需要考虑到所有运行时依赖的第三方模块的入口文件描述字段)
    - `resolve.extensions` 尽可能减少后缀尝试的可能性
    - `noParse` 对完全不需要解析的库进行忽略 (不去解析但仍会打包到 `bundle` 中，注意被忽略掉的文件里不应该包含 `import、require、define` 等模块化语句)
    - `IgnorePlugin` (完全排除模块)
    - 合理使用 `alias`
- 提取页面公共资源
    - 基础包分离
        - 使用 `html-webpack-externals-plugin`，将基础包通过 `CDN` 引入，不打入 `bundle` 中
        - 使用 `SplitChunksPlugin` 进行(公共脚本、基础包、页面公共文件)分离(`Webpack4`内置) ，替代了 `CommonsChunkPlugin` 插件
- DLL
    - 使用 `DllPlugin` 进行分包，使用 `DllReferencePlugin`(索引链接) 对 `manifest.json` 引用，让一些基本不会改动的代码先打包成静态资源，避免反复编译浪费时间。
    - `HashedModuleIdsPlugin` 可以解决模块数字 `id` 问题。
- 充分利用缓存提升二次构建速度
    - `babel-loader` 开启缓存
    - `terser-webpack-plugin` 开启缓存
    - 使用 `cache-loader` 或者 `hard-source-webpack-plugin`
- Tree shaking
    - 打包过程中检测工程中没有引用过的模块并进行标记，在资源压缩时将它们从最终的`bundle`中去掉(只能对`ES6 Modlue`生效) 开发中尽可能使用`ES6 Module`的模块，提高`tree shaking`效率。
    - 禁用 `babel-loader` 的模块依赖解析，否则 `Webpack` 接收到的就都是转换过的 `CommonJS` 形式的模块，无法进行 `tree-shaking`。
    - 使用 `PurifyCSS`(不在维护) 或者 `uncss` 去除无用 `CSS` 代码。
        - `purgecss-webpack-plugin` 和 `mini-css-extract-plugin` 配合使用(建议)
- Scope hoisting
    - 构建后的代码会存在大量闭包，造成体积增大，运行代码时创建的函数作用域变多，内存开销变大。`Scope hoisting` 将所有模块的代码按照引用顺序放在一个函数作用域里，然后适当的重命名一些变量以防止变量名冲突。
    - 必须是 `ES6` 的语法，因为有很多第三方库仍采用 `CommonJS` 语法，为了充分发挥 `Scope hoisting` 的作用，需要配置 `mainFields` 对第三方模块优先采用 `jsnext:main` 中指向的 `ES6` 模块化语法。
- 动态 Polyfill
    - 建议采用 `polyfill-service` 只给用户返回需要的 `polyfill`，社区维护。 (部分国内奇葩浏览器UA可能无法识别，但可以降级返回所需全部`polyfill`)
更多优化请参考[官网-构建性能](https://www.webpackjs.com/guides/build-performance/)。

---
---

## 代码分割的本质是什么？有什么意义呢？
代码分割的本质其实就是在 `源代码直接上线` 和 `打包成唯一脚本main.bundle.js` 这两种极端方案之间的一种更适合实际场景的中间状态。

阿卡丽：荣耀剑下取，均衡乱中求

**「用可接受的服务器性能压力增加来换取更好的用户体验。」**

源代码直接上线：虽然过程可控，但是http请求多，性能开销大。

打包成唯一脚本：一把梭完自己爽，服务器压力小，但是页面空白期长，用户体验不好。

(Easy peezy right)

---
---

## 聊一聊Babel原理吧
大多数 `JavaScript Parser` 遵循 `estree` 规范，`Babel` 最初基于 `acorn`项目(轻量级现代 `JavaScript` 解析器) `Babel` 大概分为三大部分：
- 解析：将代码转换成 `AST`。
    - 词法分析：将代码(字符串)分割为 `token` 流，即语法单元成的数组。
    - 语法分析：分析 `token` 流(上面生成的数组)并生成 `AST`。
- 转换：访问 `AST` 的节点进行变换操作生产新的 `AST`。
    - `Taro` 就是利用 `babel` 完成的小程序语法转换。
- 生成：以新的 `AST` 为基础生成代码。
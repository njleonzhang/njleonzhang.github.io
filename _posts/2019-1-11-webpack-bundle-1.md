---
layout: post
title: webpack 输出文件分析 1 - 同步引入
date: 2018-12-30
categories: 前端
cover: https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/4a30ce00-c8ef-f60a-9de8-c548cde581a0.png
tags: webpack
---

webpack 是一个神奇的工具, 他大大的提高了前端开发的便利度。使用 webpack 后, 我们可以模块化的组织前端代码, 还能实现运行时的按需加载, 那么 webpack 是怎么做到的呢？本文通过研究 webpack 的输出文件来探讨同步引入的问题。先开个头，挖个坑。

* webpack 版本: 4.28.3
* 本文示例代码: [deep-into-webpack-output-sample/output](https://github.com/njleonzhang/deep-into-webpack-output-sample/tree/master/output)

# 一个简单的例子

我们来看一个简单的例子:

```
// const.js
export let name = 'leon'
export default function print() { console.log('print: ' + name) }
```

```
// index.js
import print, { name } from './const'

console.log(name)
print()
```

我们用最简单的 webpack 配置来编译这段代码:

```js
exports.default = {
  entry: './index.js',
  mode: 'development',
  devtool: 'cheap-module', // 为了让打包后的模块更好阅读
  output: {
    filename: 'index.dist.js',
    path: path.resolve(__dirname, './dist/'),
  },
}
```

编译的完整结果为: [index.dist.js](https://github.com/njleonzhang/deep-into-webpack-output-sample/blob/master/output/dist/index.dist.js), 我们来慢慢分析。

# 输出文件的结构
这段输出代码，大架构上就是一个自调函数, 函数体的内容里是 webpack 的启动代码, 函数的参数是各模块的代码组成的一个对象。

```js
/******/ (function(modules) {
/******/   // webpack 启动代码
/******/ })
/************************************************************************/
(
  // 各模块的代码
  {
    "./const.js": (function(module, __webpack_exports__, __webpack_require__) {
      /*const.js代码...*/
    }),
    "./index.js": (function(module, __webpack_exports__, __webpack_require__) {
      /*index.js代码...*/
    })
  }
)
```

从注释我们也可以看到这段输出的代码泾渭分明。启动代码的每一行前面都有一段占位注释 `/******/`, 启动代码结束后, 接着一长串星号, 再下面就是各模块的代码了。
所有的模块的结构都是一致的, 为一个函数:

```
function(module, __webpack_exports__, __webpack_require__) {
  // 组件代码
},
```

# 启动代码
我们细细分析一下这段启动代码, 其逻辑如下:

1. 定义一个变量 `installedModules`, 用于缓存模块。
```
var installedModules = {};
```

2. 定义函数 `__webpack_require__` 参数为模块 id: `moduleId`, 用于加载和缓存模块。所谓的加载模块，实际上就是执行我们上一节提到的那个模块的函数。

    ```js
    // The require function
    function __webpack_require__(moduleId) {

      // Check if module is in cache
      // 检查模块是否已经加载了，有则直接返回模块的导出
      if(installedModules[moduleId]) {
        return installedModules[moduleId].exports;
      }
      // Create a new module (and put it into the cache)
      // 没有加载，则创建模块
      var module = installedModules[moduleId] = {
        i: moduleId,
        l: false,
        exports: {}
      };

      // Execute the module function
      // 执行模块函数
      modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);

      // Flag the module as loaded
      // 设置模块标记为已加载
      module.l = true;

      // Return the exports of the module
      // 返回模块的导出
      return module.exports;
    }
    ```

    所有加载了的模块，都缓存于 `installedModules` 中，其结构为:

    ```ts
    {
      [i: string]: {      // key 是模块 id
        i: string         // 模块 id; i 就代表 id
        l: boolean        // 是否初始化完成; l 代表 loaded
        exports: any   // 模块的导出
      }
    }
    ```

3. 接着定义了一大批工具函数和变量

    ```js
    // expose the modules object (__webpack_modules__)
    // 把所有的模块都挂载到 __webpack_require__ 的 m 属性上
    __webpack_require__.m = modules;

    // expose the module cache
    // 把所有已加载的模块都挂载到 __webpack_require__ 的 c 属性上
    __webpack_require__.c = installedModules;

    // Object.prototype.hasOwnProperty.call
    // 工具函数, 判断对象上是否有某个属性
    __webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };

    // define getter function for harmony exports
    // 工具函数, 为对象(模块的 exports )的属性添加一个 getter 方法，实际上就是用于定义模块的导出属性
    __webpack_require__.d = function(exports, name, getter) {
      if(!__webpack_require__.o(exports, name)) {
        Object.defineProperty(exports, name, { enumerable: true, get: getter });
      }
    };

    // define __esModule on exports
    // 工具方法，为模块的 exports 定义 __esModule 的标记
    __webpack_require__.r = function(exports) {
      if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
        Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
      }
      Object.defineProperty(exports, '__esModule', { value: true });
    };

    // create a fake namespace object
    // mode & 1: value is a module id, require it
    // mode & 2: merge all properties of value into the ns
    // mode & 4: return value when already ns object
    // mode & 8|1: behave like require
    __webpack_require__.t = function(value, mode) {
      if(mode & 1) value = __webpack_require__(value);
      if(mode & 8) return value;
      if((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
      var ns = Object.create(null);
      __webpack_require__.r(ns);
      Object.defineProperty(ns, 'default', { enumerable: true, value: value });
      if(mode & 2 && typeof value != 'string') for(var key in value) __webpack_require__.d(ns, key, function(key) { return value[key]; }.bind(null, key));
      return ns;
    };

    // getDefaultExport function for compatibility with non-harmony modules
    __webpack_require__.n = function(module) {
      var getter = module && module.__esModule ?
        function getDefault() { return module['default']; } :
        function getModuleExports() { return module; };
      __webpack_require__.d(getter, 'a', getter);
      return getter;
    };
    ```

4. 最后加载 entry 模块(webpack配置里的个 entry)，并返回其模块导出，我们写的模块代码这才正式被执行。

    ```
    return __webpack_require__(__webpack_require__.s = "./index.js");
    ```

# 模块代码
1. entry 模块, index.js

    `index.js` 模块的这个函数，在 `__webpack_require__` 里被执行。

    ```js
    modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
    ```

    ```js
    (function(module, __webpack_exports__, __webpack_require__) {
      "use strict";
      // 为模块的 exports 定义 __esModule 属性。
      __webpack_require__.r(__webpack_exports__);

      // 用 __webpack_require 去加载 const.js 模块
      /* harmony import */ var _const__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./const */ "./const.js");

      // 打印出模块的导出（module.exports）的 name 属性, 执行 default
      // 属性对应的函数，即 print。
      // 这 2 行对应我们写的代码。
      console.log(_const__WEBPACK_IMPORTED_MODULE_0__["name"])
      Object(_const__WEBPACK_IMPORTED_MODULE_0__["default"])()
    /***/ }),
    ```

    > 这里 `_const__WEBPACK_IMPORTED_MODULE_0__["default"]` 就是 `print` 函数, webpack 生成的代码里为什么加个 Object() 包一下呢？是为了兼容什么特殊场景么？

2. const.js 模块

    ```js
    /***/ (function(module, __webpack_exports__, __webpack_require__) {
    "use strict";

    // 为模块的 exports 定义 __esModules 属性
    __webpack_require__.r(__webpack_exports__);

    // 为模块定义导出属性。有个 name 的导出属性，其 get 函数为 return name
    // 有个 default 的属性，其 get 函数 return print
    /* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "name", function() { return name; });
    /* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "default", function() { return print; });

    let name = 'leon'
    function print() { console.log('print: ' + name) }
    /***/ }),
    ```

逻辑还是很容易理解的，我们写的模块被 webpack 的处理成了能兼容 es5 语法里的模块对象，并在 webpack 启动代码构建的环境里有序的运行。最后我们来看一眼我们模块在 webpack 模块缓存 installedModules 里的样子，应该和你的想象一致吧。

![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/5907616e-1ab8-52f8-3ad8-f47e0e7499fc.png)

![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/83a5a315-4694-2811-bdfc-88afb5ba8c58.png)

# 结语
本文中我们通过一个简单的例子分析了 webpack 启动代码的一部分内容，例子虽然简单，但是窥斑见豹，万变不离其宗。当然还有一些内容比如 `__webpack_require__.t` 和 `__webpack_require__.n`，我们并未涉及, 后续文章中我们继续分析。

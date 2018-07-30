---
layout: post
title: 搬家 - karma如何与测试框架合作2之webpack
date: 2017-09-26
categories: test
tags: karma mocha
---

[前文中](https://njleonzhang.github.io/2017/09/16/karma1.html)，我们探讨了karma的工作的一些基本过程。但是，我并没有提及es6相关的内容, 本文来填这个坑。

# 抛出疑问
在现有的karma + webpack项目里，我们经常看到这样的配置，以[vue-webpack](https://github.com/vuejs-templates/webpack/blob/master/template/test/unit/index.js)生成的代码为例：

```js
    // index.js

    // require all test files (files that ends with .spec.js)
    // const testsContext = require.context('./specs/serverVersion/', true, /tableAction.spec$/)
    // const testsContext = require.context('./specs/clientVersion/', true, /\.spec$/)
    const testsContext = require.context('./specs', true, /\.spec$/)
    testsContext.keys().forEach(testsContext)

    // require all src files except main.js for coverage.
    // you can also change this to match only the subset of files that
    // you want coverage for.
    const srcContext = require.context('../../src', true, /^\.\/(?!main(\.js)?$)/)
    srcContext.keys().forEach(srcContext)
```

虽然有注释，但是在深入研究之前我一直没有搞明白过，这段代码是什么意思？！！！`vue`组件、`es6`这些代码又是怎么被加载的呢？下面让我们慢慢揭秘。

> 本文代码： https://github.com/njleonzhang/karma-study/tree/master/karma-3

# 还是从简单的例子出发
我们把上一篇[文章][1]的例子加工一下

```js
// src/index.js

function add(x, y) {
  return x + y
}

function sub(x, y) {
  return x - y + 1
}

export {
  add,
  sub
}
```

```js
// test/add.spc.js

import {add} from '../src'

describe('add suit', _ => {
  it('test add', done => {
    add(1, 2).should.equal(3)
    done()
  })
})
```

```js
// test/sub.spc.js

import {sub} from '../src'

describe('add suit', _ => {
  it('test sub', done => {
    sub(5, 2).should.equal(3)
    done()
  })
})
```

```js
// karma.conf.js

{
    files: [
      'test/add.spec.js',
      'test/sub.spec.js'
    ],
}
```

这里我们把`add`和`sub`两个函数通过es6导出了，测试代码我们也不再自己写karma adapter了，而是用上了大家熟悉的`mocha`。执行一下`npm run test`试试，挂了：
```
Uncaught SyntaxError: Unexpected token import
```

回顾[上文][1]的内容, karma会把所有karma.config里`files`字段里描述的所有文件都加载到测试页面里执行. 也就是说`import`和`export`这样的语法，被直接加载到浏览器里面了，出错也就难免了。怎么办呢？常规方案啊：用`webpack`把es6编译成es5，再执行嘛。

# karma-webpack
`karma`和`webpack`是通过`karma-webpack`这个`karma preprocess`结合在一起的。至于什么是`preprocess`，顾名思义就是预处理器，在karma加载`files`到页面之前，karma会先对`files`通过`preprocess`进行处理。

我们来配置一下：

```js
// karma.conf.js
    files: [
      'test/add.spec.js',
      'test/sub.spec.js'
    ],

    // preprocess matching files before serving them to the browser
    // available preprocessors: https://npmjs.org/browse/keyword/karma-preprocessor
    preprocessors: {
      'test/add.spec.js': ['webpack'],
      'test/sub.spec.js': ['webpack']
    },

    webpack: {
      module: {
        loaders: [{
          test: /\.js$/,
          exclude: [/node_modules/],
          loader: "babel-loader"
        }]
      }
    }
```

我们用`babel-loader`先对测试文件预处理一下。再执行下测试试试：

```
PhantomJS 2.1.1 (Mac OS X 0.0.0): Executed 2 of 2 SUCCESS (0.008 secs / 0.001 secs)
```
果然成功了。打开浏览器看一下`add.spec.js`, 确实是被`webpack`处理过

```js
// 浏览器中的add.spec.js

/******/ (function(modules) { // webpackBootstrap
/******/ 	// The module cache
/******/ 	var installedModules = {};
/******/
/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {
      .....
/***/ (function(module, exports, __webpack_require__) {

"use strict";


var _src = __webpack_require__(1);

describe('test suit', function (_) {
  it('test add', function (done) {
    (0, _src.add)(1, 2).should.equal(3);
    done();
  });
});

/***/ }),
/* 1 */
/***/ (function(module, exports, __webpack_require__) {

"use strict";


Object.defineProperty(exports, "__esModule", {
  value: true
});
function add(x, y) {
  return x + y;
}

function sub(x, y) {
  return x - y;
}

exports.add = add;
exports.sub = sub;

/***/ })
/******/ ]);
```

到这是不是就配置好了呢？当然不是啦，文章开头的提到的`index.js`里的代码是什么意思的问题还没解释呢。仔细看一下`karma-wepack`的文档[Alternative Usage](https://github.com/webpack-contrib/karma-webpack#alternative-usage)部分：

> This configuration is more performant, but you cannot run single test anymore (only the complete suite).
> The above configuration generates a webpack bundle for each test. For many testcases this can result in many big files. The alterative configuration creates a single bundle with all testcases.

再细看一下上面中贴出的`浏览器中的add.spec.js`的代码，我们会发现`add`和`sub`的代码都包含在了里面。同样`浏览器中的sub.spec.js`的代码，也会包含`add`和`sub`的代码。实际上，`karma-wepack`会把`files`里的每一个文件都当做webpack entry去编译一次，这样如果我们有多个测试文件，就会多次打包，不但测试速度慢了，而且打包可能会重复多次，以至于包变大。

所以，`karma-wepack`向我们推荐使用文章开头提到的配置方式，就是把所有的测试代码都通过一个`index`文件加载起来。这里利用的是webpack的[require.context](https://webpack.js.org/guides/dependency-management/#require-context)和[context module API](https://webpack.js.org/guides/dependency-management/#context-module-api)，大家自行看一下文档就明白了。

再次优化一下我们的实例代码，添加一个`test/index.js`文件，并修改`karma.conf.js`的配置：

```js
// karma.conf.js
    // list of files / patterns to load in the browser
    files: [
      'index.js'
    ],

    // preprocess matching files before serving them to the browser
    // available preprocessors: https://npmjs.org/browse/keyword/karma-preprocessor
    preprocessors: {
      'index.js': ['webpack']
    },
```

```js
// test/index.js
var testsContext = require.context("./test", true, /\.spec$/);
testsContext.keys().forEach(testsContext);
```
再跑一下测试，观察一下，src和test里的代码确实都打到了一个文件里了！！

问题基本上都搞清楚了，但是细心的朋友可能发现了文章开头贴出的代码里，不仅仅require了所有的测试代码，还把`src`下的代码require了一遍？本例中`src`下的代码都都通过import的方式引用了，webpack自然也就把它们打包了。这样对测试覆盖率的统计不会有什么影响。但是，如果我们给一个库写测试用例，这个库有很多的组件，我们很有可能丢掉某个组件忘记写测试用例，这个时候如果我们不把`src`下的代码也加载进来，那么测试覆盖率的统计就会丢掉这部分代码，所以require `src`下的代码有时候也是必要的。(这个问题，可以参考[require all src files in unit/index.js useless?](https://github.com/vuejs-templates/webpack/issues/928)

# 结束语
我们接触计算机的技术的时候，一般都是先用起来，然后再深入掌握。本文是我个人在使用karma + webpack时对一些细节问题的探索,我把详细的思考过程记录在这里，希望对后来者所有帮助。

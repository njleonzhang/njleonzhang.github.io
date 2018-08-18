---
layout: post
title: typescript 2.7 编译选项详解
date: 2018-03-24
categories: 前端
tags: typescript
---

typescript的编译选项一堆，微软爸爸写文档又马马虎虎 😂,  所以官方文档本身就不清不楚，似乎还有错误的地方。本文对各个选项进行实测，以期理解其含义以及每个选项会对编译或生成代码的影响。

typescript版本 2.7.2, 参考[官方文档](https://www.typescriptlang.org/docs/handbook/compiler-options.html)。

test code: https://github.com/njleonzhang/typescript-options

## allowJs

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --allowJs | boolean | false | Allow JavaScript files to be compiled. |

是否会去编译`js`文件。

```js
// leon.js
export let author = 'leon'
```

```js
// index.ts
import { author } from './leon'
console.log(author);
```

allowJs设置为true的时候，生成的文件里会包含`leon.js`编译之后的版本，否则不会。

## allowSyntheticDefaultImports

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --allowSyntheticDefaultImports | boolean | module === "system" or --esModuleInterop | Allow default imports from modules with no default export. This does not affect code emit, just typechecking. |

直接翻译过来是：是否允许从没有default导出的模块中导入default。不影响代码的编译结果，只影响typechecking。

实际测试，似乎没什么作用. 可能我理解不对

```js
// leon.ts
export let author = 'leon'
```

```js
// index.ts
import * as leon from './leon' // 这句选项怎么变都对

console.log(leon.author);
```

## allowUnreachableCode

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --allowUnreachableCode | boolean | false | Do not report errors on unreachable code. |

无法到达的代码，是否报错。

```js
export default function test() {
  return 1
  let a = 1000  // error TS7027: Unreachable code detected.
}
```

## allowUnusedLabels

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --allowUnusedLabels | boolean | false | Do not report errors on unused labels. |

无用的标签，是否报错

```js
loop1:
for (let i = 0; i < 3; i++) {
   loop2:         // error TS7028: Unused label
   for (let j = 0; j < 3; j++) {
      if (i == 1 && j == 1) {
         continue loop1;
      }
      console.log("i = " + i + ", j = " + j);
   }
}
```

## alwaysStrict

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --alwaysStrict | boolean | false | Parse in strict mode and emit "use strict" for each source file |

用严格模式来解析ts代码，并生成严格模式的js代码
关于严格模式参考[MDN的文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode)
官方文档说默认值是false, 实测默认值是`true`

```js
// tsc index.ts
var sum = 015  // error TS1121: Octal literals are not allowed in strict mode.
+ 197
+ 142;
```

## baseUrl

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --baseUrl | string |  | Base directory to resolve non-relative module names. See Module Resolution documentation for more details. |

解析非相对模块名的基准目录。查看模块解析文档了解详情。



```js
// index.ts
import test from 'test'
```

```js
// test.ts
let test = 1

export default test
```

```json
{
  "compilerOptions": {
    "baseUrl": "." // 没有这句，则 error TS2307: Cannot find module 'test'.
  }
}
```

## charset


| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --charset | string | "utf8" | The character set of the input files. |

指定输入文件的编码方式。（一般指ts文件）

## checkJs


| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --checkJs | boolean | false | Report errors in .js files. Use in conjunction with --allowJs. |

和allowjs一起使用，js文件里有错误的时候，是否报错。


```js
// -- checkJs true
// index.js
import { a } from 'test'

a('1324') // error TS2345: Argument of type '"1324"' is not assignable to parameter of type 'number'.
```

```js
// test.ts
export function a(i: number) {
  return i
}
```

## declaration 和 declarationDir

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --declaration <br/> -d | boolean | false | Generates corresponding .d.ts file. |
| --declarationDir | string |  | Output directory for generated declaration files. |

`declaration`指是否生成`.d.ts`文件.
`declarationDir`指定生成的`.d.ts`文件的目录

## diagnostics

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --diagnostics | boolean | false | Show diagnostic information. |

显示诊断信息


设置为true的时候，会生成类似下面的诊断报告：

```
Files:            2
Lines:        19890
Nodes:        91623
Identifiers:  31621
Symbols:      24094
Types:         6581
Memory used: 60631K
I/O read:     0.00s
I/O write:    0.00s
Parse time:   0.29s
Bind time:    0.12s
Check time:   0.80s
Emit time:    0.02s
Total time:   1.23s
```

## disableSizeLimit

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --disableSizeLimit | boolean | false | Disable size limitation on JavaScript project. |

禁用JavaScript工程体积大小的限制。
> 不懂啥意思，也不知道有啥用 :-(

## downlevelIteration

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --downlevelIteration | boolean | false | Provide full support for iterables in for..of, spread and destructuring when targeting ES5 or ES3. |

在生成目标代码为es5或者es3的时候，提供对 for..of, spread and destructuring语法的支持。

> 相当于提供polyfill

```js
let a = [1, 2, 3]

let b = [4, ...a]
```

此选项为false时，转换为
```js
var a = [1, 2, 3];
var b = [4].concat(a);
```

此选项为true时，转换成:
```js
var __read = (this && this.__read) || function (o, n) {
    var m = typeof Symbol === "function" && o[Symbol.iterator];
    if (!m) return o;
    var i = m.call(o), r, ar = [], e;
    try {
        while ((n === void 0 || n-- > 0) && !(r = i.next()).done) ar.push(r.value);
    }
    catch (error) { e = { error: error }; }
    finally {
        try {
            if (r && !r.done && (m = i["return"])) m.call(i);
        }
        finally { if (e) throw e.error; }
    }
    return ar;
};
var __spread = (this && this.__spread) || function () {
    for (var ar = [], i = 0; i < arguments.length; i++) ar = ar.concat(__read(arguments[i]));
    return ar;
};
var a = [1, 2, 3];
var b = __spread([4], a);
```

## emitBOM

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --emitBOM | boolean | false | Emit a UTF-8 Byte Order Mark (BOM) in the beginning of output files. |

生成的文件是否要带BOM头

## esModuleInterop

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --esModuleInterop | boolean | false | Emit __importStar and __importDefault helpers for runtime babel ecosystem compatibility and enable --allowSyntheticDefaultImports for typesystem compatibility. |

生成的文件会为兼容babel而添加`__importStar`和`__importDefault`的helper.
这个选项会把`allowSyntheticDefaultImports`设置成true.

> 类似`allowSyntheticDefaultImports`选项，实测这个选项为true和false并不影响生成的代码。具体什么情况会影响不太清楚。


```js
export let a = 100
let b = 200
export default a
```
加不加这个选项，都生成如下代码：

```js
"use strict";
exports.__esModule = true;
exports.a = 100;
var b = 200;
exports["default"] = exports.a;
```

## forceConsistentCasingInFileNames
| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --forceConsistentCasingInFileNames | boolean | false |Disallow inconsistently-cased references to the same file. |

是否在导入文件的时候需要保证大小写一致。
> 和文件名是否一致不重要，但多次导入的时候大小写一定要一致。


```js
// Test.ts
let a = 1
export default a
```

```js
import a from './test' // 导入小写，没问题
import b from './Test' // 再导入大写，报错。 error TS1149: File name '/Users/leon/Documents/git/typescript-options/forceConsistentCasingInFileNames/Test.ts' differs from already included file name '/Users/leon/Documents/git/typescript-options/forceConsistentCasingInFileNames/test.ts' only in casing.
```

## experimentalDecorators

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --experimentalDecorators | boolean | false | Enables experimental support for ES decorators. |

启动实验的装饰器功能。

## importHelpers

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --importHelpers | boolean | false | Import emit helpers (e.g. __extends, __rest, etc..) from tslib |

是否从tslib里导入__extends, __rest等helper函数

> 这个选项似乎已经没有什么用了。

```js
let b = {
  name: 'leon'
}

let a = {
  age: 10,
  ...b
}
```
无论此项如何设置，始终会转换成：
```js
var __assign = (this && this.__assign) || Object.assign || function(t) {
    for (var s, i = 1, n = arguments.length; i < n; i++) {
        s = arguments[i];
        for (var p in s) if (Object.prototype.hasOwnProperty.call(s, p))
            t[p] = s[p];
    }
    return t;
};
var b = {
    name: 'leon'
};
var a = __assign({ age: 10 }, b);
```

## inlineSourceMap

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --inlineSourceMap | boolean | false | Emit a single file with source maps instead of having a separate file. |

是否生成inline的source map


上栗中的代码，加本选项编译，文件末尾会多出sourcemap:

```js
//# sourceMappingURL= xxxxxxx
```

## inlineSources

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --inlineSourceMap | boolean | false | Emit the source alongside the sourcemaps within a single file; requires --inlineSourceMap or --sourceMap to be set. |

将代码与sourcemaps生成到一个文件中。

上栗中的代码，在加上此项进行编译，同样会在文件结尾加上sourceMappingURL，但生成的内容不同。暂时不太明白有什么实际上的区别。

```js
sourceMappingURL = xxxxx
```

## isolatedModules

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --isolatedModules  | boolean | false | Transpile each file as a separate module (similar to “ts.transpileModule”). |

将每个文件当做一个独立的模块来转义
每个文件都需要是一个模块


```js
// index.ts
function a() {
}

// error TS1208: Cannot compile namespaces when the '--isolatedModules' flag is provided.
```
另外次选项不能和`declaration`一起使用, 不太想的明白为什么 😂

```js
error TS5053: Option 'declaration' cannot be specified with option 'isolatedModules'.
```

## jsx

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --jsx  | string | "Preserve" | Support JSX in .tsx files: "React" or "Preserve". See JSX. |

是否对jsx进行转换。一般不转译，把这个工作推迟给babel.


```js
export function a() {
  return (
    <text>a</text>
  )
}
```

编译：
```js
// --jsx "React"
"use strict";
exports.__esModule = true;
function a() {
    return (React.createElement("text", null, "a"));
}
exports.a = a;
```

```js
// --jsx "Preserve"
"use strict";
exports.__esModule = true;
function a() {
    return (<text>a</text>);
}
exports.a = a;
```

## jsxFactory

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --jsxFactory  | string | "React.createElement" | Specify the JSX factory function to use when targeting react JSX emit, e.g. React.createElement or h. |

```js
export function a() {
  return (
    <text>a</text>
  )
}
```
编译：

```js
// --jsx "React" --jsxFactory h
// 请与上个例子对比。
"use strict";
exports.__esModule = true;
function a() {
    return (h("text", null, "a"));
}
exports.a = a;
```

## lib

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --lib  | string[] |  | List of library files to be included in the compilation.<br>Possible values are:  <br>► <code>ES5</code> <br>► <code>ES6</code> <br>► <code>ES2015</code> <br>► <code>ES7</code> <br>► <code>ES2016</code> <br>► <code>ES2017</code> <br>► <code>ESNext</code> <br>► <code>DOM</code> <br>► <code>DOM.Iterable</code> <br>► <code>WebWorker</code> <br>► <code>ScriptHost</code> <br>► <code>ES2015.Core</code> <br>► <code>ES2015.Collection</code> <br>► <code>ES2015.Generator</code> <br>► <code>ES2015.Iterable</code> <br>► <code>ES2015.Promise</code> <br>► <code>ES2015.Proxy</code> <br>► <code>ES2015.Reflect</code> <br>► <code>ES2015.Symbol</code> <br>► <code>ES2015.Symbol.WellKnown</code> <br>► <code>ES2016.Array.Include</code> <br>► <code>ES2017.object</code> <br>► <code>ES2017.SharedMemory</code> <br>► <code>ES2017.TypedArrays</code> <br>► <code>esnext.asynciterable</code> <br>► <code>esnext.array</code> <br>► <code>esnext.promise</code> <br><br> Note: If <code>--lib</code> is not specified a default list of librares are injected. The default libraries injected are:  <br> ► For <code>--target ES5</code>: <code>DOM,ES5,ScriptHost</code><br>  ► For <code>--target ES6</code>: <code>DOM,ES6,DOM.Iterable,ScriptHost</code> |

编译时，可以包括的库文件的列表。默认会inject一些库的定义。
会严重实际上是影响编译是否能成功。

```js
export default function test() {
  let a = 1
  let b = new Promise(resolve => {
    resolve(1)
  })
  return a
}

// tsc lib/index.ts --lib ES5
// error TS2693: 'Promise' only refers to a type, but is being used as a value here.

// tsc lib/index.ts --lib ES6
// build pass
```

## listEmittedFiles

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --listEmittedFiles | boolean | false | Print names of generated files part of the compilation. |
| --listFiles | boolean | false | Print names of files part of the compilation. |

listEmittedFiles：列出编译生成文件
listFiles：列出参与编译的源文件

```bash
// tsc lib/index.ts --lib ES6 --listEmittedFiles --listFiles

// 生成的文件
TSFILE: /Users/leon/Documents/git/typescript-options/lib/index.js

// 参与编译的文件
/Users/leon/.nvm/versions/node/v8.7.0/lib/node_modules/typescript/lib/lib.es2015.d.ts
/Users/leon/.nvm/versions/node/v8.7.0/lib/node_modules/typescript/lib/lib.es5.d.ts
/Users/leon/.nvm/versions/node/v8.7.0/lib/node_modules/typescript/lib/lib.es2015.symbol.wellknown.d.ts
/Users/leon/.nvm/versions/node/v8.7.0/lib/node_modules/typescript/lib/lib.es2015.reflect.d.ts
/Users/leon/.nvm/versions/node/v8.7.0/lib/node_modules/typescript/lib/lib.es2015.proxy.d.ts
/Users/leon/.nvm/versions/node/v8.7.0/lib/node_modules/typescript/lib/lib.es2015.iterable.d.ts
/Users/leon/.nvm/versions/node/v8.7.0/lib/node_modules/typescript/lib/lib.es2015.symbol.d.ts
/Users/leon/.nvm/versions/node/v8.7.0/lib/node_modules/typescript/lib/lib.es2015.promise.d.ts
/Users/leon/.nvm/versions/node/v8.7.0/lib/node_modules/typescript/lib/lib.es2015.generator.d.ts
/Users/leon/.nvm/versions/node/v8.7.0/lib/node_modules/typescript/lib/lib.es2015.collection.d.ts
/Users/leon/.nvm/versions/node/v8.7.0/lib/node_modules/typescript/lib/lib.es2015.core.d.ts
lib/index.ts
```
从例子我们也能看出`--lib`的作用

## locale

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --locale | string |  | The locale to use to show error messages, e.g. en-us. <br>Possible values are:  <br>► English (US): <code>en</code> <br>► Czech: <code>cs</code> <br>► German: <code>de</code> <br>► Spanish: <code>es</code> <br>► French: <code>fr</code> <br>► Italian: <code>it</code> <br>► Japanese: <code>ja</code> <br>► Korean: <code>ko</code> <br>► Polish: <code>pl</code> <br>► Portuguese(Brazil): <code>pt-BR</code> <br>► Russian: <code>ru</code> <br>► Turkish: <code>tr</code> <br>► Simplified Chinese: <code>zh-CN</code>  <br>► Traditional Chinese: <code>zh-TW</code> |

定义报错的message语言。

```text
// --locale zh-CN
error TS2693: “Promise”仅表示类型，但在此处却作为值使用。
```

## sourceMap, mapRoot 和 sourceRoot

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --sourceMap | boolean | false | Generates corresponding .map file. |
| --mapRoot | string |  | Specifies the location where debugger should locate map files instead of generated locations. Use this flag if the .map files will be located at run-time in a different location than the .js files. The location specified will be embedded in the sourceMap to direct the debugger where the map files will be located. |
| --sourceRoot | string | | Specifies the location where debugger should locate TypeScript files instead of source locations. Use this flag if the sources will be located at run-time in a different location than that at design-time. The location specified will be embedded in the sourceMap to direct the debugger where the source files will be located. |

`sourceMap`: 生成sourceMap

`mapRoot`: 指定调试器去什么位置寻找 `map` 文件. 当运行时` .map` 文件和对应的 `.js` 文件不在同一位置时，需要使用本参数指定 `map` 的位置. 指定的位置会被内嵌到 `sourceMap` 中告诉调试器 `map` 文件所在的位置.
`sourceRoot`: 指定调试器去什么位置寻找 `ts` 文件. 当运行时 `.map` 文件和对应的 `.js` 文件不在同一位置时，需要使用本参数指定 `map` 的位置. 指定的位置会被内嵌到 `sourceMap` 中告诉调试器 `ts` 文件所在的位置.

> 用法猜测：对于商业代码，一般以会把map文件直接放出去，而会放到内网的某个地方，通过这个选项，可以帮助开发人员在内网去调试代码。

```bash
tsc lib/index.ts --sourceMap --mapRoot '\\10.32.0.1\map\1'

xxx code xxxx
//# sourceMappingURL=//10.32.0.1/map/1/index.js.map
```

## maxNodeModuleJsDepth

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --maxNodeModuleJsDepth | number | 0 | The maximum dependency depth to search under node_modules and load JavaScript files. Only applicable with --allowJs. |

去 `node_modules` 里搜索代码的层级, 默认是0.

按照这个说法，似乎默认情况下不应该应许去加载`node_modules`目录下的文件。但是事实上，默认情况下，`node_modules`下的模块是会被加载的。

```js
  - maxNodeModuleJsDepth
    - index.ts
    - node_modules
      - test1
        - index.js
        - node_modules
          - test2
            - index.js

  // maxNodeModuleJsDepth/index.ts
  import a from 'test1' // maxNodeModuleJsDepth设置成什么，这句总是对的
  import b from 'test2' // 这句总是错的。
```
> 可能我的理解有误。

## module

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --module<br/> -m | string | target === "ES3" or "ES5" ? "CommonJS" : "ES6" | Specify module code generation: <code>"None"</code>, <code>"CommonJS"</code>, <code>"AMD"</code>, <code>"System"</code>, <code>"UMD"</code>, <code>"ES6"</code>, <code>"ES2015"</code> or <code>"ESNext"</code>.<br>► Only <code>"AMD"</code> and <code>"System"</code> can be used in conjunction with <code>--outFile</code>.<br>► <code>"ES6"</code> and <code>"ES2015"</code> values may be used when targeting <code>"ES5"</code> or lower. |

指定生成代码的模块格式

```js
// index.ts
export function test() {
  return 'hello'
}
```

```js
// --module CommonJs

exports.__esModule = true;
function test() {
    return 'hello';
}
exports.test = test;
```

```js
// --module AMD

define(["require", "exports"], function (require, exports) {
    "use strict";
    exports.__esModule = true;
    function test() {
        return 'hello';
    }
    exports.test = test;
});
```

```js
// -module system
System.register([], function (exports_1, context_1) {
    "use strict";
    var __moduleName = context_1 && context_1.id;
    function test() {
        return 'hello';
    }
    exports_1("test", test);
    return {
        setters: [],
        execute: function () {
        }
    };
});
```

```
// -module UMD
(function (factory) {
    if (typeof module === "object" && typeof module.exports === "object") {
        var v = factory(require, exports);
        if (v !== undefined) module.exports = v;
    }
    else if (typeof define === "function" && define.amd) {
        define(["require", "exports"], factory);
    }
})(function (require, exports) {
    "use strict";
    exports.__esModule = true;
    function test() {
        return 'hello';
    }
    exports.test = test;
});
```

```
// -module es6 or es2015 or esnext

export function test() {
    return 'hello';
}
```

## target

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --target <br/> -t | string | "es3" | Specify ECMAScript target version: "ES3" (default), "ES5", "ES6"/"ES2015", "ES2016", "ES2017" or "ESNext". <br/> Note: "ESNext" targets latest supported ES proposed features. |

指定目标代码的版本。

```
// index.ts
export function test() {
  return new Promise(resolve => {
    resolve(1)
  })
}
```

```
// --target es3
"use strict";
var __assign = (this && this.__assign) || Object.assign || function(t) {
    for (var s, i = 1, n = arguments.length; i < n; i++) {
        s = arguments[i];
        for (var p in s) if (Object.prototype.hasOwnProperty.call(s, p))
            t[p] = s[p];
    }
    return t;
};
exports.__esModule = true;
function test() {
    return new Promise(function (resolve) {
        var a = {
            a: 1,
            b: 2
        };
        resolve(__assign({ c: 3 }, a));
    });
}
exports.test = test;
```

```
// --target es5
"use strict";
var __assign = (this && this.__assign) || Object.assign || function(t) {
    for (var s, i = 1, n = arguments.length; i < n; i++) {
        s = arguments[i];
        for (var p in s) if (Object.prototype.hasOwnProperty.call(s, p))
            t[p] = s[p];
    }
    return t;
};
Object.defineProperty(exports, "__esModule", { value: true });
function test() {
    return new Promise(function (resolve) {
        var a = {
            a: 1,
            b: 2
        };
        resolve(__assign({ c: 3 }, a));
    });
}
exports.test = test;
```

```
// --target es6
export function test() {
    return new Promise(resolve => {
        let a = {
            a: 1,
            b: 2
        };
        resolve(Object.assign({ c: 3 }, a));
    });
}
```

```
// --target esnext
export function test() {
    return new Promise(resolve => {
        let a = {
            a: 1,
            b: 2
        };
        resolve({
            c: 3,
            ...a
        });
    });
}
```

## moduleResolution

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --moduleResolution | string | module === "AMD" or "System" or "ES6" ? "Classic" : "Node" | Determine how modules get resolved. Either "Node" for Node.js/io.js style resolution, or "Classic". See Module Resolution documentation for more details. |

定义typescript如何去查找module，详情查看[官方说法](https://www.typescriptlang.org/docs/handbook/module-resolution.html)

> 如果遇到node_modules里的模块找不到的情况，可以尝试把此项设置为node

## newLine

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --newLine | string | | Use the specified end of line sequence to be used when emitting files: "crlf" (windows) or "lf" (unix).” |

生成的代码里使用什么样的换行符

## noEmit

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --noEmit | boolean | false | Do not emit outputs. |
| --noEmitHelpers | boolean | false | Do not generate custom helper functions like __extends in compiled output. |

noEmit: 不生成编译结果。
> 😂 那还编译个毛。。。。

noEmitHelpers: 不生成helper函数. 设置为true，则`__assign`，`__extends`等helper函数的实现，不会出现在生成的文件中。

```
export function test() {
  let a = {
    a: 1
  }
  return {
    b: 2,
    ...a
  }
}
```

```
// --noEmitHelpers true
export function test() {
    var a = {
        a: 1
    };
    return __assign({ b: 2 }, a);
}
```

```
// --noEmitHelpers false
var __assign = (this && this.__assign) || Object.assign || function(t) {
    for (var s, i = 1, n = arguments.length; i < n; i++) {
        s = arguments[i];
        for (var p in s) if (Object.prototype.hasOwnProperty.call(s, p))
            t[p] = s[p];
    }
    return t;
};
export function test() {
    var a = {
        a: 1
    };
    return __assign({ b: 2 }, a);
}
```

## noEmitOnError

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --noEmitOnError | boolean | false | Do not emit outputs if any errors were reported. |

编译有错误的时候，是否还生成文件。

## noErrorTruncation

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --noEmitOnError | boolean | false | Do not truncate error messages. |

不截断错误消息

## noFallthroughCasesInSwitch

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| ----noFallthroughCasesInSwitch | boolean | false | Report errors for fallthrough cases in switch statement. |

不允许swith中存在有内容的但没有break的case.

```
export let a = 1

switch(a) {
  case 1:
    console.log(a)  // error TS7029: Fallthrough case in switch.

  case 2:
}

```

## noImplicitAny, noImplicitReturns, noImplicitThis and noImplicitUseStrict

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --noImplicitAny | boolean | false | Raise error on expressions and declarations with an implied any type. |
| --noImplicitReturns | boolean | false | Report error when not all code paths in function return a value. |
| --noImplicitThis | boolean | false | Raise error on this expressions with an implied any type. |
| --noImplicitUseStrict | boolean | false | Do not emit "use strict" directives in module output. |

noImplicitAny 不允许使用隐式的使用`any`
noImplicitReturns 不允许有隐式的return, 即所有分支都要显示的return。实测似乎无效。
noImplicitThis 不允许使用隐式的this，需要明确this的类型
noImplicitUseStrict 生成的文件中是否使用"use strict"。这TM和这选项的名字完全不是一个意思啊。为true的时候，生成的代码不会添加"use strict"

```
// --noImplicitAny true
function a1(b) {  // error TS7006: Parameter 'b' implicitly has an 'any' type.
  return b
}
```

```
// --noImplicitReturns true
function a2(b) {  // 编译通过，与文档似乎不符
  if (b) {
    console.log(b)
    return b
  }
}
```

```
// --noImplicitThis true
function a1(b: any) { // error TS2683: 'this' implicitly has type 'any' because it does not have a type annotation.
  return this.b
}
```

## noLib

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --noLib | boolean | false | Do not include the default library file (lib.d.ts). |

编译的时候不加载`lib.d.ts`.
> 那你还能编译？😂

## noResolve

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --noResolve | boolean | false | Do not add triple-slash references or module import targets to the list of compiled files. |

编译时，忽略`///<reference path>`和`import`引入的文件。

> 😂这种选项到底有啥用

```
// index.ts
/// <reference path="./my.d.ts" />

import { a } from './Test'
```

```
// my.d.ts
declare module A {
  export function test(number, string): any
}
```

```
// test.ts
export let a = 1
```

```
// tsc noResolve/index.ts  --listFiles

/Users/leon/.nvm/versions/node/v8.7.0/lib/node_modules/typescript/lib/lib.d.ts
noResolve/my.d.ts
/Users/leon/Documents/git/typescript-options/noResolve/Test.ts
noResolve/index.ts
```

```
// tsc noResolve/index.ts --noResolve --listFiles
noResolve/index.ts(3,19): error TS2307: Cannot find module './Test'.
/Users/leon/.nvm/versions/node/v8.7.0/lib/node_modules/typescript/lib/lib.d.ts
noResolve/index.ts
```

## noStrictGenericChecks

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --noStrictGenericChecks | boolean | false | Disable strict checking of generic signatures in function types. |

禁用严格的函数泛型检查。

> 不清楚具体指什么。有知道的朋友请指教

## noUnusedLocals, noUnusedParameters

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --noUnusedLocals | boolean | false | Report errors on unused locals. |
| --noUnusedParameters | boolean | false | Report errors on unused parameters. |

noUnusedLocals: 不允许无用的临时变量
noUnusedParameters: 不允许未使用的参数
```
export function A() { // error TS6133: 'b' is declared but its value is never read.
  let a = 0  // error TS6133: 'a' is declared but its value is never read.
  return
}
```

## outDir, outFile

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --outDir | string |  | Redirect output structure to the directory. |
| --outFile | string |  | Concatenate and emit output to single file. The order of concatenation is determined by the list of files passed to the compiler on the command line along with triple-slash references and imports. See output file order documentation for more details. |

outDir: 把导出文件重定向到某个目录
outFile: 把所有的文件拼接并生成到单个文件中。

## paths

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| path | string |  | Redirect output structure to the directory. |

只能用于tsconfig中，解析模块的时候的路径，地址是相对于baseUrl的。详情查看[官方说法](https://www.typescriptlang.org/docs/handbook/module-resolution.html)

## preserveConstEnums

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| preserveConstEnums | boolean | false | Do not erase const enum declarations in generated code. See const enums documentation for more details. |

从生成的代码中擦除静态枚举的声明。
> 实测怎么设置似乎都不起作用。😂

## preserveSymlinks

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| preserveSymlinks | boolean | false | Do not resolve symlinks to their real path; treat a symlinked file like a real one. |

不把符号链接解析为其真实路径；将符号链接文件视为真正的文件。
> 不太明白啥意思。测试了`import`一个`linux`软连接，但是这个选项并没有什么区别。

```
// realIndex.ts
export let a = 1
```

```
// index.ts --> realIndex.ts

// 此选项并不影响 tesc index.ts 的编译
```

## pretty

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --pretty | boolean | false | Stylize errors and messages using color and context. |

对console上错误和消息做颜色和样式的美化处理。

## project

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --project <br/> -p | string | | Compile a project given a valid configuration file. The argument can be a file path to a valid JSON configuration file, or a directory path to a directory containing a tsconfig.json file. See tsconfig.json documentation for more details. |

指定一个`tsconfig.json`文件

## removeComments

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --removeComments | boolean | false | Remove all comments except copy-right header comments beginning with /*! |

删除注释

## rootDir

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --rootDir | string | | Specifies the root directory of input files. Only use to control the output directory structure with --outDir. |

指定输入文件的根目录，仅和outDir一起用于管理输出目录的结构。

```
// ./test/index.ts
let a = 1
```

```
// tsc test/index.ts --outDir ./test/dist/ --rootDir .
生成 ./test/dist/test/index.js

// tsc test/index.ts --outDir ./test/dist/
生成 ./test/dist/index.js
```

## rootDirs

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| rootDirs | string[] | | List of root folders whose combined content represent the structure of the project at runtime. See Module Resolution documentation for more details. |

指定多个根目录，并以此确定运行时的项目结构。
[详见文档](https://www.typescriptlang.org/docs/handbook/module-resolution.html#virtual-directories-with-rootdirs).

> 说人话：允许将不同的目录，通过这个选项都指定为根目录，从而使导入文件的时候比较方便。

## skipLibCheck

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --skipLibCheck | boolean | false | Skip type checking of all declaration files (*.d.ts). |

忽略所有的声明文件（ *.d.ts）的类型检查。

> 暂时不知道怎么用，尝试了几种方法似乎都不起作用。

```
// test.js
export let a = [1, ,2, 3]
```

```
// test.d.ts
export declare let a: number[];
```

```
// index.ts
import { a } from './test'
a[0] = '12'
```
加不加这个选项都会报错：
```
error TS2322: Type '"12"' is not assignable to type 'number'.
```

## strict

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --strict | boolean | false | Enable all strict type checking options. <br>Enabling <code>--strict</code> enables <code>--noImplicitAny</code>, <code>--noImplicitThis</code>, <code>--alwaysStrict</code>, <code>--strictNullChecks</code>, <code>--strictFunctionTypes</code> and <code>--strictPropertyInitialization</code>. |

开启一系列的强制检测。

## strictFunctionTypes, strictPropertyInitialization, strictNullChecks

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --strictFunctionTypes | boolean | false | Disable bivariant parameter checking for function types. |
| --strictPropertyInitialization | boolean | false | Enusre non-undefined class properties are initialized in the constructor. |
| --strictNullChecks | boolean | false | In strict null checking mode, the null and undefined values are not in the domain of every type and are only assignable to themselves and any (the one exception being that undefined is also assignable to void). |

`strictFunctionTypes`: 启用函数类型的严格检查。详见： https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-6.html

`strictPropertyInitialization`: 要求未定义的class成员变量，一定要在构造函数里进行初始化。

`strictNullChecks`: `null` 和 `undefined`, 不属于包含在其他任何类型，所以他们只能 `null` 只能赋值给 `null` 类型或 `any` `类型，undefined` 只能赋值给 `undefined` 类型或者 `any` 类型。但有一个例外: `undefined` 可以被赋值给 `void` 类型

```
class A {
  name: string
  age = 10

  constructor(age: number) { // error TS2564: Property 'name' has no initializer and is not definitely assigned in the constructor.
    this.age = age
  }
}

let a1: number = null       // error TS2322: Type 'null' is not assignable to type 'number'.
let a2: string = undefined  // error TS2322: Type 'undefined' is not assignable to type 'string'.
```

## stripInternal

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --stripInternal | boolean | false | Do not emit declarations for code that has an /** @internal */ JSDoc annotation. |

实验属性。不为有`/** @internal */`标记的代码生成declaration

## suppressExcessPropertyErrors

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --suppressExcessPropertyErrors | boolean | false | Suppress excess property checks for object literals. |

不对对象自变量的额外属性进行类型检测

```
export class A {
  name: string
  age = 10

  constructor(age: number) {
    this.age = age
  }
}

let a: A = {
  name: 'leon',
  age: 30,
  gender: 'male'
}

// --suppressExcessPropertyErrors false
error TS2322: Type '{ name: string; age: number; gender: string; }' is not assignable to type 'A'.
Object literal may only specify known properties, and 'gender' does not exist in type 'A'.

//  --suppressExcessPropertyErrors true
pass
```

## suppressImplicitAnyIndexErrors

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --suppressImplicitAnyIndexErrors | boolean | false | Suppress --noImplicitAny errors for indexing objects lacking index signatures. See issue #1232 for more details. |

修复[#1232](https://github.com/Microsoft/TypeScript/issues/1232#issuecomment-64510362)的一个问题。

## traceResolution

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --traceResolution | boolean | false | Report module resolution log messages. |

打印出module resolve过程的log。当moudle找不到的时候，可以打出log进行排查。

```
Module resolution kind is not specified, using 'NodeJs'.
Loading module as file / folder, candidate module location '/Users/leon/Documents/git/typescript-options/skipLibCheck/test', target file type'TypeScript'.
File '/Users/leon/Documents/git/typescript-options/skipLibCheck/test.ts' does not exist.
File '/Users/leon/Documents/git/typescript-options/skipLibCheck/test.tsx' does not exist.
File '/Users/leon/Documents/git/typescript-options/skipLibCheck/test.d.ts' exist - use it as a name resolution result.
```

## types, typeRoots

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --types | string[] | | List of names of type definitions to include. See @types, –typeRoots and –types for more details. |
| --typeRoots | string[] | |List of folders to include type definitions from. See @types, –typeRoots and –types for more details. |

指定tyes文件列表和目录列表。

> 当你import一个第三方库的时候，ts会根据这个库的package.json里的`types` 或者 `typings`属性指定的目录去加载type. 可以参照[vue的package.json](https://unpkg.com/vue/package.json)和[antd的package.json](https://unpkg.com/antd/package.json)

## emitDecoratorMetadata

| Option | Type | Default | Desc |
| -- | -- | -- | --|
| --emitDecoratorMetadata | boolean | false | Emit design-type metadata for decorated declarations in source. See issue #2577 for details. |

> 关于ts的装饰器，暂未研究，有空再看。 😂

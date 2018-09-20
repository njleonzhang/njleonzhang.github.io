---
layout: post
title: Javascript 的二进制类型
date: 2018-09-20
categories: js
cover: https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/22296fc7-f1f9-f868-0ff1-b818c0df656e.png
tags: javascript
---

二进制的处理方面，js 一直很弱。在 [Typed Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray) 引入之前，js 甚至没有读写二进制内容的能力。随着越来越多 api 的引入, js 也慢慢可以处理 `文件`, `网络流` 这些二进制 buffer。起初 Web 的 js 标准库里没有二进制处理的相关内容，所以 `Node.js` 就自己搞了一套。现在 Web 自己也搞起来了，这就导致了 Web 环境和 Node 环境里的二进制内容处理方式有些不同。

# 基本的二进制对象
`ArrayBuffer`, `SharedArrayBuffer`, `Typed Array`, `DataView` 几种类型是 web 和 node.js 环境里通用的。

## ArrayBuffer
[ArrayBuffer](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer) 是通用的、固定长度的, **未加工的 (raw)** 二进制数据 buffer。ArrayBuffer 的内容无法被直接读写，但是我们可以 `typed array ` 或者 `DataView` 来对其进行处理。

我们看一下 ArrayBuffer 定义:

```typescript
// ArrayBuffer 成员的定义
interface ArrayBuffer {
    /**
      * Read-only. The length of the ArrayBuffer (in bytes).
      */
    readonly byteLength: number;

    /**
      * Returns a section of an ArrayBuffer.
      */
    slice(begin: number, end?: number): ArrayBuffer;
}

// ArrayBuffer 的构造函数
interface ArrayBufferConstructor {
    new(byteLength: number): ArrayBuffer;
}
declare const ArrayBuffer: ArrayBufferConstructor;
```
ArrayBuffer 并没有提供一个读取、修改其内容的方法。我只能够创建、读取其长度或者复制它。

```
let buffer = new ArrayBuffer(8);
console.log(buffer.byteLength); // 8
let buffer1 = buffer.slice(0,1)
console.log(buffer1.byteLength); // 1
```
这玩意是只读的，没啥搞头，我们来看看有搞头的 `Typed Array ` 和 `DataView`

> `SharedArrayBuffer` 的 api 和 ArrayBuffer 类似，不同的是 `SharedArrayBuffer` 可用于不同 web 页面之间的内存共享，详情请参考: [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer)

## Typed Array
`Typed Array` 表示的是一簇类，它们的 API 相似的, 且它们的实例都是 array-like 的.

`Typed Array` 包括:
```
Int8Array();
Uint8Array();
Uint8ClampedArray();
Int16Array();
Uint16Array();
Int32Array();
Uint32Array();
Float32Array();
Float64Array();
```

以 `Int8Array` 为例, 我们看一下它定义的摘要, 完整的定义请移步 [github](https://github.com/Microsoft/TypeScript/blob/67d8263b30e5a8b60cc04899b8ff1b6f69544343/lib/lib.es5.d.ts#L1603):

```typescript
// 以下为部分定义
/**
  * A typed array of 8-bit integer values. The contents are initialized to 0. If the requested
  * number of bytes could not be allocated an exception is raised.
  */
interface Int8Array {
    /**
      * The size in bytes of each element in the array.
      */
    readonly BYTES_PER_ELEMENT: number;

    ...
    forEach(callbackfn: (value: number, index: number,
      array: Int8Array) => void, thisArg?: any): void;
    set(array: ArrayLike<number>, offset?: number): void;
    sort(compareFn?: (a: number, b: number) => number): this;
    toString(): string;

    [index: number]: number;
    ......
}
interface Int8ArrayConstructor {
    readonly prototype: Int8Array;
    new(length: number): Int8Array;
    new(arrayOrArrayBuffer: ArrayLike<number> | ArrayBufferLike): Int8Array;
    new(buffer: ArrayBufferLike, byteOffset: number, length?: number): Int8Array;
    ....
}
declare const Int8Array: Int8ArrayConstructor;
```

```
let arrayBuffer = new ArrayBuffer(4);
let int8Array = new Int8Array(a) // [0, 0, 0, 0]
int8Array[2] = 10 // [0, 0, 10, 0]
```

可以看到，`Int8Array` 有着大量的 api, 它可以对其内容进行各种读写操作。

为了便于理解 `Typed Array`，MDN 将其对应成 `c 语言` 中各种类型的数组，如[下表](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray#TypedArray_objects):

![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/0e18eca9-e86a-bf8d-6c39-3a367249ea89.png)


那么 `Typed Array` 和 `ArrayBuffer` 又是什么关系呢？我们来看下图:
<img src='https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/a079feaf-26e2-1815-02cc-df2dc35f3699.png' width='500' />

可以看到 `ArrayBuffer` 是一个8个 `bit` 为一个单位的内存数组, 上图中 `ArrayBuffer` 的长度是 8，就代表着它是 8 * 8 即 64 `bit` 的内存块。内存这东西，在没有 `格式` 的情况下就是一串 0 1, 0 1 的乱码，只有在有了 `格式` 的条件下才有意义。一种 `Typed Array` 就是一种 `格式`。不同的 `格式` 去理解同一段内存，得到的结果自然是不一样的。`Chrome` 给出了对这段二进制内存的 `格式` 猜测，如果它是 `Int8Array` 那么它的值是 ...; 如果它是 `Int32Array`，那么它的值是...。这也就是为什么 ArrayBuffer 被称为是 `未加工的 (raw)`, 它没有格式，**未加工的**, 是无意义的原料。

## DataView
`DataView` 提供了对 `ArrayBuffer` 内容的底层读写接口。使用这些接口用户不需要考虑[平台相关的字节顺序](https://developer.mozilla.org/en-US/docs/Glossary/Endianness), 即大端序和小端序。(这里就不深入讨论了，有兴趣的童鞋自行谷歌)

```typescript
interface DataView {
    readonly buffer: ArrayBuffer;
    readonly byteLength: number;
    readonly byteOffset: number;
    getFloat32(byteOffset: number, littleEndian?: boolean): number;
    setFloat32(byteOffset: number, value: number, littleEndian?: boolean): void;
    ...
}

interface DataViewConstructor {
    new(buffer: ArrayBufferLike, byteOffset?: number, byteLength?: number): DataView;
}
declare const DataView: DataViewConstructor;
```

简单来说，`DataView` 就提供了各种方法来让你操作 `ArrayBuffer`，这里我举2个例子，有兴趣的童鞋可以自行测试。

```
let buffer = new ArrayBuffer(4)
let dataView = new DataView(buffer)
dataView.setInt8(0, 10)

/*
  DataView(4) {}
  buffer: ArrayBuffer(4)
    [[Int8Array]]: Int8Array(4) [10, 0, 0, 0]
    [[Int16Array]]: Int16Array(2) [10, 0]
    [[Int32Array]]: Int32Array [10]
    [[Uint8Array]]: Uint8Array(4) [10, 0, 0, 0]
*/

dataView.setInt16(2, 5000, true)
/*
  DataView(4) {}
    buffer: ArrayBuffer(4)
      [[Int8Array]]: Int8Array(4) [10, 0, -120, 19]
      [[Int16Array]]: Int16Array(2) [10, 5000]
      [[Int32Array]]: Int32Array [327680010]
      [[Uint8Array]]: Uint8Array(4) [10, 0, 136, 19]
*/
```

# Web 环境里特有的二进制对象
[Blob](https://developer.mozilla.org/en-US/docs/Web/API/Blob) 和 [File](https://developer.mozilla.org/en-US/docs/Web/API/File) 是 web 环境里常用的二进制对象。

## Blob
`Blob` 表示一个不可变的文件对象。

看看定义，`Blob` 很简单。

```typescript
interface Blob {
    /**
     * The size, in bytes, of the data contained in the Blob object.
     */
    readonly size: number;
    readonly type: string;
    slice(start?: number, end?: number, contentType?: string): Blob;
}

type BufferSource = ArrayBufferView | ArrayBuffer;
type BlobPart = BufferSource | Blob | string;

interface BlobPropertyBag {
    type?: string;
}

declare var Blob: {
    prototype: Blob;
    new(blobParts?: BlobPart[], options?: BlobPropertyBag): Blob;
};
```

可以看到 Blob 的构造函数接受 2 个参数，blobParts 是一个 buffer 的数组。options 则是只含有一个 `type` 属性的对象。`type` 属性指明的是这个 Blob 对象的 [MIME类型](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types)。

![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/573d64ba-e7d0-5fe7-12dd-cd0e8ee3ffef.png)

```
var debug = {hello: "world"};
var blob = new Blob([JSON.stringify(debug, null, 2)], {type : 'application/json'});
```

## File
`File` 继承于 `Blob`, 多了 一些属性，其内容一样无法修改.

```
interface File extends Blob {
    readonly lastModified: number;
    readonly name: string;
}

declare var File: {
    prototype: File;
    new(fileBits: BlobPart[], fileName: string, options?: FilePropertyBag): File;
};
```

Web 接口返回的类型都是 `File`, 它是 Web 环境里表示文件的主要类型，其中最典型的就是 type 属性为 file 的 input 了:

```html
<input type="file" id="input" onchange="handleFiles(this.files)">

<script>
  var inputElement = document.getElementById("input");
  inputElement.addEventListener("change", handleFiles, false);

  function handleFiles() {
    if (this.files) {
      var fileList = this.files;
      [].slice.apply(fileList).map(file => {
        console.log(file)
      })
    }
  }
</script>
```

打开 `console` 尝试一下吧:

<div style='border: 1px #72729e solid; padding: 10px; margin-bottom: 10px;'>
  <p>我是例子: </p>
  <input type="file" id="input" onchange="handleFiles(this.files)">
</div>

<script>
  var inputElement = document.getElementById("input");
  inputElement.addEventListener("change", handleFiles, false);

  function handleFiles() {
    if (this.files) {
      var fileList = this.files;
      [].slice.apply(fileList).map(file => {
        console.log(file)
      })
    }
  }

  function drag(ev)
  {
    window.ev = ev
    ev.dataTransfer.setData('Text',ev.target.id);
  }
</script>

注意, 这里的 `input` 上的 `files` 属性挂的并不是 `File`, 而是一个 array-like 对象 `FileList`:

```
interface FileList {
    readonly length: number;
    item(index: number): File | null;
    [index: number]: File;
}
```

更多的关于 File 的 Api 有:
* [FileReader](https://developer.mozilla.org/en-US/docs/Web/API/FileReader) 用于读出 `Blob` 或者 `File` 对象里的内容。
  ```
    var reader = new FileReader();
    reader.onload = function (event) {
      console.log(event.target.result)   // 在 onload 回调里拿到文件内容。
    };
    reader.readAsText(file); // 读文件
  ```
* [DataTransfer](https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer) Html5 拖拽事件相关，[例子](https://developer.mozilla.org/en-US/docs/Web/API/File/Using_files_from_web_applications#Example_Using_object_URLs_to_display_images#Selecting_files_using_drag_and_drop)
* [URL.createObjectURL()](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL) 通过内存文件创建出一个临时的 URL，[例子](https://developer.mozilla.org/en-US/docs/Web/API/File/Using_files_from_web_applications#Example_Using_object_URLs_to_display_images#Example_Using_object_URLs_to_display_images)。
* [XMLHttpRequest.send()](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/send) 用于向后台发送一个 http 请求，请求体可以是一个 `Blob`, 虽然一般不这么干。
* ...

# Node 环境里特有的二进制对象

# Buffer
Node 里的 [Buffer](https://nodejs.org/api/buffer.html) class 继承了 [Uint8Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array)，并实现了更多的接口，它的大小是固定的，和 ArrayBuffer 一样, 表示着一块 `未加工的 (raw)` 内存。

以下为定义的摘要，完整定义请异步 [github](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/14d0a6329aaf71eaa43e6d87e43e6ec74ea17c1c/types/node/index.d.ts#L272):

```
interface Buffer extends Uint8Array {
  write(string: string, offset?: number, length?: number, encoding?: string): number;
  slice(start?: number, end?: number): Buffer;
  readDoubleBE(offset: number, noAssert?: boolean): number;
  writeDoubleBE(value: number, offset: number, noAssert?: boolean): number;
  */
  ...
}

/**
 * Raw data is stored in instances of the Buffer class.
 * A Buffer is similar to an array of integers but corresponds to
 * a raw memory allocation outside the V8 heap.  A Buffer cannot be resized.
 */
declare var Buffer: {
  // @deprecated api start
    new(str: string, encoding?: string): Buffer;
    new(size: number): Buffer;
    new(array: Uint8Array): Buffer;
    new(arrayBuffer: ArrayBuffer | SharedArrayBuffer): Buffer;
    new(array: any[]): Buffer;
    new(buffer: Buffer): Buffer;
  // @deprecated api end

    from(arrayBuffer: ArrayBuffer | SharedArrayBuffer,
      byteOffset?: number, length?: number): Buffer;
    from(data: any[]): Buffer;
    from(data: Uint8Array): Buffer;
    from(str: string, encoding?: string): Buffer;
    ...
};
```

`Buffer` 的构造 (from) 函数显示它可以从多种途径被创建出来，`Uint8Array`, `ArrayBuffer`, `SharedArrayBuffer` 就包含在其中。这就为 Web 环境里的二进制数据传递给 Node.js 来处理提供了通道。

这里提一下 Buffer 的 toString 方法:

```
/*
  Valid string encodings: 'ascii'|'utf8'|'utf16le'|
    'ucs2'(alias of 'utf16le')|'base64'|
    'binary'(deprecated)|'hex', 'utf8' is default
*/

toString(encoding?: string, start?: number, end?: number): string;
```

其作用在于，通过给定的编码方式将二进制 buffer 解码成一个字符串。 上面的注释中列举了所有支持的编码方式，其中 `utf8` 是默认的。眼尖的你可能已经发现了 `base64` 这个选项, 这说明如果你有一个图片的 Buffer, 那么你就可以把它转成一个 `base64` 编码的字符串。

```
let leon = Buffer.from('里昂')
leon.toString() // '里昂'

// let's say image is buffer of a image, then
image.toString('base64') // 图片的 base64 编码字符串
```

那么如果 Buffer 是一个文件的内存，我们怎么知道这块 Buffer 对应的是什么文件呢？Buffer 是 `未加工的 (raw)`, 本身不会指明自己的格式, 所以它并没有 File 或者 Blob 那样指明 MIME type 属性。[file-type](https://github.com/sindresorhus/file-type) 库给我们提供了方法，这个库通过识别文件的 <a  href='http://en.wikipedia.org/wiki/Magic_number_(programming)#Magic_numbers_in_files'>magic number</a> 来识别 Buffer 的文件类型。

Buffer 是 Nodejs 中主要的二进制类型，大量的在 Nodejs 标准库里被应用，比如: [文件系统](https://nodejs.org/api/fs.html)和[网络请求](https://nodejs.org/api/http.html)

* 文件系统库读取文件 api 拿到的基本都是 Buffer.

```
export function readFile(
  path: PathLike | number,
  options: { encoding?: null; flag?: string; } | undefined | null,
  callback: (err: NodeJS.ErrnoException, data: Buffer) => void
): void;
```

* 从文件系统读出的数据流里的内容也可以是 Buffer:

```
export class Readable extends Stream implements NodeJS.ReadableStream {
  ...
  emit(event: "data", chunk: any): boolean; // chunk: <Buffer> | <string> | <any>
  ...
}
```
> 这可以对应于 tcp 层网络流的处理了。

* [Http](https://nodejs.org/api/http.html) 也涉及各种的 Buffer。

# FormData
最后我们还要提一下 [FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData) 这个类。其实大家并不陌生，浏览器做 form 提交时, 给后台发的 http 请求的格式就是 `multipart/form-data`, 此时 http body 里的内容就是一个 `FormData` 对象。有趣的是 `Node.js` 没有**这个类**。那么在 `Node.js` 里去模拟一个 form 提交岂不是搞不了?!

我们先来看一下 FormData 的定义:

```
type FormDataEntryValue = File | string;

interface FormData {
    append(name: string, value: string | Blob, fileName?: string): void;
    delete(name: string): void;
    get(name: string): FormDataEntryValue | null;
    getAll(name: string): FormDataEntryValue[];
    has(name: string): boolean;
    set(name: string, value: string | Blob, fileName?: string): void;
    forEach(callbackfn: (value: FormDataEntryValue, key: string, parent: FormData) => void, thisArg?: any): void;
}

declare var FormData: {
    prototype: FormData;
    new(form?: HTMLFormElement): FormData;
};
```

我们能否自己用 `Node.js` 来实现一个兼容 Web `FormData` 的类呢？上面的声明中 Blob 和 File 在 Node 里是没有的，但是 Buffer 在 `Node` 是可以用来描述`未加工的 (raw)` 内存的，所以理论上可以用 Buffer 去描述任何内存块。说说容易，真的实现这个描述还是千难万难的，所幸 [alex indigo](https://github.com/alexindigo) 已经做到了，他提供了 [form-data](https://github.com/form-data/form-data) 库来让我们在 Node 环境里创建一个可以用于 form 提交的 `form-data`.

# 总结
本文大体的介绍了 Web 环境和 Node 环境里的二进制类型, 类型有点多, 但总体还是有迹可循, 各司其职的。我们只要了解他们之间的关系，未来用起来就能得心应手了。

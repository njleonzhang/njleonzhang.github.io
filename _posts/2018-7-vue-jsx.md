---
layout: post
title: vue jsx 实践
date: 2018-08-17
categories: 前端
tags: vue
---

关于如何写页面的 html，React 的是有坚定的方案的，即 `jsx`. 而 Vue 在这方面就选择更丰富。看看 [vue 2.0 release announce](https://vuejs.org/2016/04/27/announcing-2.0/) 里的说法,

```
Developers tend to have strong opinions on templates vs. JSX.

....

Well, why not have both? In Vue 2.0, you can keep using the familiar template syntax, or drop down to the virtual-DOM layer whenever you feel constrained by the template DSL. Instead of the template option, just replace it with a render function. You can even embed render functions in your templates using the special <render> tag! The best of both worlds, in the same framework.
```

Vue 起源于 `Angular 1`, 并成功的解决了 `Angular 1` 的性能、框架庞大复杂等问题，加之 `Angular 2` 迟迟不问世，且基本抛弃了 `Angular 1` 的 api, 种种因素吧，Vue 继承了大量 `Angular 1` 的用户。在 Vue 1.0 到 2.0 版本的过渡过程中，尤大肯定是谨慎思考了过 `templates or JSX` 的问题的。

* 完全用 Jsx 的话，一方面和 react 太像了，另一方面可能失去原先从 `Angular 1` 继承过来的以及使用 `vue 1.0` 版本的大量用户。
* 完全用 template 的话，很多逻辑实现起来真的会让人崩溃。

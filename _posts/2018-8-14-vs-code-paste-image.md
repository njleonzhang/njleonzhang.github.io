---
layout: post
title: 实现一个 vscode 粘贴图片的插件
date: 2018-08-14
categories: Misc
tags: vscode extension
---

最近常用 vscode 写 markdown, 比较头疼的一个问题就是引用图片了。markdown 里引用图片，需要一个图片的链接。想要为 local 机器上的一张图生成一个链接并用在 markdown 文档里，通常有2种做法:

1. 把图片放到项目里去，一并提交，让图片和你的 markdown 站点挂到一个站下面，这样就可以通过相对路径来访问了图片了. 比如，把图片都放到 `assets` 目录下，然后在 markdown 里这样写 `![](/assets/image.png)`。

2. 把图片放在 CDN 上，然后在 markdown 里引用。比如，我就喜欢用 github 当 cdn 使用，方法可参照 [gist](https://gist.github.com/vinkla/dca76249ba6b73c5dd66a4e986df4c8d).

方案1需要在自己的 repo 里维护这些图片。方案2要自己去上传图片，步骤也挺繁琐。那么能不能通过插件来自动完成这些步骤呢？

# 我心中的理想方案
如果能像拷贝粘贴文字一样，把图片粘贴 markdown 里就好了。

具体步骤：
1. 拷贝一个图片（图片存在于系统的剪贴板中）
2. 在 vscode 里的 markdown 文件里，执行粘贴命令，触发插件的逻辑。
3. 插件从系统的剪贴板中读取图片
4. 插件将图片进行压缩
5. 插件把压缩后的图片上传到 cdn，并得到 cdn 上的链接
6. 插件把 `![](压缩后图片的cdn链接)` 插入到光标的位置上。

用户体验：
  ```
  拷贝一个图片 -> markdown 里粘贴图片 -> ![](压缩后图片的cdn链接)插入到光标处
  ```

完美的计划！

<img src="https://cdn.rawgit.com/njleonzhang/image-bed/master/assets/56eefaa6-1939-a474-7a84-0601702003db.png" style='width: 300px' />

**但是**，步骤3缺是一个难点。目前还没有纯 node.js 的库能够从系统剪贴板里拿到图片，具体讨论可参照讨论：[vscode issue 217](https://github.com/Microsoft/vscode/issues/217) 和 [vscode issue 4972](https://github.com/Microsoft/vscode/issues/4972)。

![](https://cdn.rawgit.com/njleonzhang/image-bed/master/assets/b38cc6b9-25dc-a55f-2498-29ceb0162d4f.png)

# 现有的方案
目前开源的方案有 [vscode-paste-image](https://github.com/mushanshitiancai/vscode-paste-image). 这个插件另辟蹊径，在 mac 系统里，使用 [applescript](https://developer.apple.com/library/archive/documentation/AppleScript/Conceptual/AppleScriptLangGuide/introduction/ASLR_intro.html) 读取系统剪贴板里的图片，在 windows 系统上使用 [powershell](https://docs.microsoft.com/en-us/powershell/scripting/setup/installing-windows-powershell?view=powershell-6), 在 linux 上使用 bash 和 一个剪贴板工具库 [xclip](https://github.com/astrand/xclip). 总体来说，就是每次都开一个进程去执行一段脚本来读取剪贴板里的图片。

但是这个库，没有实现`图片压缩` 和 `cdn上传`，而且现在的代码在我的 mac 上执行也有问题。[applescript](https://developer.apple.com/library/archive/documentation/AppleScript/Conceptual/AppleScriptLangGuide/introduction/ASLR_intro.html) 接触的实在是少，也不知道出了什么问题。种种原因导致我想要自己造了一个轮子。

# 我的方案
上图中，`octref` 提到目前**从系统剪贴板里读取图片**有一个稳定的可行方案：electron 的 [clipboard.readImage](https://electronjs.org/docs/api/clipboard) api. 我们都知道 vscode 是基于 electron 实现的，那么真这么简单么？可以在 vscode 插件里调用 [clipboard.readImage](https://electronjs.org/docs/api/clipboard) api? 答案是否定的，出于某些原因，vscode 竟然不支持在 extension 代码里去使用 electron 的api, 具体的讨论可参见：[vscode issue 3011](https://github.com/Microsoft/vscode/issues/3011).

明明整个 vscode 都基于 electron，但是就是不让你调他的 api，你气不气？没办法，曲线救国吧。来看看咱的方案：

![](https://user-images.githubusercontent.com/13174059/43622590-7e58580e-970f-11e8-8edd-06b97ffedf49.png)

你不让我调用 electron 的 api，那我单独开一个 electron 进程来读取系统剪贴板里。

这个 electron 进程常住于系统中做作为一个 IPC server。当我们**在 vscode 里的 markdown 文件里执行粘贴命令**时，插件就向这个 electron 进程发一个 IPC 消息，这个 electron 进程收到消息后就去读取系统剪贴板，并把结果通过 IPC 消息返回给我们的 vscode 插件。

接下来的事情就简单了，我选用 [tinypng](https://tinypng.com/) 来压缩图片，然后将图片放到 cdn 上，目前只支持[七牛](https://www.qiniu.com/en)。

> 8月16日更新，七牛不允许使用测试域名了，所以没有备案域名就不能再用了，现在实现了对 github 的做 CDN 的支持。

OK, 打完收工，看一下效果：

![](https://user-images.githubusercontent.com/13174059/43623851-146acf7e-9716-11e8-83b9-6fc68bcce2e0.gif)

> electron 进程单独做成了一个 npm 包，以减小插件的大小

> 关于这个插件具体使用和注意事项：请参照[项目工程](https://github.com/njleonzhang/vscode-extension-mardown-image-paste)的 readme.

# 总结
实现的代码虽然很简单，但是找到解决方案的过程真的很曲折。把思路和最终的工具分享在此，希望对大家有用。

> 项目地址:
  * https://github.com/njleonzhang/vscode-extension-mardown-image-paste
  * https://github.com/njleonzhang/electron-image-ipc-server

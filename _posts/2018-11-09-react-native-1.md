---
layout: post
title: react native 开发实践1 - 踩坑
date: 2018-11-12
categories: 前端
cover: https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/9c247e5e-dc6f-283d-f9f8-493171cdc9f9.png
tags: react-native
---

最近在用 react native 做一个小项目, 较为深度的体验了一下 react native 开发, 本文记录一下心得。
本系列文章, 涉及 react native 的坑、调试、版本管理等方面。

react native 各版本差异较大，所以先亮出我的环境:

```
➜ react-native info

React Native Environment Info:
System:
  OS: macOS 10.14.1
  CPU: x64 Intel(R) Core(TM) i5-5257U CPU @ 2.70GHz
  Memory: 1.14 GB / 16.00 GB
  Shell: 5.3 - /bin/zsh
Binaries:
  Node: 8.11.3 - ~/.nvm/versions/node/v8.11.3/bin/node
  Yarn: 1.10.1 - /usr/local/bin/yarn
  npm: 6.4.1 - ~/.nvm/versions/node/v8.11.3/bin/npm
  Watchman: 4.9.0 - /usr/local/bin/watchman
SDKs:
  iOS SDK:
    Platforms: iOS 12.1, macOS 10.14, tvOS 12.1, watchOS 5.1
  Android SDK:
    Build Tools: 22.0.1, 23.0.1, 23.0.2, 23.0.3, 24.0.0, 24.0.1, 24.0.2, 24.0.3, 25.0.0, 25.0.1, 25.0.2, 25.0.3, 26.0.1, 26.0.2, 26.0.3, 27.0.3, 28.0.2, 28.0.3
    API Levels: 10, 14, 17, 19, 21, 22, 23, 24, 25, 26, 27, 28
IDEs:
  Android Studio: 3.2 AI-181.5540.7.32.5056338
  Xcode: 10.1/10B61 - /usr/bin/xcodebuild
npmPackages:
  @types/react: ^16.4.16 => 16.4.18
  @types/react-native: ^0.57.4 => 0.57.7
  react: 16.5.0 => 16.5.0
  react-native: 0.57.2 => 0.57.2
npmGlobalPackages:
  react-native-cli: 2.0.1
  react-native-create-library: 3.1.2
  react-native-git-upgrade: 0.2.7
  react-native-rename: 2.2.2
```

这是这个系列的第一篇, 先谈谈我遇到的一些典型的坑，原因及解决方法。

# 无法链接 Packer server

如果 app 的 remote debug 启动了，那么你看到的报错可能是:

```
Runtime is not ready for debugging. Make sure Packer server is running
```

如果 app 没有启动 remote debug，你看到的错误可能是:

```
Connection to http://localhost:8081/debugger-proxy?role=client xxxxxx
```

这种情况说明调试版的 App 无法访问到你通过 `npm start` 启动的服务器。

这个问题，有这么几种可能性原因和解决方法:

1. 你电脑上的，服务器没有启动。（听起来很傻。。。不过确实会忘记启动）
2. 有时候你能看到电脑上的 Packer server 已经有代码编译的进度条了, 但是手机上还是显示连不到服务器的错误。这种情况等代码编译结束后, 在手机端 reload 一下就好了。
3. 如果你是真机运行 ios，看一下你的手机和电脑在不在一个局域网下。
4. 如果你是真机运行 android, 看一下你的手机 App 是否按[文档](https://facebook.github.io/react-native/docs/running-on-device#method-2-connect-via-wi-fi)配置了 debug server host。
5. 如果你 follow [官方文档](https://facebook.github.io/react-native/docs/running-on-device#3-configure-app-to-use-static-bundle)，对 ios bundle 打包进行了如下的优化，那么恭喜你，你大概率会遇到这样的问题。
  ```
    if [ "${CONFIGURATION}" == "Debug" ]; then
     export SKIP_BUNDLING=true
    fi
  ```
  这个原因在于，打开 `SKIP_BUNDLING` 开关之后, 我们本意是在开发的时候不打包 js 从而节约 ios 原生项目编译的时间, 但是这个选项会带来一个副作用，就是 `ip.txt` 文件不会生成, 在开发版中 ios app 依赖这个文件来寻找 Packer Server 的地址。这是一个 [bug](https://github.com/facebook/react-native/issues/20553), 而且早在 8 月份就[修复](https://github.com/facebook/react-native/pull/20554)了, 但是至今还未发布。😂 官网教你的配置会坑你, 还能说什么。

# bundle cache 问题
react 的代码, 在第一次打包后会被 cache 在系统里, 后续的打包是以增量的形式来打包的. 这避免了每次都去完整的打包, 当然是好事, 问题就在于这套 cache 或者 watcher 系统不太稳定。经常在你修改文件名, 添加, 删除文件的时候出问题。用户会遇到各种各样的报错, 比如:

```
module xxxxxxx does not exist in the Haste module map
```

再如：

```
Could not get BatchedBridge, make sure your bundle is packaged correctly，
```

再如:

```
Unable to resolve module /../react-transform-hmr/lib/index.js
```

解决方案也不稳定, 多数情况下, 重置cache 启动就可以了:

```
watchman watch-del-all
npm start -- --reset-cache
```

严重的时候，甚至需要重装依赖, 更严重是时候竟然要重新生成 android 和 ios 项目:

```
rm -rf node_modules && cnpm i  // 重装依赖
rm -rf ios                     // 重新生成项目
rm -rf android
react-native upgrade
```

> 重新生成项目无疑是不能接受的，尤其是在用户配置了大量第三方 native module 的情况下。上帝保佑, 祝你好运。

# remote debug 在 android 设备上卡住
如果你调试发现代码卡在 `AsyncStorage` 相关的地方，那么恭喜你，这是 android `AsyncStorage` 的一个 [bug](https://github.com/facebook/react-native/issues/12830), 2017年3月10号就被提出来了, github 上喷声一片, facebook 就是不修, 我只能说 amazing！ 问题的原因在于在部分 android 设备或虚拟机上, `AsyncStorage.getItem` 等操作返回的 promise 永远不会被 fulfill 或者 reject. 😂。

# xcode ios 项目编译错误 file cannot be found
xcode 里编译 ios 项目时，会遇到各种文件找不到的情况, 有一类是 `react-native/third-party/` 目录下的一些文件找不到, 比如:

```
Build input file cannot be found: '....../node_modules/react-native/third-party/double-conversion-1.1.6/src/strtod.cc'
```

这种问题的解决方案是再编译一次。。。。。首次编译的时候，十有八九出现 `react-native/third-party/` 下某个文件找不到的错误，不要慌再编译一次通常就好了。如果还没好，google 一下，具体问题具体分析。

比如我遇到过一个 `RCTBundleURLProvider.h` 文件找不到的问题, 按 stack overflow 上的一个[回答](https://stackoverflow.com/a/44039891/4225768)是可以解决.

关于这些问题的原因，在下才疏学浅，不明觉厉，有懂行的朋友, 可以文后留言指教。

# 在一些 android 设备上 debug 版本的 app 显示为一个白屏

在 Android 设备上安装 app 后, 打开并按[文档](https://facebook.github.io/react-native/docs/running-on-device#method-2-connect-via-wi-fi)配置了 debug server host。此时的现象是手机白屏, 电脑上的 Packer server 的 terminal console 里也没有显示任何有 client 连上来的信息。

解决方案: 在手机系统的设置 app 权限的地方给我们这个调试 app 打开一个叫 `悬浮窗` (pop-up window) 的权限。 把 app 彻底关闭，重新打开，一切就恢复正常了。

# 结语
react native 的坑真是多到令人发指, 第一次实践一个项目, 过程可谓磕磕绊绊, 稍微熟悉之后遇到的问题也越来越少, 但是同样的问题又会在新同事的环境里发生。😂 看来 react native 是喜欢欺负新手的。本文介绍了个人遇到的几个坑, 希望对大家有帮助。

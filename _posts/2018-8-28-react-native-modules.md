---
layout: post
title: RN native module 插件开发
date: 2018-08-28
categories: 前端
tags: react-native
---

[React native 官网](https://facebook.github.io/react-native/)上有关于 native module 的开发文档, [ios](https://facebook.github.io/react-native/docs/native-modules-ios) 和 [android](https://facebook.github.io/react-native/docs/native-modules-android)（假设你已经熟悉了官方的这个2篇文档）。 文档给出了实现 RN native module 的 Api 说明和基本的例子。例子演示性的在 RN 的工程里直接添加了现实 native module 相关的源文件。按照这2篇文档去实现 android 和 ios 的 native module 后，其目录结构如下：

```bash
.
├── App.js
├── index.js
├── android
|   ├── app
|   |   ├── src/com/***/
|   |   |   ├── MainActivity.java
|   |   |   ├── MainApplication.java
|   |   |   ├── NativeModule.java          // 继承 ReactContextBaseJavaModule 的类，
|   |   |   |                              // 实现导出的给 js 的 api 接口
|   |   |   └── NativeModulePackage.java   // 继承 ReactPackage 的类,
|   |   |   |                              // 注册 native module
|   |   └── ***
|   └── ***
└── ios
    ├── projectNameFolder
    |   ├── AppDelegate.m
    |   ├── AppDelegate.h
    |   ├── NativeModule.h                 // 声明导出的给 js 的 api 接口,
    |   |                                  // 声明该类实现 RCTBridgeModule 接口
    |   ├── NativeModule.m                 // 实现导出的给 js 的 api 接口
    |   └── ***
    ├── main.m
    └── ***
```

我们可以看到 native module 的实现代码（`NativeModule.java`，`NativeModulePackage.java`, `NativeModule.h`, `NativeModule.m`）被直接插在了 RN 工程的项目里了。

对于演示场景，这样的实现没有问题，但是到了生产中就是另外一回事了。往往我们生产中的 native module 会更复杂，可能会涉及很多的源文件和第三方 SDK, 还要在不同的 RN 工程之间复用, 所以把 native module 独立工程化在生产中是必要的。

实际上我们对此并不陌生, 因为我们所使用的知名第三方 RN 插件均是脱离具体 RN 工程的独立模块，比如: [react-native-navigation](https://github.com/wix/react-native-navigation)。

## 构建 Native Module 插件工程
本文的主要思路是将 native module 的 ios 原生代码做成一个 `static Library 工程`， android 原生代码做成一个 `Android Library`, 最后把 `static Library 工程`、`Android Library 工程`以及`js 代码`一起打包成 npm 包, 便于发布。目录结构如下:

```
.
├── android                                // Android Library, Android 的原生实现
|   ├── app
|   |   ├── src/com/***/
|   |   |   ├── NativeModule.java          // 继承 ReactContextBaseJavaModule 的类，
|   |   |   |                              // 实现导出的给 js 的 api 接口
|   |   |   └── NativeModulePackage.java   // 继承 ReactPackage 的类,
|   |   |   |                              // 注册 native module
|   |   └── ***
|   └── ***
└── ios                                    // Ios static Library, ios 的原生实现
|   ├── NativeModuleFolder
|   |   ├── NativeModule.h                 // 声明导出的给 js 的 api 接口,
|   |   |                                  // 声明该类实现 RCTBridgeModule 接口
|   |   ├── NativeModule.m                 // 实现导出的给 js 的 api 接口
|   |   └── ***
├── demo                                   // 示例的RN工程，同时也是开发的测试工程
|   ├── android
|   |   └── ***
|   ├── ios
|   |   └── ***
|   ├── App.js
|   └── index.js
├── index.js                               // 封装 js 的接口，便于使用。(可选的)
└── package.json                           // for release as npm package
```

示例项目: https://github.com/njleonzhang/play-rn-native-module

### 创建 demo 目录
demo 目录下就是一个普通的 rn 项目，rn 项目的创建过程参见 [RN 文档](https://facebook.github.io/react-native/docs/native-modules-android), 创建命令:

```
react-native init demo
```

### 创建 Android Library
Android Studio 并不能直接创建一个 Android Library 的项目, 我通过以下的步骤曲线救国:

1. 用 Android Studio 打开上面创建出来的 demo 目录下的 android 项目, 再创建模块。 步骤: `Android studio` -> `File` -> `New` -> `New Module...` -> `Android Library`. 完成后在 `demo/android` 的目录下会生成了我们想要 `Android Library`（[示例项目](https://github.com/njleonzhang/play-rn-native-module) 这个 `Android Library` 的名称为 `react-toast-module`）。
2. 把 `demo/android` 下生成的这个 `Android Library` 模块剪贴到项目根目录的下 `android` 目录里。为了让 demo 项目能够正确的导入这个模块，我们需要修改一下 demo 目录下 Android 项目的配置文件: `settings.gradle` 和 `demo/app/build.gradle`:

    ```
    // settings.gradle
    include ':app', ':react-toast-module'

    project(':react-toast-module').projectDir =
      new File(rootProject.projectDir, '../../android')
    ```

    ```
    // app/build.gradle
    dependencies {
        compile fileTree(dir: "libs", include: ["*.jar"])
        compile "com.android.support:appcompat-v7:${rootProject.ext.supportLibVersion}"
        compile "com.facebook.react:react-native:+"  // From node_modules
        compile project(':react-toast-module')
    }
    ```
  3. 添加 react-native 作为这个模块的依赖

      ```
      // ./android/build.gradle
      dependencies {
          compile fileTree(dir: 'libs', include: ['*.jar'])
          compile 'com.android.support:appcompat-v7:26.1.0'
          testCompile 'junit:junit:4.12'
          androidTestCompile('com.android.support.test.espresso:espresso-core:3.0.2', {
              exclude group: 'com.android.support', module: 'support-annotations'
          })

        compile "com.facebook.react:react-native:+"  // From node_modules
      }
      ```

  4. 按[文档](https://facebook.github.io/react-native/docs/native-modules-android)实现 native module.

> 以上关于 Android 原生开发部分的步骤，使用环境是: RN 版本 `0.56.0`, Android Studio 版本 `3.1`

### 创建 Ios static Library
Xcode 支持我们直接创建一个 Ios static Library 的工程，这就不需要像创建 Android Library 那么曲折啦，步骤如下:

1. 在项目的 `ios` 目录下，用 XCode 创建一个 ios 工程，类型为静态库. 步骤: `xcode` -> `File` -> `new` -> `Project` -> `Cocoa Touch Static Library`.
2. 按照 [RN 文档](https://facebook.github.io/react-native/docs/native-modules-ios) 实现 native module，即实现 js 要调用的接口，并通过宏导出。
3. 打开 `demo/ios` 下的示例 ios 项目 `demo.xcodeproj`，把上一步实现的 native module 静态库项目（示例代码里的`./ios/Print/Print.xcodeproj` 和 `./ios/SwiftPrint/SwiftPrint.xcodeproj`）添加作为 Demo 工程的 Libraries里。
  <img src='https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/460b99a5-12dd-c291-351d-164641f3d6ab.png' width='300'>

4. 把 native module 工程的目标静态库，引入到 RN 工程（本例中的demo工程）的链接过程中，步骤：`demo Project` -> `targets` -> `demo` -> `Build Phases` -> `Link Binary with Libraries` -> 添加上面创建的静态库，[示例项目](https://github.com/njleonzhang/play-rn-native-module) 中为(`libPrint.a` 和 `libSwiftPrint.a`).
  ![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/dd9b2237-4f8d-cb2f-6c8a-c8b3e6191917.png)

#### 使用 Swift 开发时的注意点
如果你和我一样不熟悉 OC 的 smallTalk 语系的代码，那么你可能会想使用 swift 作为开发语言，那么这里有一些注意事项:

  * 你无法彻底摆脱 OC, 所以需要处理好 swift 和 OC 互调
    * OC 调 swift，就 import `项目名-Swift.h`
    * Swift 调 OC, 在 `项目名-Bridging-Header.h` 文件里引入 Swift 用到的 OC 类的头文件

    > Swift 不支持宏，所以 Swift 实现的接口，最终还是要通过 OC 的宏来导入. 如果你封装第三方库，那个库很有可能是 OC 写的, 那么你就需要从 Swift 去调用 OC 了。

  * 使用 Swift 实现的接口时，注意 `@objc` 的使用

    需要在 @objc 里把函数的声明完整的写一遍:

      ```
        @objc(test:callback:)
        func test(str: NSString, callback: RCTResponseSenderBlock) {
            NSLog("swift print \(str)")
            callback([1, ["x": 1, "y": 2, "z": 3, "date": NSDate().timeIntervalSince1970]]);
        }
      ```
    上述方案实在是冗余, 根据 [stackoverflow](https://stackoverflow.com/questions/39692230/got-is-not-a-recognized-objective-c-method-when-bridging-swift-to-react-native) 上 `James Wang` 的回答, 可以通过在函数的第一个参数前加个 `_` 来省略 `@objc` 里的函数的声明：

      ```
        @objc
        func test(_ str: NSString, callback: RCTResponseSenderBlock) {
            print("swift print \(str)");
            NSlog("swift print \(str)")
            callback([1, ["x": 1, "y": 2, "z": 3, "date": NSDate().timeIntervalSince1970]]);
        }
      ```

  * RN 项目(即 [示例项目](https://github.com/njleonzhang/play-rn-native-module) 里的 demo 项目) 里的 ios 工程需要有一个空的 swift 文件, 否则链接会出错.
    这点[官方文档](https://facebook.github.io/react-native/docs/native-modules-ios)里有详细的说明:

    > Important when making third party modules: Static libraries with Swift are only supported in Xcode 9 and later. In order for the Xcode project to build when you use Swift in the iOS static library you include in the module, your main app project must contain Swift code and a bridging header itself. If your app project does not contain any Swift code, a workaround can be a single empty .swift file and an empty bridging header.

总体上说，使用 swift 还是有点麻烦，语法上虽然看着舒服点，但是需要去摸索 Swift 语言，还要处理混用 OC 和 Swift 的一些奇葩的地方。So, 你怎么选呢？

> 以上关于 ios 原生开发部分的步骤，使用环境是: RN 版本 `0.56.0`, Xcode 版本 `9.4.1`

### index.js 和 package.json
[示例项目](https://github.com/njleonzhang/play-rn-native-module) 中, 我并没有写 `index.js` 和 `package.json`.

我们可以看下插件使用时候的[代码](https://github.com/njleonzhang/play-rn-native-module/blob/master/demo/App.js#L19)，然后脑补一下：

```
// demo/App.js

import {Platform, StyleSheet, Text, View, NativeModules} from 'react-native';

***

if (Platform.OS === 'ios') {
  NativeModules.Print.test1('fuck', (p1, p2) => {
    console.log(p1, p2)
  })
  NativeModules.SwiftPrint.test('you', (p1, p2) => {
    console.log(p1, p2)
  })

  console.log(NativeModules.Print.firstDayOfTheWeek, NativeModules.SwiftPrint.firstDayOfTheWeek)
} else {
  console.log(NativeModules.Toast)
  NativeModules.Toast.show('hello', NativeModules.Toast.LONG)
}
```

每次使用自定义的接口的时候，都要通过 `NativeModules` 的变量来访问，我们通过在 `index.js` 里创建一些辅助函数来简化一下吧:

```
// index.js

import { NativeModules } from 'react-native';

export let { Print, SwiftPrint, Toast } = NativeModules
```

简化使用:

```
// package.json 里设置 main 属性为 index.js
import { Print, SwiftPrint, Toast } from 'our-rn-native-module-package-name'

***

if (Platform.OS === 'ios') {
  Print.test1('fuck', (p1, p2) => {
    console.log(p1, p2)
  })
  SwiftPrint.test('you', (p1, p2) => {
    console.log(p1, p2)
  })

  console.log(Print.firstDayOfTheWeek, SwiftPrint.firstDayOfTheWeek)
} else {
  console.log(Toast)
  Toast.show('hello', Toast.LONG)
}
```

当然我们的示例插件很简单，这个 `index.js` 可有可无, 但是对于复杂的插件来说，`index.js` 会有更多的帮助方法，工具类的封装和是插件启动的一系列代码。

## 使用插件
RN 插件发布后，一般都要写一段长长的说明文档，来告诉用户怎么去在 ios 和 android 项目里链接插件的原生代码。 比如[react-native-navigation](https://github.com/wix/react-native-navigation) 的光配置部分就要3个章节：[installation-ios](https://wix.github.io/react-native-navigation/#/installation-ios)、[installation-android](https://wix.github.io/react-native-navigation/#/installation-android) 和 [Usage](https://wix.github.io/react-native-navigation/#/usage). Ios 配置一篇，android 配置一篇，js 的基本使用一篇.

非要搞得这么复杂么？原生部分的配置不能自动化么？不能不说这些原生分布的手动配置过程正是 RN 不成熟的地方，也是阻碍 RN 发展的地方。cordova 也有 native plugin，其原理和 RN native module 基本一样，但是 cordova 插件大多都能自动配置，不需要手工干预。

当然在这一方面 react 社区的努力也是有目共睹。 [rnpm](https://github.com/rnpm/rnpm) 被 merge 到了 react 代码中后，`react-native link` 为 native plugin 自动化配置提供了规范, 理论上只要插件的开发人员提供相应的配置脚本，基本所有的插件都能做到自动化配置，比如, [极光推送的 rn 插件](https://github.com/jpush/jpush-react-native)，自动化配置做得就很棒。(虽然它文档写得很乱😱)

关于如何为自己的插件添加配置脚本，让 `react-native link` 能够稳定工作，我还没有研究清楚。 [rnpm](https://github.com/rnpm/rnpm) 的文档实在是简陋，[RN 官方](https://facebook.github.io/react-native/docs/linking-libraries-ios)关于如何第三方插件如何支持 `react-native link` 似乎更是没有描述。😂 这里留个坑吧，有空再来填。

示例项目: https://github.com/njleonzhang/play-rn-native-module

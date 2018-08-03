---
layout: post
title: 通过 CodeFund 赚点零花钱
date: 2018-07-31
categories: Misc
tags: Misc
---

想利用自己的 github pages 上 host 的静态页面（博客，组件库的文档）来赚点钱？准备一个 paypal 账户，就可以开始了。

# 开始
[CodeFund](https://codefund.io/) 的前身是 [CodeSpansor](https://medium.com/codefund/fighting-for-open-source-sustainability-introducing-code-sponsor-577e0ccca025) 致力于帮助开源开发者获得收入，来维持开源开发活动。开源开发本身是一种无偿的，奉献的活动，对于大多数的维护者来说，花费了大量的精力，但是往往不但得不到任何的回报，还会忙于处理各种 `issue`，更有甚者会因为一些原因被用户谩骂。比如这个 [issue](https://github.com/njleonzhang/cordova-baidu-yingyan/issues/8), 完全就不讲道理。

![](http://pcs7p33sr.bkt.clouddn.com/c9d86c3e-1b62-f8cc-a106-ac2a7c288714)

那么如何维持开源开发者的热情呢，[CodeFund](https://codefund.io/) 的作者觉得还是钱比较实在，如果你有技术博客，项目的文档（demo）等网站，并且有一定的浏览量，那么就可以通过 [CodeFund](https://codefund.io/) 在网站上挂点技术广告来赚钱。

来试试吧：

* 注册并登陆 https://codefund.io/
* 填写你需要挂广告的网站的信息
    ![](http://pcs7p33sr.bkt.clouddn.com/e8a557ce-e1b4-3649-0f23-ca39c31a9d3e)
* 得到一段类似下面这段代码

    ```
      <script src="https://codefund.io/scripts/51d43327-eea3-4e27-bd44-e075e67a84fb/embed.js"></script>
      <div id="codefund_ad"></div>

    ```
* 把这段代码拷贝到你的网站的源码里，广告就会自动渲染啦。

[CodeFund](https://codefund.io/) 的广告都符合统一的设计，并且技术相关，不会让你的技术博客看起来太突兀。比如:

![](http://pcs7p33sr.bkt.clouddn.com/d45f793a-70b1-3df3-9dee-12111b0f44d5)

如果你的网站和 **[我的](https://njleonzhang.github.io/vue-data-tables/)** 一样是用 [docsify](https://docsify.js.org/#/) 写的，那么你还可以使用我写的插件 [docsify-plugin-codefund](https://github.com/njleonzhang/docsify-plugin-codefund) 来生成广告。

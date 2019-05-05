---
layout: post
title: 大屏上的全屏页面的自适应适配方案
date: 2018-08-15
categories: 前端
tags: rem viewport flexible
---

通常来说PC端的页面并不像移动端页面那样对屏幕大小和分别率有那么强的依赖。一般的页面都是取屏幕中间的一块宽度(1280px), 两边留白, 高度随着内容的长度滚动。这样无论窗口怎么变化，页面都是可用的。比如，我们的这个[页面](http://zywulian.com/static/html/solution/community-ai.html). 然而也有少数的页面，天生就是要在 pc 端全屏显示的，其中最为典型的代表就是全屏的 dashboard 页面。比如：

当然，如果 dashboard 页面是内嵌在一些管理页面里的，通常是允许滚动的。
![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/ba7c9abd-2f10-3890-cfed-f8c389248b53.png)

但是，如果 dashboard 是用于官方宣传，比如在电视机或者广告屏上的展示的时候，通常是不允许滚动条出现的。比如:

![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/0349a840-ed31-06be-207c-e748883522b7.png)

> 这种 dashboard 有个帅气的名字叫数据可视化。

# 通常的做法
为了实现全屏的这种 dashboard, 通常的做法就是要对宽度和高度都做百分比（网格）来实现了。但是这种方案的缺点在于:
  * 实现太麻烦。设计师给的设计稿通常是 px 为单位标注的，我们需要仔细的计算宽度和高度的比例，然后小心处理页面的布局。
  * 难以处理屏幕宽高比与设计图不符时，带来的元素变形。所以最后展示的屏幕不能和设计稿的屏幕的宽高比差距太大。

  比如，下面这个简单的页面就是用百分比方案来做的。设计师给的图的比例为 16: 9。

  <div class='sizable-iframe' data-link='https://quellingblade.github.io/postcss-px-to-rem/percent.html'>
  </div>

  <div class='sizable-iframe' data-link='https://quellingblade.github.io/postcss-px-to-rem/percent.html' data-width='300' data-height='300'>
  </div>

  <br/>

  当窗口比例是 16 : 9 的时候黄色的长方形显示符合设计，当窗口变成正方形的时候，黄色部分也跟着变方了，这必然会影响显示效果。

  > 可以在浏览器中打开，改变窗口大小页面来体验这个[百分比方案](https://quellingblade.github.io/postcss-px-to-rem/percent.html)。

# 理想的效果
  我心目中的理想效果可能是像下面这个页面一样，无论窗口怎么变，我们的内容都保持原来的比例，并尽量占满窗口（类似 background contain 的效果）。

  * 屏幕尺寸和设计稿比例(16:9)一致时，占满屏幕
  <div class='sizable-iframe' data-link='https://quellingblade.github.io/postcss-px-to-rem'>
  </div>

  <br/>
  * 屏幕尺寸比设计图比例瘦时，上下留白，左右占满，并上下居中, 显示的比例保持16：9
  <div class='sizable-iframe' data-link='https://quellingblade.github.io/postcss-px-to-rem' data-width='300' data-height='300'></div>

  <br/>
  * 屏幕尺寸比设计图比例胖时，左右留白，上下占满，并左右居中, 显示的比例保持16：9
  <div class='sizable-iframe' data-link='https://quellingblade.github.io/postcss-px-to-rem' data-width='600' data-height='300'></div>

  <br/>
  > 可以在浏览器中打开，改变窗口大小页面来体验这个[flexible方案](https://quellingblade.github.io/postcss-px-to-rem/percent.html)

# rem 方案

  熟悉移动端的自适应方案的朋友对 rem 适应方案，肯定不陌生，最出名的就是阿里的 [lib-flexible](https://github.com/amfe/lib-flexible) 方案。可能你已经猜到，本文的这个方案肯定也是基于 rem 的.

  rem (font size of the root element), 是 css3 的引入的一个大小单位。即相对于根元素的 font-size 值的大小。所谓根元素在网页里一般就是 html. 举例说明：下例中，`html 的 font-size 大小`是 20px, 那么 1.4rem 和 2.4 rem 就分别代表着 28px 和 48px 了。

  ```
    html { font-size: 20px; }
    test1 {
      width: 1.4rem; //1.4 × 20px = 28px
    }
    test2 {
      height: 2.4rem; //2.4 × 20px = 48px
    }
  ```

  假设我们的设计稿其尺寸为 1920 * 1280 px，并且实际运行这个网页的屏幕分别率也是 1920 * 1280. 那么，这网页就好做了。简单粗暴地，按图中的元素的尺寸和位置，直接利用绝对定位把所有元素撸出来就行了。比如，设计稿中有这样一个元素:

  ```
  .doughnut {
    top: 150px;
    left: 30px;
    width: 400px;
    height: 300px;
  }
  ```

  这不刚介绍了 rem, 我们试着用 rem 为单位来写一下 `doughnut` 元素的 css。我们把页面的 html 元素的 font-size 设置为 1920 / 10 = 192 px. 那么 `doughnut` 这个元素就应该写作:

  ``` css
  .doughnut {
    top:  .781rem;     // 150 / 192 = 0.781
    left: .156rem;     // 30  / 192 = 0.156
    width: 2.083rem;   // 400 / 192 = 2.083
    height: 1.563rem;  // 300 / 192 = 1.564
  }
  ```
  恩。。。。这不是有病么？算成 rem，然后设置一下 html 的 font-size 让浏览器再算回去？显摆自己的数学好么？23333。

  注意上上面有一个假设，**屏幕大小正好是 1920 * 1280**。这个假设真的很假，根本不可能，一旦用了 px，那么一切长宽都死了。这时你再看一眼 rem，真实的长度为：

    ```
      实际长度(px) = rootFontSize (html 的 font-size) * rem 长度
    ```

  那么这个实际长度必然有一下特点：

  * 所有长度的比例必然和设计图一致。
  * 实际的显示长度完全由 `html 的 font-size` 值决定（线性关系）。

  我们来证明一下：

> 设：设计稿上有任一1条线: A, A 的长度为 $x$, `计算 rem 值的基准`为 $z$ <br/>
> 那么 css 里，A 的长度表示为 $\frac xz$ (rem)
>
> 设网页运行时的 html 的 font-size 值为 $f_s$，
>
>  那么 A 的实际显示长度就分为 $\frac {x f_s} z$ (px)
>
>  所以：
>    1. 对于任意2条线，其实际长度的比例为 $\frac {x_1 f_s} z $ ： $\frac {x_2 f_s} z $ 就等于 他们在设计图上的比例 $x_1$ : $x_2$
>    2. 对于任意一条线，$x$ 和 $z$ 是固定值，其实际值随着 $f_s$ 值线性关系变化的

  我们取设计图的边框的4条线来分析, 那么`设计稿`，`真实显示（画布）`和 `显示窗口（全屏时，即为屏幕）`的关系如下图所示:

  <img src="https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/70ef9642-13d0-a15c-ff25-23feaef1029b.png" width='500'>

  * 设计稿的宽高分别为 $ax$ 和 $ay$, 比例为 $x$ : $y$.
  * 实际显示的范围（我们称其为画布）比例和设计稿保持一致，所以可以设实际显示的画布的宽高为$bx : by$, 且所有线的实际显示长度是由 `html 的 font-size 值` $f_s$ 线性决定的。
  * 屏幕的尺寸不确定，假设其宽度为 $w$, 高度为 $h$

  > 小结一下，用了上面提到的 rem 来方案后，我们做出来的页面是一个**和设计稿比例一致的**，并且**大小根据网页运行时的 html 的 font-size的值缩放**的页面。

  既然页面的大小可以按**html 的 font-size的值**缩放，那么如果我希望**画布的实际显示宽度**始终和**浏览器窗口宽度**保持一致的话（即下图这样的状态），**html 的 font-size的值**应该如何设置呢？

  <img src="https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/19e45d16-534a-9ac3-578c-83bb3f01b45f.png" width='500'>

  我们假设 `计算 rem 值的基准` 为设计稿宽度的 $ \frac 1 q$:

  >  假设，设计稿窗口宽为 $ax$，高为 $ay$, 则`计算 rem 值的基准` $z$ 为 $\frac {ax} q $ <br/>
  >
  >  那么按上面的公式，浏览器中画布实际的<br/>
  >    宽度为 $\frac {ax f_s}{\frac {ax}q} = f_sq$，<br/>
  >    高度为 $\frac {ay fs}{\frac {ax}q} = {\frac {qyf_s} x} $
  >
  >  浏览器窗口的宽度 $w$ 要等于画布实际的宽度，即 $w = f_s q$，则 $f_s = {\frac wq}$

  好的，从数学回到我们的工程中来，我们的设计稿尺寸是 1920 * 1280。我们取 $q$ 这个值为 10, 则 `计算 rem 值的基准` $z$ 为 ${\frac {ax} {q}} = {\frac {1920} {10}} = 192 $. 然后我们把**所有元素的长、宽、位置、字体大小等原来 `px` 单位都转换成 rem**，网页加载后，我们**用 js 去计算当前浏览器窗口的宽度，并设置 `html 的 font-size` $f_s$ 为`当前浏览器窗口的宽度` $w$ 的  $ \frac 1 q$，即 $\frac w {10}$，这样我们就做出了一个100%宽度的、等比例缩放设计稿的页面**了。

  > 通过这样的设置，我就得到了一个**和设计稿比例一致的**，**宽度与窗口大小一致**的页面。

  到此为止，就是现有 rem 方案的核心内容了。

  说了半天，都是别人的方案，读者可能会问了，那博主你干了啥？

# 我的方案
  回头想一下，我们要的是什么？现在这个方案，能满足我们的要求么？我们来逐条分析：
  * 屏幕（窗口）尺寸和设计稿比例($x:y$)一致时，占满屏幕
  <img src="https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/327c8e32-cb68-0615-fa7f-b0bbec3dc4f1.png" width='500'>

    这种情况肯定没问题，屏幕和真实页面完美重合.

  * 屏幕（窗口）尺寸比设计图比例瘦时，上下留白，左右占满，并上下居中, 显示的比例保持($x:y$)
  <img src="https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/19e45d16-534a-9ac3-578c-83bb3f01b45f.png" width='500'>
  这种情况也没问题，真实页面高度小于屏幕，然后页面内容上下居中就可以了。

  * 屏幕（窗口）尺寸比设计图比例胖时，左右留白，上下占满，并左右居中, 显示的比例保持($x:y$)
  <img src="https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/c2b3b4d8-759c-14ce-2ffe-6ab2f59b15b4.png" width='500'>
  问题出现了，在这种场景下，页面的高度超出了屏幕的高度，这就会导致垂直滚动条了。这就是我的方案处理的地方了。在这种场景下，我们需要页面的高度缩小为屏幕的高度，当然为了保持比例页面的宽度等也要等比例缩小.换句话说我们要把所有的线的长度等比例缩小，缩小后画布的高度要等于屏幕的高度，即下图所示的状态:

  <img src='https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/d312b3ec-ded3-746f-9e59-7daa68811f30.png' width='500'>

  要等比例缩小所有的长度，那么操作 `html 的 font-size` $f_s$ 的值就能做到了。我们上面的分析中，我们已经计算了页面（画布）的真实高度
  > 那么按上面的公式，浏览器中页面（画布）的真实<br/>高度为
  > $\frac {ay fs}{\frac {ax}q} = {\frac {qyf_s} x} $

  在此场景下，我们需要把画布真实高度值缩小到屏幕的高度,

  > 设窗口的高度为 $h$,<br/>
  > 设缩小比例为 $s$, 则有
  > $h = {\frac {qyf_s} x} s$, 得 $s = {\frac {xh} {qyf_s}}$ <br/>
  > 即我们需要在页面（画布）真实高度上 ${\frac {qyf_s} x} $ 乘上个缩小系数 ${\frac {xh} {qyf_s}}$，可以使页面（画布）的真是高等于窗口的高度了<br/>
  > 又因为 $f_s = \frac wq$, 则这个缩小系数可变换为: <br/>
  > ${\frac {xh} {qy {\frac wq}}} = {\frac {xh} {yw}} = {\frac xy} / {\frac wh}$, 即 $\frac {设计稿宽高比} {窗口宽高比}$

  Bingo. 综上可知: 当窗口尺寸比设计图比例胖时，只要我们在原来 $f_s$ 值的基础上，乘上 $\frac {设计稿宽高比} {窗口宽高比}$ 的缩小系数，就可以实现我们想要的效果了。

# 工程应用
好了，前面的理论，你看着头疼，我写着更头疼。终于到了喜闻乐见的运用环节了。

按照前面的一顿操作，应用这个方案，我们需要做2件事:
（以设计稿的尺寸为 1920 * 1280 为例）
  1. 在 css 表示长度的时候，用设计稿上的长度除以 192, 算得 rem 的值。
  2. 页面内写一段 js 代码，根据我们上面的公式去计算并设置 `html 元素的 font-size` 值。

关于第1点：如果告诉你所有的长度你要自己算。。。可能这个方案马上就没人用了，因为真的要算死人。参照 [lib-flexible](https://github.com/amfe/lib-flexible) 的方案，我写了一个 [post-css 插件](https://github.com/QuellingBlade/postcss-px-to-rem)来帮助你做这个计算，效果就是你不用算了，图上是多少长度，你写多少就行了，这个 rem 的转换由[插件](https://github.com/QuellingBlade/postcss-px-to-rem)完成。

关于第2点：这段 js 代码我已经为你写好了 [lib-flexible-for-dashboard](https://github.com/QuellingBlade/lib-flexible-for-dashboard), 直接嵌入你的 html 里就行。考虑到这段 js 代码，会计算 font-size 的值，这个值会决定所有的长度，所以这个值要优先计算出来，最好的方案就是把这段代码拷贝到 html 的 head 里去（这个操作被称为 inline）. 为了方便你使用webpack 和 npm 管理这个库，我们还为你准备了一个 [webpack 插件](https://github.com/QuellingBlade/html-webpack-inline-plugin)，帮助你去做 inline.

# 示例
> talk is cheap, show me the [code](https://github.com/njleonzhang/flexible-pc-full-screen)

设计稿是这样的一个 1920 * 1280（16：9）的图:
![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/562e47bd-abda-dae4-7765-68706b5a978e.png)

看看我们实现的[效果](http://www.njleonzhang.com/flexible-pc-full-screen/):

<div class='sizable-iframe' data-link='https://www.njleonzhang.com/flexible-pc-full-screen/'>
</div>

<div class='sizable-iframe' data-link='https://www.njleonzhang.com/flexible-pc-full-screen/' data-width='300' data-height='300'></div>

<div class='sizable-iframe' data-link='https://www.njleonzhang.com/flexible-pc-full-screen/' data-width='600' data-height='300'></div>

iframe 太小所以 echart 里的图会挤在一起看不清, 但是整个页面确实是能够保持对屏幕的适应的。

# 总结
我是站在巨人的肩膀上，针对全屏 pc 页面这一特别场景做出了一个个人比较满意的方案，希望能给有需要的朋友一些帮助。

> 相关项目：

  https://github.com/njleonzhang/flexible-pc-full-screen
  https://github.com/QuellingBlade/postcss-px-to-rem
  https://github.com/QuellingBlade/lib-flexible-for-dashboard
  https://github.com/QuellingBlade/html-webpack-inline-plugin

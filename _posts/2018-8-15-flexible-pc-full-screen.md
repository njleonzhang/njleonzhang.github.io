---
layout: post
title: 大屏上的全屏页面的自适应适配方案
date: 2018-08-15
categories: solution
tags: rem viewport flexible
---

通常来说PC端的页面并不像移动端页面那样对屏幕大小和分别率有那么强的依赖。一般的页面都是取屏幕中间的一块宽度(1280px), 两边留白, 高度随着内容的长度滚动。这样无论窗口怎么变化，页面都是可用的。比如，我们的这个[页面](http://zywulian.com/static/html/solution/community-ai.html). 然而也有少数的页面，天生就是要在 pc 端全屏显示的，其中最为典型的代表就是全屏的 dashboard 页面。

当然，如果 dashboard 页面是内嵌在一些管理页面里的，通常是允许滚动的。
![](https://cdn.rawgit.com/njleonzhang/image-bed/master/assets/ba7c9abd-2f10-3890-cfed-f8c389248b53.png)

但是，如果 dashboard 是用于官司宣传，在电视机或者广告屏上的展示的时候，通常是不允许滚动条出现的。

![](https://cdn.rawgit.com/njleonzhang/image-bed/master/assets/0349a840-ed31-06be-207c-e748883522b7.png)

> 这种 dashboard 似乎有个帅气的名字叫数据可视化。

# 通常的做法
为了实现全屏的这种 dashboard, 通常的做法就是要对宽度和高度都做百分比（网格）来实现了。但是这种方案的缺点在于:
  * 实现太麻烦。设计师给的设计稿通常是 px 为单位标注的，我们需要仔细的计算宽度和高度的比例，然后小心处理页面的布局。
  * 难以处理屏幕宽高比与设计图不符时，带来的元素变形。所以最后展示的屏幕不能和设计稿的屏幕的宽高比差距太大。

  比如，下面这个简单的页面就是用百分比方案来做的，设计师给的图是一个 16: 9 的全屏效果，如果我们就用百分比来做，效果可能就是这样的：

  <div class='sizable-iframe' data-link='https://quellingblade.github.io/postcss-px-to-rem/percent.html'>
  </div>

  <div class='sizable-iframe' data-link='https://quellingblade.github.io/postcss-px-to-rem/percent.html' data-width='300' data-height='300'>
  </div>

  <br/>

  随着窗口比例的变化，整个画面会跟着变形。

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

  > 可以在浏览器中打开，改变窗口大小页面来体验这个[flexible方案](https://quellingblade.github.io/postcss-px-to-rem/percent.html)

  也许这个例子太简单了，那么我们来看一个相对复杂的真实的[例子](http://www.njleonzhang.com/flexible-pc-full-screen/):

  设计稿是这样的一个 1920 * 1280（16：9）的图:
  ![](https://cdn.rawgit.com/njleonzhang/image-bed/master/assets/562e47bd-abda-dae4-7765-68706b5a978e.png)

  看看我们实现的[效果](http://www.njleonzhang.com/flexible-pc-full-screen/):

  <div class='sizable-iframe' data-link='http://www.njleonzhang.com/flexible-pc-full-screen/'>
  </div>

  <div class='sizable-iframe' data-link='http://www.njleonzhang.com/flexible-pc-full-screen/' data-width='300' data-height='300'></div>

  <div class='sizable-iframe' data-link='http://www.njleonzhang.com/flexible-pc-full-screen/' data-width='600' data-height='300'></div>

  iframe 太小所以 echart 里的图会挤在一起看不清, 但是整个页面确实是能够保持对屏幕的适应的。

# rem 方案

  熟悉移动端的自适应方案的朋友对 rem 适应方案，肯定不陌生，最出名的就是阿里的 [lib-flexible](https://github.com/amfe/lib-flexible) 方案。可能你已经猜到，本文的这个方案肯定也是基于 rem 的.

  rem (font size of the root element), 是 css3 的引入的一个大小单位。即相对于根元素的 font-size 值的相对大小。所谓根元素在网页里一般就是 html, 如下例 html 的 font-size 大小是 20px, 那么 1.4rem 和 2.4 rem 就分别代表着 28px 和 48px 了。

  ```
    html { font-size: 20px; }
    test1 {
      width: 1.4rem; //1.4 × 20px = 28px
    }
    test2 {
      height: 2.4rem; //2.4 × 20px = 48px
    }
  ```

  以上文中提到的设计稿为例，其尺寸为 1920 * 1280 px. 为了便于理解，我们先假设实际运行这个网页的屏幕分别率就是 1920 * 1280. 那么，这网页就好做了。简单粗暴地，按图中的元素的尺寸和位置，直接利用绝对定位把所有元素撸出来。比如设计稿中左上角的方形区域的 css, 就这可以这样来写:

  ```
  .doughnut {
    top: 150px;
    left: 30px;
    width: 400px;
    height: 300px;
  }
  ```

  那么现在我不想用 px 了， 我想用 rem 来作为单位。也行，我们把页面的 html 元素的 font-size 设置为 1920 / 10 = 192 px. 那么 `doughnut` 这个元素就应该写作:

  ``` css
  .doughnut {
    top:  .781rem;     // 150 / 192 = 0.781
    left: .156rem;     // 30  / 192 = 0.156
    width: 2.083rem;   // 400 / 192 = 2.083
    height: 1.563rem;  // 300 / 192 = 1.564
  }
  ```
  恩。。。。这不是有病么？算成 rem，然后设置一下 html 的 font-size 让浏览器再算回去？显摆自己的数学好么？我没有那么无聊啊，23333。

  注意上面说了，假设屏幕大小正好是 1920 * 1280。这个假设真的很假，实际上根本不可能，一旦用了 px 一切长宽都死了。这时你再看一眼 rem 的方案，真实的长度为：

    ```
      实际长度(px) = rootFontSize (html 的 font-size) * rem 长度
    ```

  那么这个实际长度必然有一下特点：

  * 所有长度的比例必然和设计图一致。
  * 实际的显示长度完全由 `html 的 font-size` 值决定（线性关系）。

  <img src="https://cdn.rawgit.com/njleonzhang/image-bed/master/assets/70ef9642-13d0-a15c-ff25-23feaef1029b.png" width='500'>

  我们来证明一下：

> 假设设计稿上有2条线: A 和 B, A 的长度为 `x`, B 的长度为 `y`, `计算 rem 值的基准`为 `z` <br/>
> 那么 css 里，A 和 B 的长度分别为 `x` / `z` (rem), `y` / `z`（rem）
>
>  再设网页运行时的 html 的 font-size 值为 `fs`，
>
>  那么 A 和 B 的真实长度就分别为 `x` / `z` * `fs` (px) 和 `y` / `x` * `fs` (px)
>
>  设计图上 A 长度 和 B 长度的比例为：`x` : `y` <br/>
>  真实网页中 A 长度 和 B 长度的比例为: (`x` / `z` * `fs`) / (`y` / `x` * `fs`) = `x` : `y`
>
>  所以：
>    1. 设计图上 A 长度 和 B 长度的比例 必然等于 真实网页中 A 长度 和 B 长度。
>    2. x, y, z 都是固定值，那么 A 和 B 的真实值由 fs 来决定了。（线性关系）


  所以，用了上面提到的 rem 来方案后，我们做出来的页面是一个**和设计稿比例一致的**，并且**大小根据网页运行时的 html 的 font-size的值缩放**的页面。

  既然页面的大小可以按**html 的 font-size的值**缩放，那么如果我希望页面的宽度始终和浏览器窗口保持一致的话（即下图这样的状态），**html 的 font-size的值**应该如何设置呢？

  <img src="https://cdn.rawgit.com/njleonzhang/image-bed/master/assets/19e45d16-534a-9ac3-578c-83bb3f01b45f.png" width='500'>

  我们假设 `计算 rem 值的基准` 为设计稿宽度的 q 分之 1:

  >  假设，设计稿窗口宽为 `ax`，高为 `ay`, 则`计算 rem 值的基准` `z` 为 `ax` / `q`<br/>
  >
  >  那么按上面的公式，浏览器中真实页面的<br/>
  >    宽度为 `ax` / (`ax` / `q`) * `fs` = `fs` * `q`，<br/>
  >    高度为 `ay` / (`ax` / `q`) * `fs` = `q` * `y` * `fs` / `x`
  >
  >  假设浏览器窗口的宽度为 `w`，那么当 `w` = `fs` * `q` 的时候，
  >  浏览器中真实页面的宽度就等于浏览器的宽度了, `fs` = `w` / `q`

  好的，从数学回到我们的工程中来，我们的设计稿尺寸是 1920 * 1280。我们取 `q` 这个值为 10, 则 `计算 rem 值的基准` `fs` 为 `ax` / `q` = 1920 / 10 = 192. 然后我们把**所有元素的长、宽、位置、字体大小等原来 `px` 单位都转换成 rem**，网页加载后，我们**用 js 去计算当前浏览器窗口的宽度，并设置 `html 的 font-size` `fs` 为`当前浏览器窗口的宽度` `w `的 1 / q**, 即 `w` / 10，这样我们就做出了一个**100%宽度的、等比例缩放设计稿的页面**了。

  到此为止，就是现有 rem 方案的核心内容了。

  说了半天，都是别人的方案，读者可能会问了，那博主你干了啥？是时候来说说我到底干了什么了。

# 我的方案
  回头想一下，我们要的是什么？现在这个方案，能满足我们的要求么？我们来逐条分析：
  * 屏幕尺寸和设计稿比例(16:9)一致时，占满屏幕
  <img src="https://cdn.rawgit.com/njleonzhang/image-bed/master/assets/327c8e32-cb68-0615-fa7f-b0bbec3dc4f1.png" width='500'>

  这种情况肯定没问题，屏幕和真实页面完美重合.

  * 屏幕尺寸比设计图比例瘦时，上下留白，左右占满，并上下居中, 显示的比例保持16：9
  <img src="https://cdn.rawgit.com/njleonzhang/image-bed/master/assets/19e45d16-534a-9ac3-578c-83bb3f01b45f.png" width='500'>
  这种情况也没问题，真实页面高度小于屏幕，然后页面内容上下居中就可以了。

  * 屏幕尺寸比设计图比例胖时，左右留白，上下占满，并左右居中, 显示的比例保持16：9
  <img src="https://cdn.rawgit.com/njleonzhang/image-bed/master/assets/c2b3b4d8-759c-14ce-2ffe-6ab2f59b15b4.png" width='500'>

  问题出现了，在这种场景下，页面的高度超出了屏幕的高度，这就会导致垂直滚动条了。这就是我的方案处理的地方了。
  在这种场景下，我们需要页面的高度缩小为屏幕的高度，同时将页面的宽度等比例缩小。即如下图所示:
  <img src='https://cdn.rawgit.com/njleonzhang/image-bed/master/assets/d312b3ec-ded3-746f-9e59-7daa68811f30.png' width='500'>

  解决的关键还是在于 `html 的 font-size` 的值该如何设置。看一下我们上面的推到，有一个高度，我算了但是并没有用：

  > 那么按上面的公式，浏览器中真实页面(画布元素)的<br/>
  > 高度为 `ay` / (`ax` / `q`) * `fs` = `q` * `y` * `fs` / `x`

  在此场景下，我们需要把页面（画布）高度值缩小到屏幕的高度,


  > 设窗口的高度为 `h`,<br/>
  > 则我们需要在页面（画布）高度（`q` * `y` * `fs` / `x`）乘上个系数: `x` * `h` / (`q` * `y` * `fs`) <br/>
  > 因为 `fs` = `w` / `q`, 我们变换一下: <br/>
  > `x` * `h` / `q` * `y` * `fs` = `x` * `h` / (`q` * `y` * `w`) * `q` = (`x` / `y`) / (`w` / `h`), 即设计稿宽高比 / 窗口宽高比

  Bingo. 在这种特殊的场景下，我们把页面缩小，即乘上`设计稿宽高比 / 窗口宽高比`，就可以现实我们想要的效果了。

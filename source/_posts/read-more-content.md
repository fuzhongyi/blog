---
title: 纯 CSS 实现多行省略
date: 2020-06-28 23:48:29
tags: css
categories : 技术水波文
---

![](/read-more-content/header-img.png)

## 什么是多行省略

当字数达到一定程度超出限制行数就会显示省略号。

## -webkit-line-clamp

我们可以使用 `-webkit-line-clamp` 简单实现。

``` css
div {
    width: 300px;
    overflow : hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
}
```

![](/read-more-content/2-1.png)

但 `-webkit-line-clamp` 是一个不规范的属性（unsupported WebKit property），它并没有出现在 `CSS` 规范草案中。因此这个方案并不完美。

## 一般方案

1. 通过脚本截取内容获取高度判断显隐查看更多
2. 换行显示，在新的一行显示查看更多

## 新的方案

利用右浮动原理——右浮动元素从右到左依次排列，不够空间则换行。蓝色块、红色块、绿色块依次右浮动，蓝色块高度小于 6 行文字时，绿色块在右边，蓝色块高度大于等于 6 行文字时，左下角刚好够绿色块排列的空间，于是绿色块就到左边了。

![](/read-more-content/3-1.gif)

```html
<div class="contain">
  <div class="content">内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容容内容内容内容内容内容内容内容内容内容内容内容内容内容容内容内容内容内容内容内容内容内容</div>
  <div class="placeholder"></div>
  <div class="more">...更多</div>
</div>
```

```css
@-webkit-keyframes width-change {
  0%,
  100% {
    width: 400px;
  }
  50% {
    width: 300px;
  }
}

.contain {
  line-height: 20px;
  animation: width-change 8s ease infinite;
}

.content {
  margin-left: -80px;
  float: right;
  width: 100%;
  background: rgba(0, 0, 255, 0.5);
}

.placeholder {
  float: right;
  width: 80px;
  height: 100px;
  background: rgba(255, 0, 0, 0.5);
}

.more {
  position: relative;
  float: right;
  width: 80px;
  height: 20px;
  background: rgba(0, 255, 0, 1);
  left: 100%;
  transform: translate(-100%, -100%);
}
```

我们可以稍作改动：

``` diff
.more {
  position: relative;
  float: right;
  width: 80px;
  height: 20px;
  background: rgba(0, 255, 0, 1);
+ left: 100%;
+ transform: translate(-100%, -100%);
}
```

![](/read-more-content/3-2.gif)

这样 `更多` 就移动到了我们预设的位置，通过文字溢出截断就达到我们最终想要的效果。

``` diff
.contain {
  line-height: 20px;
  animation: width-change 8s ease infinite;
+ max-height: 100px;
+ overflow: hidden;
}
```

![](/read-more-content/3-3.gif)

<iframe height="265" style="width: 100%;" scrolling="no" title="ellipsis" src="https://codepen.io/fuzhongyi/embed/wvMEpyv?height=265&theme-id=dark&default-tab=css,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/fuzhongyi/pen/wvMEpyv'>ellipsis</a> by 歆月
  (<a href='https://codepen.io/fuzhongyi'>@fuzhongyi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

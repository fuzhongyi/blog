---
title: Material Design 涟漪效果
date: 2018-01-26 15:39:35
categories: 技术水波文
tags: [html,css,javascript]
---

![](/material-design-ripple-effect/header-img.png)

## 设计思路

### 结构

分析 Material Design 的 Button Ripple Effect 不难发现，涟漪的扩散是在 Button 的内部进行的，所以我们需要在触发的时候向其内部加入`span`（自定义）。

### 扩散半径

`span`是以圆向四周扩散，为完全扩散到`button`边缘，`span`的直径应是`button`宽高最长那一边的 2 倍。

### 扩散中心点

扩散是以鼠标点击为中心，所以我们需要获取到鼠标点击相对`button`的位置，但这还不够，因为扩散是从中心开始的，所以还要减去`span`的半径长度，如下图：

![](/material-design-ripple-effect/1-1.png)

+ **left = clentX - button.left - 直径 / 2**

+ **top = clentX - button.top - 直径 / 2**


## 示例

<script async src="//jsfiddle.net/xyue/co1pne81/embed/"></script>

实际应该在`button`内部扩充一个`span`填满整个`button`，再赋予这个`span`添加涟漪的点击效果，这样`button`就不会因为必须添加 *position: relative;overflow: hidden;* 而影响布局了，小伙伴赶快去试试吧！😉

---
### 参考

[Material Design 水波纹点按效果](https://idiotwu.me/material-design-ripple-effect-with-css3-and-javascript/)
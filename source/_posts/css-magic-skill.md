---
title: CSS魔法技巧 [ing...]
date: 2017-12-15 17:09:01
tags: css
categories: 技术水波文 
---

![](/css-magic-skill/header-img.jpg)

## 利用 content 属性 attr 抓取资料

>鼠标悬浮提示文字，类似 github 的这种，如图：

![](/css-magic-skill/1-1.png)

相信大家第一想到的就是伪元素——`after`、`before`，利用 content 来显示不同的提示文字，但不同元素提示文字不一定相同。在不能使用 javascript 的前提条件下，我们该如何去控制 content 的内容呢？

利用 content 属性`attr`抓取资料。
在 attr 里面写入 html 中属性名称，这样伪元素 (:after) 就能获取到该属性的值。

```html
<div data-msg="Open this file in Github Desktop">hover</div>
```

```css
div{
		width:50px;
		position:relative;
		cursor: pointer;
	}
	div:hover:after{
		content:attr(data-msg);
		position:absolute;
		left:0;
		top:25px;
		width:400%;
		text-align:center;
		font-size: 12px;
		line-height:30px;
		color: #fff;
		background: #1b1f23;
		border-radius: 3px;
	}
```
效果，如图：

![](/css-magic-skill/1-2.png)

同样的，你还可以结合其他强大的选择器使用，例如：**使用属性选择器选择空链接**

显示没有文本值但是`href`属性具有链接的`a`元素的链接：
```css
a[href^="http"]:empty::before {
  content: attr(href);
}
```
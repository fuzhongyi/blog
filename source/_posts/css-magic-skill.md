---
title: CSS 黑魔法小技巧 [ing...]
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

除此之外，你还可以结合其他选择器使用，例如：**使用属性选择器选择空链接**

显示没有文本值但是`href`属性具有链接的`a`元素的链接：

```css
a[href^="http"]:empty::before {
  content: attr(href);
}
```

## tab 切换

实现 Tab 切换的难点在于如何使用 CSS 接收到用户的点击事情并对相关的节点进行操作。即：

1. 如何接收点击事件
2. 如何操作相关DOM

这里我们采用`:target`伪类接收点击事件。

>URL 带有后面跟有锚名称 #，指向文档内某个具体的元素。这个被链接的元素就是目标元素(target element)。`:target`选择器可用于选取当前活动的目标元素。

在 :target 触发的时候分别去操作相关 DOM。

```html
<ul class="nav">
	<li><a href="#tab1">tab1</a></li>
	<li><a href="#tab2">tab2</a></li>
</ul>
<div id="tab1">111111111111</div>
<div id="tab2">222222222222</div>
```

```css
#tab1,#tab2{
	display: none;
}
#tab1:target,#tab2:target{
	display: block;
}
```

![](/css-magic-skill/2-1.gif)

至此，两个问题都已经解决，剩下的就是一些样式的修补工作。

## 自定义文字下划线

使用`linear-gradient`定义下划线

```html
<span class="wave">南月凡歆</span>
<span class="dashed">南月凡歆</span>
```

```css
span{
	position: relative;
	display: inline-block;
}
span:after{
	content: '';
	display: block;
	position: absolute;
	top: 110%;
	width: 100%;
	height: 3px;
}

/**
 *  波浪线，渐变构造'X'，截取上半部分，得到一个'V',从而结合 repeat 形成波浪线 
 */
span.wave:after {  
	background: linear-gradient(135deg, transparent, transparent 45%, red, transparent 55%, transparent 100%),
	linear-gradient(45deg, transparent, transparent 45%, red, transparent 55%, transparent 100%);
	background-size: 6px 6px;

}
span.dashed:after {
	background: linear-gradient(90deg,#03A9F4 50%,transparent 0);
	background-size: 6px 6px;
}
```

![](/css-magic-skill/3-1.png)

## 利用 pointer-events 禁用事件效果

`pointer-events`：字面理解是鼠标事件。

当使用 pointer-events:none 时，表示它将捕获不到任何事件，而只是让事件穿透到它的下面。

>场景：在做`tab`切换的时候，当选中当前项，禁用当前标签的事件，只有切换其他`tab`的时候，才重新请求新的数据。

```html
<ul>
    <li>
        <a class="tab active" href="https://google.com">google</a>
    </li>
    <li>
         <a class="tab" href="https://facebook.com">facebook</a>
    </li>
    <li>
        <a class="tab" href="https://stackoverflow.com">stackoverflow</a>
     </li>
</ul>
```

```css
.active{
	pointer-events: none;
}
```

适合 pointer-events 的场景有很多。禁用事件；作为覆盖物不影响下层操作，遇到这类情况都可以考虑喲。

## rem 布局

移动端用 rem 布局时候，根据不同的屏幕宽度要设置不同的 font-size 来做到适配。
>例如：以`750px`设计稿作为基准，根节点设置 font-size 为 100px，只考虑 DPR 为 2 的最简情况。

 ```javascript
 document.querySelector('html').style.fontSize = `${window.innerWidth / 7.5 }px`;
 ```

等同与

```css
html{
font-size: calc(100vw  /  7.5)
}
```

## 三角形

在不使用图片的情况，实现一个简单的三角形箭头

```html
<div id="triangle-top"></div>
```

```css
#triangle-top {
	width: 0;
	height: 0;
	border-left: 30px solid transparent;
	border-right: 30px solid transparent;
	border-bottom: 60px solid red;
}
```

![](/css-magic-skill/6-1.png)


## 文本溢出省略

有时我们会遇到这样的需求，将一段文字单行，或者多行显示，超出的部分用省略号代替。相信很多同学都曾用 javascript 截取字符串拼接省略号以达到这样的效果，但由于每个字实际表现宽度的不一致，往往需要去单独的控制截取长度。

但其实，用 css 就能完美的解决。

```html
<div>借一盏午夜街头 昏黄灯光 照亮那坎坷路上人影一双 借一寸三九天里 冽冽暖阳 融这茫茫人间刺骨凉</div>
```

### 单行文本

```css
div{
	width: 300px;
	overflow : hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
}
```

![](/css-magic-skill/7-1.png)

### 多行文本

```css
div{
	width: 300px;
	overflow : hidden;
	text-overflow: ellipsis;
	display: -webkit-box;
        -webkit-line-clamp: 2;
        -webkit-box-orient: vertical;
}
```

![](/css-magic-skill/7-2.png)
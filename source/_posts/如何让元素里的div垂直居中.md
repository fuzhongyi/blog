---
title: 如何让元素里的div垂直居中
date: 2017-09-08 17:35:58
tags: 
    - css
    - 居中
categories : 技术水波文
---

{% fi /如何让元素里的div垂直居中/header-img.jpg %}

## 已知宽高元素
{% note info %}
绝对定位与负边距实现。利用绝对定位，将元素的top和left属性都设为50%，再利用margin边距，将元素回拉它本身高宽的一半，实现垂直居中。
{% endnote %}
``` css
#container {
    position:relative;
}

#div {
    position:absolute;
    width: x;
    height: y;
    top: 50%;
    left: 50%;
    margin: -x / 2 0 0 -y / 2;
}
```
<!-- more -->
## 未知宽高元素
{% note info %}
### 方法一
使用绝对定位与margin。
{% endnote %}
```css
#container {
    position:relative;
}

#div {
    position: absolute;
    margin: 0 auto;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
}
```
{% note info %}
### 方法二
当要被居中的元素是内联元素的时候，将父级容器设置为display:table-cell，配合text-align:center和vertical-align:middle即可以实现水平垂直居中。
{% endnote %}
```css
#container {
    display:table-cell;
    text-align:center;
    vertical-align:middle;
}
```
{% note info %}
### 方法三
使用css3的transform。
{% endnote %}
```css
#container {
    position:relative;
}
 
#div {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}
```
{% note info %}
### 方法四
使用flex布局。
{% endnote %}
```css
#container {
    display:flex;
    justify-content:center;
    align-items: center;
}
```
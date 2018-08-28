---
title: CSS 代码书写规范
date: 2018-08-28 11:43:43
tags: css
categories: 技术水波文 
---

![](/css-code-writing-standard/header-img.jpg)

# 文件

在样式文件的第一行首个字符位置写上 @charset 规则，编码名用 “UTF-8”。

```css
@charset "UTF-8";

.xyue{}
```

# 格式化

样式书写一般有两种：

+ 展开格式 (Expanded)

```css
.xyue {
    display: block;
    width: 100px;
}
```

+ 紧凑格式 (Compact)

```css
.xyue {display: block;width: 100px;}
```

使用展开格式书写样式

# 大小写

样式选择器，属性名，属性值关键字均使用小写字母书写，属性字符串允许使用大小写。

```css
/* YES */ 
.xyue {
    font-family: 'Microsoft YaHei UI';
}

/* NO */
.XYUE {
    DISPLAY: BLOCK;
}
```
# 选择器

1. 尽量少用通用选择器 *
2. 不使用 ID 选择器
3. 不使用无具体语义定义的标签选择器

```css
/* YES */
.xyue {}
.xyue li {}
.xyue li p{}

/* NO */
*{}
#xyue {}
.xyue div{}
```

# 代码易读性

## 每个属性声明末尾都要加分号

```css
/* YES */
.xyue { 
    width: 100%; 
}

/* NO */
.xyue{ 
    width: 100%
}
```

## 左括号与类名之间一个空格，冒号与属性值之间一个空格

```css
/* YES */
.xyue { 
    width: 100%; 
}

/* NO */
.xyue{ 
    width:100%;
}
```

## 逗号分隔的取值，逗号之后一个空格

```css
/* YES */
.xyue {
    box-shadow: 1px 1px 1px #000, 2px 2px 2px #999;
}

/* NO */
.xyue {
    box-shadow: 1px 1px 1px #000,2px 2px 2px #999;
}

```

## 为单个css选择器或新申明开启新行

```css
/* YES */
.xyue, 
.xyue_logo, 
.xyue_hd {
    color: #ff0;
}
.nav{
    color: #fff;
}

/* NO */
.xyue,.xyue_logo,.xyue_hd {
    color: #ff0;
}.nav{
    color: #fff;
}
```

## 颜色值 `rgb()` `rgba()` `hsl()` `hsla()` `rect()` 中不需有空格，且取值不要带有不必要的 0

```css
/* YES */
.xyue {
    color: rgba(255,255,255,.5);
}

/* NO */
.xyue {
    color: rgba( 255, 255, 255, 0.5 );
}
```

## 属性值十六进制数值能用简写的尽量用简写

```css
/* YES */
.xyue {
    color: #fff;
}

/* NO */
.xyue {
    color: #ffffff;
}
```

## 不要为 0 指明单位

```css
/* YES */
.xyue {
    margin: 0 10px;
}

/* NO */
.xyue {
    margin: 0px 10px;
}
```

# 属性值引号

```css
/* YES */
.xyue { 
    font-family: 'Microsoft YaHei UI';
}

/* NO */
.xyue { 
    font-family: "Microsoft YaHei UI";
}
```

# 属性书写顺序

遵循以下顺序：

1. 布局定位属性：display / position / float / clear / visibility / overflow
2. 自身属性：width / height / margin / padding / border / background
3. 文本属性：color / font / text-decoration / text-align / vertical-align / white- space / break-word
4. 其他属性（CSS3）：content / cursor / border-radius / box-shadow / text-shadow / background:linear-gradient … 


 ```css
.xyue {
    display: block;
    position: relative;
    float: left;
    width: 100px;
    height: 100px;
    margin: 0 10px;
    padding: 20px 0;
    font-family: Arial, 'Helvetica Neue', Helvetica, sans-serif;
    color: #333;
    background: rgba(0,0,0,.5);
    -webkit-border-radius: 10px;
    -moz-border-radius: 10px;
    -ms-border-radius: 10px;
    -o-border-radius: 10px;
    border-radius: 10px;
}
```
 ***CSS3 浏览器私有前缀在前，标准前缀在后***



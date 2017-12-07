---
title: ionic3启动白屏解决
date: 2017-11-27 17:25:06
tags: ionic
categories: 问题记录
---

![](/ionic-start-white-screen/header-img.png)

## 问题描述

APP 启动后，会有一段时间的白屏，影响体验

## 解决

### 第一步

向 `config.xml` 配置文件中添加以下内容

``` xml
<preference name="AutoHideSplashScreen" value="false" />    <!-- 禁止自动隐藏 -->
<preference name="ShowSplashScreen" value="true" />         <!-- 显示启动画面 -->
<preference name="ShowSplashScreenSpinner" value="true" />  <!-- 显示启动加载灰圈 -->
<preference name="FadeSplashScreen" value="true" />         <!-- 启动淡出效果 -->
```


### 第二步

向 `/src/app/main.ts` 文件中添加以下内容

``` typescript
import { enableProdMode } from '@angular/core';
enableProdMode();// 加快启动速度
```

---
完成以上步骤重新 build.
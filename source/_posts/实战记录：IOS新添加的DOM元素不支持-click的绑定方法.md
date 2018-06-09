---
title: 实战记录：IOS新添加的DOM元素不支持.click的绑定方法
date: 2017-08-12 10:30:11
tags: [IOS兼容性,javascript,JQuery]
---

​    周末来加班准备下周项目从线工作，坐着等QA和审核等部门的问题解决，目前帝都的外边下着忽大忽小的雷阵雨， 昨天下班在雨中漫步回家，今天早起去体检又雨中漫步来上班，有一种说不出的感觉；好了天气预报说完了，下班就来说说我在上一片文章提到正在做的项目遇到的一个小问题，这里和大家分享下；

 就像题目看到的在ios中动态加入的dom元素是不支持click绑定方法的，当时遇到这个问题我没有细想直接换成了touchstart来检测，发现可以通过，由于项目紧张就没有详查原因，记录下来今天做了一个查询：

大概意思是,如果要用$(document).click()来绑定新生成的DOM元素,必须给此DOM添加css样式

```
cursor: pointer;
```

或用touchstart方法；我并没有亲测css样式这个解决 方案，这里做个记录，下次遇到用一下试试！如下是解答地址：

http://stackoverflow.com/questions/3705937/document-click-not-working-correctly-on-iphone-jquery




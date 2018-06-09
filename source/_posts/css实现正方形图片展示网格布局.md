---
title: css实现正方形图片展示网格布局
date: 2017-08-16 14:00:17
tags: [css,移动端]
---

 这两周手里的项目第一期0的阶段已经基本结束了，后边的迭代还没有开始，所以这两天闲来没事在梳理在做项目时遇到的问题和需要优化的点，昨天晚上加班点击做过的东西发现一个可优化的问题，之前没有做是因为时间紧任务重，所以用了一个比较暴力的方法暂时解决了，但是当时我思考中有一个思路觉得可以解决却没有用，所以就用了一点时间解决这个问题，下边就来说说场景：

在app里有个列表页和详情页，列表页大概就和朋友圈布局差不多，有内容和图片，详情页里只是多了个评论，列表的页面是原生的，详情页是h5的，下面问题来了，在原生里图片是正方形的，不管我上传什么样的图片都是正方形，看上去是图片裁剪，但实际上如果宽>高这个正方形里展示的是图片高度100%，宽度截取然后展示平铺展示，这样看没有变形效果；我们在h5页面当时没有做太多想法，因为公司用的是七牛云的图片存贮，所以用了七牛云的API里的裁剪，直接硬生生的搞成了正方向，就变成了如下效果：

app详情效果

<!--more-->

![1](\images\css正方形\1.png)

h5详情页用七牛云裁剪后的效果

![2](\images\css正方形\2.png)

有木有很丑，裁剪的很生硬，由于是移动端所以宽度是不固定的，发现这个问题时我第一个闪现的思路就是用flex布局，内部再给个据对定位，在用tranform做个居中显示，但是再细节一点没有多想，所以就用了七牛云的裁剪先上了，昨天想到这个问题，我就按照我的思路做了一个小demo，发现思路对了，但平铺不了还是有一点点不完美，所以有了之后的不断思考和常识，找到了最终完美的解决方案：

<p data-height="265" data-theme-id="0" data-slug-hash="BdmVLo" data-default-tab="result" data-user="it_hack" data-embed-version="2" data-pen-title="css 正方形图片布局" class="codepen">See the Pen <a href="https://codepen.io/it_hack/pen/BdmVLo/">css 正方形图片布局</a> by kevin (<a href="https://codepen.io/it_hack">@it_hack</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

在这种布局下，每个容器的宽高比就越接近图片的真实宽高比，多数图片都能显示出其主要部分。

这里用到了一个css不常见的属性object-fit；对于这个属性的具体用法推荐大家可以到张鑫旭大大的博客中学习查看，附上地址[半深入理解CSS3 object-position/object-fit属性](半深入理解CSS3 object-position/object-fit属性)

目前还没发现我这个css实现移动端正方形图片显示有什么问题，还在为自己能解决这个问题而激动中……后续如果有问题或需要优化的希望大家可以提出建议，我发现问题的话也会随时更新问题和优化方案；
---
title: vue中Swipe插件在data的值更新后为什么不会重新更新dom？
date: 2017-05-08 17:59:33
tags: [vue,随笔]
---

 今天记录个随笔，希望大家在应用vue各种插件构成中少踩一些坑；

上周做公司内部一个累死数据看板的页面，在原来vue的后台中加了一个组件；这个组件其实就是一个轮播效果其他的没什么太多东西，本来以为是so easy的事，谁想到遇到了个问题，一开始没想明白，用了一天时间我基本搞明白问题，今天记录下来，希望亲们有遇到相同问题时先考虑是不是我这次踩的坑；

我在vue轮播组件式选用了[vue-awesome-swiper](https://surmon-china.github.io/vue-awesome-swiper/),这个vuejs组件是基于[swiper](http://www.swiper.com.cn/)开发的；至于这个组件是怎么用的可到github上查看这里不做详细介绍，下面上我的代码

```
<template>
<span>{{activeIndex}}---{{in}}</span>
   <swiper :options="swiperOption" class="swiperbox" :class="{swiper2:activeIndex==1,swiper3:activeIndex==2}">
        <swiper-slide class="swiperItem" v-for="(keyP,itemBox) in swiperTitle">{{activeIndex}}---{{in}}
        </swiper-slide> 
       <div class="swiper-pagination" slot="pagination"></div>
   </swiper>
</template>

return {
       swiperOption: {
         height: 300,
         autoplay: false,//20000
         loop : true,
         autoplayDisableOnInteraction : false,
         setWrapperSize :true,
         pagination : '.swiper-pagination',
         paginationClickable :true,
         mousewheelControl : true,
         observer:true,
         observeParents:true,
         onSlideChangeStart: function(swiper){
           vm.activeIndex=swiper.realIndex;
           vm.in=swiper.activeIndex;
           console.log(vm.activeIndex,vm.in)
         }
       },
       activeIndex:0
}
```

这段轮播看上去没什么问题，假如我们有3个slide索引分别是：0=>1=>2，我们手动切换会发现0=>1=>2正序切换怎么切都ok，当我们从0=>2时出问题了，在onSlideChangeStart回调中activeIndex和in控制台是更新了索引的，但是在dom中swiper-slide里的dom数据是未更新的，而swiper外的dom是更新的，这是怎么回事呢？我google后发现[segmentfault](https://segmentfault.com/q/1010000009147101/a-1020000009309928)有相同的问题，有人回答说要配置swiper的观察者模式为true，我想说这是分情况的，这是针对组件数据是异步进去时开启观察者管用，但我这个情况是不起作用的，经过多方考证发现这是swiper的loop问题，开启了loop的话，swiper只是自己在dom上自己复制了两个节点进去，但是vm是不知道这个节点状态的，所以导致在0=>2时出现不更新dom的问题，我最后的解决办法是舍弃了loop这个功能，反正开了自动轮播，手动切换就关了loop吧；

希望大家在vue中用swiper时注意一下这个loop问题，尽量绕开loop;



如上文章有点潦草，还望谅解；有问题随时艾特本人；
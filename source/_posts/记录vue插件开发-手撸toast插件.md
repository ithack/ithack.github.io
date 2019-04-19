---
title: 记录vue插件开发-手撸toast插件
date: 2019-01-17 17:51:32
tags: [vue,笔记,toast]
---

之前有个项目比较赶，我过去支援开发时发现一个toast插件，写成公用方式可以直接`this.$toast()`调用；但由于写这个公用插件的人是个新手，所以把插件写成了组件，每次想用toast都要`import toast from './components/toast'`引入组件，奈何已经成型，所以今天把关于VUE写插件的方法分享出来，貌似网上也有类似文章，但我作为非专业程序猿，一贯作风是写出来让自己记忆深刻也能没事拿出来看看！

词穷代码，代码来补！关于用到的vue方法可直接[官网查看](https://cn.vuejs.org/)！

如果觉得不错的话，点点关注，点点赞，谢谢大家，你们的支持！下面进入正文；

### 正文

首先既然是插件，我的理解就是只需要调用传值即可实现，想toast这么常用的插件我们不可能每一个组件都引用文件（不管是js还是vue文件）都是多余！

首先，我们需要创建一个toast.vue的模版组件内容如下:

<!-- more -->

```
<template>
  <div v-show='visible' class='toast'>
    <i>{{message}}</i>
  </div>
</template>

<script>
  export default {
    name: "toast",
    data(){
      return {
        message:"",
        visible:true
      }
    }
  }
</script>

<style lang='less' scoped>
  .toast {
    position: fixed;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%) scale(1);
    word-wrap: break-word;
    text-align: center;
    z-index: 888;
    max-width: 80%;
    color: #fff;
    border-radius: 20px;
    padding: 30px 34px;
    background: rgba(0, 0, 0, 0.7);
    overflow: hidden;
  }
</style>
```

组件很简单，就是使用vue的数据绑定，默认显示toast内容

下面我们就要正式写vue的插件了，请注意，前方高能>>>>>

```javascript
import toast from './toast.vue'
let Toast={};
Toast.install=(Vue,option={})=>{
  const ToastTem=Vue.extend(toast);
  let removeDom = event => {
    if (event.target.parentNode.childNodes.length > 1) {
      event.target.parentNode.removeChild(event.target)
    } else {
      event.target.parentNode.parentNode.removeChild(event.target.parentNode)
    }
  }
  // 实现toast的关闭方法
  ToastTem.prototype.close=function(){
    this.visible=false;
    this.$el.addEventListener('transitionend',removeDom)
  }
  // 用户可以在Vue实例（Vue单文件就是一个Vue实例）通过this.$toast来访问以下内容
  Vue.prototype.$toast=(option)=>{
    // toast实例挂载到刚创建的div
    var instance = new ToastTem().$mount(document.createElement('div'));
    let duration = option.duration || 3000;
    instance.message = typeof option==='string'? option :option.message;
    // 将toast的DOM挂载到DOM上
    document.body.appendChild(instance.$el);
    // 自动关闭功能的实现
    setTimeout(function () {
      instance.close()
    }, duration)
  }
}
export default Toast
```

接下来，你就可以在入口文件进行使用刚刚开发的插件啦!

toast用法：

```javascript
//入口文件引入toast
import Toast from './plugins/toast.js';
import Vue from 'vue'

Vue.use(Toast)

//组件中直接调用
this.$toast('toast测试内容')
```

好了，toast插件写完了，可以在任何组件中直接用`this.$toast()`是不是很方便！



如有问题，欢迎大家指正缺陷！如果喜欢请留下你的脚步！！！！


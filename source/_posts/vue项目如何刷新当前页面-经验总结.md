---
title: vue项目如何刷新当前页面-经验总结
date: 2019-06-13 10:35:54
tags: [vue]
---

### 前言

很久没有更新了，今天这个题目更多的可能是一点灵感，在最近项目中有一个刷新当前页面需求，但通过思考也并不是刷新当前页面，这样做固然简单，但对于用户体验并不是很完美，其实只需要刷新当前路由组件，而页面中整体框架不需要重新渲染，所以就考虑有没有办法只重新加载某一个组件，这样用户只会看到数据或某一块的刷新，有点像ajax的方式进行局部刷新，会提高一下用户体验；

### 正文



vue项目刷新当前页面，可能这已经不是个问题，目前大家知道的有应该最少有两种解决方案：

```
localhost.reload() 

或

this.$router.go(0)
```
如上是在遇到刷新时最直接能想到的方法，但是这个方法有个弊端就是刷新页面会导致空白页面的出现，这样的用户体验很不好；
![image](/images/2019/59399155-93d82300-8dc5-11e9-94ab-a23a5e0c8b3e.png)

<!-- more -->如上这样一个布局我们只需要重新让“main”区域组件进行重新reload，这时恰巧这个区域是一个router-view组件进行渲染的，那么我们提高一下用户体现可以用如下方法实现：

```
<div id="app">
    <el-container>
      <el-header>
        <Header></Header>
      </el-header>
      <el-row :gutter="20" class="zvip-cont">
        <el-col :span="4" class="zvip-left">
          <Aside />
        </el-col>
        <el-col :span="20" class="zvip-right">
          <router-view v-if="isReload"></router-view> //此处进行判断渲染
        </el-col>
      </el-row>
    </el-container>
  </div>
provide() { // 利用vue的provide方法
    return {
      reload: this.reload
    }
  },
  data() {
    return {
      isReload: true
    }
  },
methods: {
    reload() {
      this.isReload = false
      this.$nextTick(function() {
        this.isReload = true
      })
    }
  }
```
然后在需要当前页面刷新的页面中注入App.vue组件提供（provide）的 reload 依赖，然后直接用this.reload来调用就行

```
export default {
  inject: ["reload"],
  methods: {
    routerReload(val) {
      this.reload()
    },
  }
}
```
在众多刷新的方法里这个方法目前是个人觉得体验最好的一种，当然可能还有其他的方式，但目前我还没有想到到，欢迎交流！！
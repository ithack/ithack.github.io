---
title: Vue奇淫技笔记
date: 2018-09-26 15:43:22
tags: [vue,随笔]
---

​	vue可视化项目暂时告一段落了，后续整理项目结构会更新上来，今天闲来无事先更新一把vue一些奇淫笔记，之所以称之为奇淫，可能是因为觉得有意思，也可能是觉得比较好玩，并不一定建议开发中用，这些奇淫的技能有些可能透过代码熟悉vue的运行机制等问题；

##### 奇淫笔记一：

​	如果你只在子组件里面改变父组件的一个值，不妨试试 `$emit('input')`,会直接改变 v-model
我们正常的父子组件通信是 父组件通过 props 传给子组件，子组件通过 `this.$emit('eventName',value)`通知父组件绑定在 @eventName 上的方法来做相应的处理。
​	但是这边有个特例，vue 默认会监听组件的 input 事件，而且会把子组件里面传出来的值，赋给当前绑定到 v-model 上的值
**正常用法 - 父组件**

```javascript
<template>
  <subComponent :data="param" @dataChange="dataChangeHandler"></subComponent>
</template>

<script >
  export default {
    data () {
      return {
        param:'xxxxxx'
      }
    },
    methods:{
      dataChangeHandler (newParam) {
        this.param = newParam
      }
    }
  }
</script>
```

<!--more-->

**正常用法 - 子组件**

```javascript
<script >
  export default {
    methods:{
      updateData (newParam) {
        this.$emit('dataChange',newParam)
      }
    }
  }
</script>
```

**利用默认 input 事件 - 父组件**

```javascript
<template>
  <subComponent  v-model="param"></subComponent>
</template>
```

**利用默认 input 事件 - 子组件**

```javascript
<script >
  export default {
    methods:{
      updateData (newParam) {
        this.$emit('input',newParam)
      }
    }
  }
</script>
```

这样，我们就能省掉父组件上的一列席处理代码，vue 会自动帮你处理好

这种方法只适用于改变单个值的情况，且子组件对父组件只需简单的传值，不需要其他附加操作(如更新列表)的情况。

补充一个 `this.$emit('update:fidldName',value)`方法

具体用法如下:
父组件
`<subComponent field1.sync="param1" field2.sync="param2"></subComponent>`

子组件

```javascript
<script >
  export default {
    methods:{
      updateData1 (newValue) {
        this.$emit('update:field1',newValue)
      },
      updateData2 (newValue) {
        this.$emit('update:field2',newValue)
      }
    }
  }
</script>
```

该方法,个人认为比较适用于 要更新的数据不能绑定在 v-model 的情况下,或者要双向通信的数据大于 1 个(1个也可以用,但我个人更推荐 input 的方式, 看个人喜好吧),但又不会很多的情况下.

##### 奇淫笔记二：

非父子组件传递数据；通过使用一个空的Vue实例作为中央事件总线。**index.js**

```javascript
let bus = new Vue()
Vue.prototype.bus = bus;
```

相邻组件1,通过$emit()传递数据
`this.bus.$emit("toChangeTitle","首页");`
相邻组件2，通过$on()接收数据

```javascript
mounted(){   
   this.bus.$on('toChangeTitle', function (title) {
       console.log(title)
  })
}
```

##### 奇淫笔记三：

vue中`@click="event()"`，添加()与不添加()的区别应该是 Vue 对函数调用表达式额外用了一个函数做了层包装。加与不加括号的区别在于事件对象参数 event 的处理。不加括号时，函数第一个参数默认为 event；加了括号后，需要手动传入 $event 才能获得事件对象。

```javascript
<template>
    <div class="btn-item">
        <button class="btn btn-success" @click="sure($event)">确定</button>
        <button class="btn btn-default" @click="quit">取消</button>
    </div>
</template>

<script>
  export default{
      name:'test',
      data(){
          return {

          }
      },
      methods:{
          sure(e){
               console.log(e.target);
          },
          quit(e){
              console.log(e.target);
          }
      }
  }
</script>
```

**======================华丽的分割线====================**

//181017更新

##### 奇淫笔记四：

vue单页路由跳转后scrollTop问题：当一个页面滚动了，点击页面上的路由调到下一个页面时，跳转后的页面也是滚动的，滚动条并不是在页面的顶部！

可能有的小伙伴加上window.scrollTop(0,0);来解决问题，但是这个太繁琐了；

又或者在入口加入：

```
router.afterEach(function (to) {
  window.scrollTo(0, 0)
})
```

路由跳转后就不会出现滚动的问题了。但是这种做法是不友好的！我们可以使用scrollBehavior (to, from, savedPosition) {}来解决问题。

在我们写路由的时候做个如下处理：

```
import Vue from 'vue'
import Router from 'vue-router'
Vue.use(Router);

Vue.use(Router)

export default new Router({
  routes: [
  	……
  ],
  scrollBehavior (to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition
    } else {
      return { x: 0, y: 0 }
    }
  }
})
```



scrollBehavior 方法接收 to 和 from 路由对象。第三个参数 savedPosition 当且仅当 popstate 导航 (通过浏览器的 前进/后退 按钮触发) 时才可用。它的使用有很多种，可以试试。

##### 奇淫笔记五

深度作用选择器,场景：scoped的样式，希望影响到子组件的默认样式；

在样式中设置完scoped在浏览器解析为如下图这样，a是个div，a div里面包含一个组件里面解析完了div的样式名字为b，想在父组件影响到子组件的默认样式。

```
<div data-v-fcaef538 class="a">
	<div data-v-35b15bcb class="b"></div>
</div>
```

解决办法：

```
<style scoped>
    .a >>> .b { /* ... */ }
</style>
    //有些像Sass之类的预处理器无法正确解析>>>。这种情况下你可以使用/deep/操作符取而代之- - - -这是一个>>>的别名，同样可以正常工作。
<style scoped lang=“scss”>
    .a /deep/ .b { /* ... */ }
</style>
```

//待更新
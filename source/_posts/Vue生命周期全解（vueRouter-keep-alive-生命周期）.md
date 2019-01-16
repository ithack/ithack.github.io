---
title: 'Vue生命周期全解（vueRouter,keep-alive,生命周期）'
date: 2018-09-26 16:11:17
tags: [vue,vue-router]
---

最近在开发一个项目时我发现，关于Vue的钩子函数，可能很多人只停留在一些很简单常用的钩子(`created`,`mounted`)，并没有仔细的去研究过加入keep-alive,vue-router后的执行顺序等问题；

##### Vue-Router导航守卫

有的时候，我们需要通过路由来进行一些操作，比如最常见的登录权限验证，当用户满足条件时，才让其进入导航，否则就取消跳转，并跳到登录页面让其登录。

为此我们有很多种方法可以植入路由的导航过程：**全局的, 单个路由独享的, 或者组件级的**,推荐优先阅读[路由文档](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html)

##### 关于钩子的一些知识：

###### 路由钩子函数的错误捕获

如果我们在全局守卫/路由独享守卫/组件路由守卫的钩子函数中有错误，可以这样捕获：

```
router.onError(callback => { 
// 2.4.0新增 并不常用，了解一下就可以了 
  console.log(callback, 'callback');
});
```

在路由文档中还有更多的[实例方法](https://router.vuejs.org/zh/api/#router-%E5%AE%9E%E4%BE%8B%E6%96%B9%E6%B3%95)：动态添加路由等，有兴趣可以了解一下。

<!-- more -->

###### 跳转死循环，页面永远空白

我了解到的，很多人会碰到这个问题，来看一下这段伪代码：

```
router.beforeEach((to, from, next) => {
  if(登录){
     next()
  }else{
      next({ name: 'login' }); 
  }
});
```

看逻辑貌似是对的，但是当我们跳转到`login`之后，因为此时还是未登录状态，所以会一直跳转到`login`然后死循环，页面一直是空白的，所以：我们需要把判断条件稍微改一下。

```
if(登录 || to.name === 'login'){ next() } // 登录，或者将要前往login页面的时候，就允许进入路由
```

###### 全局后置钩子的跳转：

文档中提到因为router.afterEach不接受`next`函数所以也不会改变导航本身，意思就是只能当成一个钩子来使用，但是我自己在试的时候发现，我们可以通过这种形式来实现跳转：

```
// main.js 入口文件
import router from './router'; // 引入路由
router.afterEach((to, from) => {
  if (未登录 && to.name !== 'login') {
    router.push({ name: 'login' }); // 跳转login
  }
});
```

额，通过router.beforeEach 也完全可以实现且更好，我就骚一下。

##### 完整的路由导航解析流程(不包括其他生命周期)：

1. 触发进入其他路由。
2. 调用要离开路由的组件守卫`beforeRouteLeave`
3. 调用局前置守卫：`beforeEach`
4. 在重用的组件里调用 `beforeRouteUpdate`
5. 调用路由独享守卫 `beforeEnter`。
6. 解析异步路由组件。
7. 在将要进入的路由组件中调用`beforeRouteEnter`
8. 调用全局解析守卫 `beforeResolve`
9. 导航被确认。
10. 调用全局后置钩子的 `afterEach` 钩子。
11. 触发DOM更新(`mounted`)。
12. 执行`beforeRouteEnter` 守卫中传给 next 的回调函数



------
##### 你不知道的keep-alive[我猜你不知道]

在开发Vue项目的时候，大部分组件是没必要多次渲染的，所以Vue提供了一个内置组件`keep-alive`来**缓存组件内部状态，避免重新渲染**，[文档在这里](https://cn.vuejs.org/v2/api/#keep-alive)。

> 文档：和 `<transition>`相似，`<keep-alive>` 是一个抽象组件：它自身不会渲染一个 DOM 元素，也不会出现在父组件链中。

###### 用法：

**缓存动态组件：**

`<keep-alive>`包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们，此种方式并无太大的实用意义。

```
<!-- 基本 -->
<keep-alive>
  <component :is="view"></component>
</keep-alive>

<!-- 多个条件判断的子组件 -->
<keep-alive>
  <comp-a v-if="a > 1"></comp-a>
  <comp-b v-else></comp-b>
</keep-alive>
```

**缓存路由组件：**

使用`keep-alive`可以将所有路径匹配到的路由组件都缓存起来，包括路由组件里面的组件，`keep-alive`大多数使用场景就是这种。

```
<keep-alive>
    <router-view></router-view>
</keep-alive>
```

###### 生命周期钩子：

这篇既然是Vue钩子函数的专场，那肯定要扣题呀~

在被`keep-alive`包含的组件/路由中，会多出两个生命周期的钩子:`activated` 与 `deactivated`。

> 文档：在 2.2.0 及其更高版本中，activated 和 deactivated 将会在 树内的**所有嵌套组件**中触发。

**activated在组件第一次渲染时会被调用，之后在每次缓存组件被激活时调用**。

**activated调用时机：**

第一次进入缓存路由/组件，在`mounted`后面，`beforeRouteEnter`守卫传给 next 的回调函数之前调用：

```
beforeMount=> 如果你是从别的路由/组件进来(组件销毁destroyed/或离开缓存deactivated)=>
mounted=> activated 进入缓存组件 => 执行 beforeRouteEnter回调
```

因为组件被缓存了，**再次进入缓存路由/组件时，不会触发这些钩子**：

```
// beforeCreate created beforeMount mounted 都不会触发。
```

所以之后的调用时机是：

```
组件销毁destroyed/或离开缓存deactivated => activated 进入当前缓存组件 
=> 执行 beforeRouteEnter回调
// 组件缓存或销毁，嵌套组件的销毁和缓存也在这里触发
```

**deactivated：组件被停用(离开路由)时调用**

**使用了keep-alive就不会调用beforeDestroy(组件销毁前钩子)和destroyed(组件销毁)，因为组件没被销毁，被缓存起来了**。

这个钩子可以看作`beforeDestroy`的替代，如果你缓存了组件，要在组件销毁的的时候做一些事情，你可以放在这个钩子里。

如果你离开了路由，会依次触发：

组件内的离开当前路由钩子beforeRouteLeave =>  路由前置守卫 beforeEach =>
全局后置钩子afterEach => deactivated 离开缓存组件 => activated 进入缓存组件(如果你进入的也是缓存路由)
// 如果离开的组件没有缓存的话 beforeDestroy会替换deactivated 
// 如果进入的路由也没有缓存的话  全局后置钩子afterEach=>销毁 destroyed=> beforeCreate等

那么，如果我只是想缓存其中几个路由/组件，那该怎么做？

###### 缓存你想缓存的路由：

**Vue2.1.0之前:**

想实现类似的操作，你可以：

1. 配置一下路由元信息
2. 创建两个`keep-alive`标签
3. 使用`v-if`通过路由元信息判断缓存哪些路由。


```
<keep-alive>
    <router-view v-if="$route.meta.keepAlive">
        <!--这里是会被缓存的路由-->
    </router-view>
</keep-alive>
<router-view v-if="!$route.meta.keepAlive">
    <!--因为用的是v-if 所以下面还要创建一个未缓存的路由视图出口-->
</router-view>
//router配置
new Router({
  routes: [
    {
      path: '/',
      name: 'home',
      component: Home,
      meta: {
        keepAlive: true // 需要被缓存
      }
    },
    {
      path: '/:id',
      name: 'edit',
      component: Edit,
      meta: {
        keepAlive: false // 不需要被缓存
      }
    }
  ]
});
```

** Vue2.1.0版本之后：**

使用路由元信息的方式，要多创建一个`router-view`标签，并且每个路由都要配置一个元信息，是可以实现我们想要的效果，但是过于繁琐了点。

幸运的是在Vue2.1.0之后，Vue新增了两个属性配合`keep-alive`来有条件地缓存 路由/组件。

**新增属性：**

- `include`：匹配的 路由/组件 会被缓存
- `exclude`：匹配的 路由/组件 不会被缓存

`include`和`exclude`支持三种方式来有条件的缓存路由：采用逗号分隔的字符串形式，正则形式，数组形式。

正则和数组形式，必须采用`v-bind`形式来使用。

**缓存组件的使用方式**：

```
<!-- 逗号分隔字符串 -->
<keep-alive include="a,b">
  <component :is="view"></component>
</keep-alive>

<!-- 正则表达式 (使用 `v-bind`) -->
<keep-alive :include="/a|b/">
  <component :is="view"></component>
</keep-alive>

<!-- 数组 (使用 `v-bind`) -->
<keep-alive :include="['a', 'b']">
  <component :is="view"></component>
</keep-alive>
```

**但更多场景中，我们会使用keep-alive来缓存路由**：

```
<keep-alive include='a'>
    <router-view></router-view>
</keep-alive>
```

**匹配规则：**

1. **首先匹配组件的name选项**，如果`name`选项不可用。
2. 则匹配它的**局部注册名称**。 (父组件 `components` 选项的键值)
3. **匿名组件，不可匹配**。

比如路由组件没有`name`选项，并且没有注册的组件名。

1. 只能匹配当前被包裹的组件，**不能匹配更下面嵌套的子组件**。

比如用在路由上，只能匹配路由组件的`name`选项，不能匹配路由组件里面的嵌套组件的`name`选项。

1. 文档：`<keep-alive>`**不会在函数式组件中正常工作**，因为它们没有缓存实例。
2. **exclude的优先级大于include**

也就是说：当`include`和`exclude`同时存在时，`exclude`生效，`include`不生效。

```
<keep-alive include="a,b" exclude="a">
  <!--只有a不被缓存-->
  <router-view></router-view>
</keep-alive>
```

**当组件被exclude匹配，该组件将不会被缓存，不会调用activated 和 deactivate**

------

##### 组件生命周期钩子

![](/images/vue生命周期全解/1.jpg)

这张图片已经讲得很清楚了，很多人这部分也很清楚了，大部分生命周期并不会用到，这里提一下几点：

1. **ajax请求最好放在created里面**，因为此时已经可以访问`this`了，请求到数据就可以直接放在`data`里面。
2. 这里也碰到过几次，面试官问：ajax请求应该放在哪个生命周期。
3. **关于dom的操作要放在mounted里面**，在`mounted`前面访问dom会是`undefined`。
4. 每次进入/离开组件都要做一些事情，用什么钩子：

- 不缓存：

  进入的时候可以用`created`和`mounted`钩子，离开的时候用`beforeDestory`和`destroyed`钩子,`beforeDestory`可以访问`this`，`destroyed`不可以访问`this`。

- 缓存了组件：

```
缓存了组件之后，再次进入组件不会触发`beforeCreate`、`created` 、`beforeMount`、 `mounted`，**如果你想每次进入组件都做一些事情的话，你可以放在`activated`进入缓存组件的钩子中**。

同理：离开缓存组件的时候，`beforeDestroy`和`destroyed`并不会触发，可以使用`deactivated`离开缓存组件的钩子来代替。
```



------

##### 触发钩子的完整顺序：

将路由导航、`keep-alive`、和组件生命周期钩子结合起来的，触发顺序，假设是从a组件离开，第一次进入b组件：

1. `beforeRouteLeave`:路由组件的组件离开路由前钩子，可取消路由离开。
2. `beforeEach`: 路由全局前置守卫，可用于登录验证、全局路由loading等。
3. `beforeEnter`: 路由独享守卫
4. `beforeRouteEnter`: 路由组件的组件进入路由前钩子。
5. `beforeResolve`:[路由全局解析守卫](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html#%E5%85%A8%E5%B1%80%E8%A7%A3%E6%9E%90%E5%AE%88%E5%8D%AB)
6. `afterEach`:路由全局后置钩子
7. `beforeCreate`:组件生命周期，不能访问`this`。
8. `created`:组件生命周期，可以访问`this`，不能访问dom。
9. `beforeMount`:组件生命周期
10. `deactivated`: 离开缓存组件a，或者触发a的`beforeDestroy`和`destroyed`组件销毁钩子。
11. `mounted`:访问/操作dom。
12. `activated`:进入缓存组件，进入a的嵌套子组件(如果有的话)。
13. 执行beforeRouteEnter回调函数next。

转载自：

> 作者：OBKoro1
>
> 链接：https://juejin.im/post/5b41bdef6fb9a04fe63765f1
>
> 来源：掘金
>
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


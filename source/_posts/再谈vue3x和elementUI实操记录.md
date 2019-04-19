---
title: 再谈vue3x和elementUI实操记录
date: 2019-04-19 15:11:22
tags: [vue, elementUI, 封装]
---

### 前言

​	不得不说最近很忙，996，996ICU非常火爆（当然我说的忙也是忙搬砖），然而我大JD也几乎每天都上头条，站在风口浪尖被吐槽！这里我就不说啥了，每个公司其实都不好过，就好像每个程序员都不好过一样；对于996我一直以来都觉得如果非要按时间来要求员工来工作的话那真没什么意义，有意义的事情就算公司不要求你996我想你也会去加班加点搞还毫无怨言；但如果你根本没那么多事情可做，但单位要求你996那不但对个人是损失，对公司来说也完全是坏处多益处基本没有的！就好像现在有些大公司强制996，有些部门有事情做单没到需要996来搞，那么员工不就是纯属在公司耗时间吗？而且每个人都对公司报有怨言，这种强制性不考虑现实的制度完全是懒人想出来政治套路，治标不治本（甚至连标都治不了）！

​	以上言论只是作者本人一点小小的看法，欢迎吐槽！

下面进入正题，最近我做的事情完全不需要996这种制度来强制我，我接到项目时的想法是：007都不一定能保证完成任务！大概场景如下：

​	一个后台系统从无到有开始做，更恶心的事有一些表单是完全后端下发前端要做渲染和填写的校验，当我听到这里时我内心还是平静的，单之后的一句话我就炸了，一个半月搞完！NM一个完全没有的系统到上线一个半月，后端+前端+数据录入完全让人死的节奏，倒排期的感觉；更恶心的事这个系统一共拆了六个需求，我们刚看到第一个需求就已经告诉我们这个系统要一个半月以后要，这NM是我第N次在此公司遇到这种事了，业务方就是天，业务方每一句话基本都能搞死搬砖的（真实感受）

说这这么多，明显感觉最近不止不景气，还没时间吐槽，篇幅自己都觉得长了；好了现在说说这个系统进入正文！

### VUE3框架配置搭建

​	一开始我想着用React搭建，但是和考虑到其他人使用习惯和项目紧迫性问题，我还是选择了VUE；因为大家用vue比react熟练项目这么紧最后确定用vue3+elementUI搭建，这还是我从2.0之后第一次用3.0搭建vue，当然这次搭建不是单纯的copy方式搭建，这样对我来说就毫无意义了，搭建之前我考虑了一下多人开发场景，加入了如下插件：

<!-- more -->

- prettier 用来格式化代码，[webstorm配置](https://www.yuque.com/xudong/gkhpg3/gancb2)
- commitlint 用来标准化git代码提交
- pre-commit 提交前代码检查
- onchange  根据prettier配置保存文件时自动格式化文件

如上是在eslint检查之外单独做的一些配置文件和工具，不能说完全保证多人开发的个人习惯，但是像tab空2个还是4个符号的问题还是能做到强制统一的；

这里要说一下在用vue3.0脚手架搭建项目遇到的问题

**1.vue-cli3 一直运行 /sockjs-node/info?t= 的问题**

首先 sockjs-node 是一个JavaScript库，提供跨浏览器JavaScript的API，创建了一个低延迟、全双工的浏览器和web服务器之间通信通道。

服务端：sockjs-node（<https://github.com/sockjs/sockjs-node）>
客户端：sockjs-clien（<https://github.com/sockjs/sockjs-client）>

如果你的项目没有用到 sockjs，vuecli3 运行 npm run serve 之后 network 里面一直调研一个接口：<http://localhost:8080/sockjs-node/info?t=1462183700002>

作为一个有节操的程序猿，实在不能忍受，特意自己研究了下源码，从根源上关闭这个调用

- 找到/node_modules/sockjs-client/dist/sockjs.js
- 找到代码的 1605行

```
try {
// self.xhr.send(payload); 把这里注掉
} catch (e) {
	self.emit('finish', 0, '');
	self._cleanup(false);
}
```

**2.关于vue3.0的props强制配置类型和默认值校验问题**

其实这就是eslint规则配置问题，本来之前一直是.eslintrc.js里配置的，然而我发现vue3.0的配置规则多了一些不一样的东西，通过查资料发现vue3.0脚手架安装的项目包子带了eslint-plugin-vue中的规则配置放在vue文件夹下，这里说一下如何配置：

```
'rules': {  
/* for vue */  
// 禁止重复的二级键名  
// @off 没必要限制  
'vue/no-dupe-keys': 'off',  
// 禁止出现语法错误  
'vue/no-parsing-error': 'error',  
// 禁止覆盖保留字  
'vue/no-reservered-keys': 'error',  
// 组件的 data 属性的值必须是一个函数  
// @off 没必要限制   
'vue/no-shared-component-data': 'off',  
// 禁止 <template> 使用 key 属性  
// @off 太严格了  
'vue/no-template-key': 'off',  
// render 函数必须有返回值  
'vue/require-render-return': 'error',  
// prop 的默认值必须匹配它的类型  
// @off 太严格了  
'vue/require-valid-default-prop': 'off',  
  
// 计算属性必须有返回值  
  
'vue/return-in-computed-property': 'error',  
  
// template 的根节点必须合法  
  
'vue/valid-template-root': 'error',  
  
// v-bind 指令必须合法  
  
'vue/valid-v-bind': 'error',  
  
// v-cloak 指令必须合法  
  
'vue/valid-v-cloak': 'error',  
  
// v-else-if 指令必须合法  
  
'vue/valid-v-else-if': 'error',  
  
// v-else 指令必须合法  
  
'vue/valid-v-else': 'error',  
  
// v-for 指令必须合法  
  
'vue/valid-v-for': 'error',  
  
// v-html 指令必须合法  
  
'vue/valid-v-html': 'error',  
  
// v-if 指令必须合法  
  
'vue/valid-v-if': 'error',  
  
// v-model 指令必须合法  
  
'vue/valid-v-model': 'error',  
  
// v-on 指令必须合法  
  
'vue/valid-v-on': 'error',  
  
// v-once 指令必须合法  
  
'vue/valid-v-once': 'error',  
  
// v-pre 指令必须合法  
  
'vue/valid-v-pre': 'error',  
  
// v-show 指令必须合法  
  
'vue/valid-v-show': 'error',  
  
// v-text 指令必须合法  
  
'vue/valid-v-text': 'error',  
  
//  
// 最佳实践  
//  
// @fixable html 的结束标签必须符合规定  
// @off 有的标签不必严格符合规定，如 <br> 或 <br/> 都应该是合法的  
  
'vue/html-end-tags': 'off',  
  
// 计算属性禁止包含异步方法  
  
'vue/no-async-in-computed-properties': 'error',  
  
// 禁止出现难以理解的 v-if 和 v-for  
  
'vue/no-confusing-v-for-v-if': 'error',  
  
// 禁止出现重复的属性  
  
'vue/no-duplicate-attributes': 'error',  
  
// 禁止在计算属性中对属性修改  
  
// @off 太严格了  
  
'vue/no-side-effects-in-computed-properties': 'off',  
  
// 禁止在 <textarea> 中出现 {{message}}  
  
'vue/no-textarea-mustache': 'error',  
  
// 组件的属性必须为一定的顺序  
  
'vue/order-in-components': 'error',  
  
// <component> 必须有 v-bind:is  
  
'vue/require-component-is': 'error',  
  
// prop 必须有类型限制  
  
// @off 没必要限制  
  
'vue/require-prop-types': 'off',  
  
// v-for 指令的元素必须有 v-bind:key  
  
'vue/require-v-for-key': 'error',  
  
//  
// 风格问题  
//  
// @fixable 限制自定义组件的属性风格  
// @off 没必要限制  
  
'vue/attribute-hyphenation': 'off',  
  
// html 属性值必须用双引号括起来  
  
'vue/html-quotes': 'error',  
  
// @fixable 没有内容时，组件必须自闭和  
  
// @off 没必要限制  
  
'vue/html-self-closing': 'off',  
  
// 限制每行允许的最多属性数量  
  
// @off 没必要限制  
  
'vue/max-attributes-per-line': 'off',  
  
// @fixable 限制组件的 name 属性的值的风格  
  
// @off 没必要限制  
  
'vue/name-property-casing': 'off',  
  
// @fixable 禁止出现连续空格  
  
// TODO: 李德广  触发 新版本 typeerror：get 'range' of undefined  
  
// 'vue/no-multi-spaces': 'error',  
  
// @fixable 限制 v-bind 的风格  
  
// @off 没必要限制  
  
'vue/v-bind-style': 'off',  
  
// @fixable 限制 v-on 的风格  
  
// @off 没必要限制  
  
'vue/v-on-style': 'off',  
  
// 定义了的 jsx element 必须使用  
  
'vue/jsx-uses-vars': 'error'  
  
```

如上是关于eslint-plugin-vue中的配置API说明，可以按照配置eslint的方式配置如下.eslintrc.js配置文件

```
module.exports = {
  root: true,

  env: {
    node: true,
    browser: true,
    jquery: true
  },

  extends: ["plugin:vue/essential", "@vue/prettier"],// "plugin:vue/strongly-recommended"

  rules: {
    "no-console": "off",
    "no-debugger": "off",
    semi: [0, "always"],
    "vue/require-prop-types": "off", // prop 取消必须有类型限制
    "vue/require-v-for-key": "off", // v-for 指令的元素必须有 v-bind:key
    "vue/valid-v-for": "off",
    'vue/max-attributes-per-line': 0,
  },

  parserOptions: {
    parser: "babel-eslint"
  },
}

```

更多关于eslint的配置说明请查看[Eslint安装与配置详解](http://ithack.github.io//2019/04/19/Eslint安装与配置详解.html)



### 项目开发

虽然在开发后台系统时用了elementUI库，饿了么的库颗粒比较细，我们实际操作时可能会经常遇到多个组件重复利用或者互相交互的效果比如：table+Pagination ，这里就不做说怎么封装了，封装的table已经上传到[github](https://github.com/fe-cli/vue3.0-template)

1.关于父组件用props方式传值给子组件，而父组件props数据是异步获取，导致子组件刚进来时没有这个数据，虽然在调试工具里能看到数据更新，子组件的生命周期还是无法获取到数据问题解决办法如下

```
// asyncData为异步获取的数据，想传递给子组件使用
<template>
  <div>
    父组件
    <child :child-data="asyncData" v-if="asyncData"></child>
  </div>
</template>

<script>
  import child from './child'
  export default {
    data: () => ({
      asyncData: ''
    }),
    components: {
      child
    },
    created () {
    },
    mounted () {
      // setTimeout模拟异步数据
      setTimeout(() => {
        this.asyncData = 'async data'
        console.log('parent finish')
      }, 2000)
    }
  }
</script>
```

子组件

```
<template>
  <div>
    子组件{{childData}}
  </div>
</template>

<script>
  export default {
    props: ['childData'],
    data: () => ({
    }),
    created () {
      console.log(this.childData) // 空值
    },
    methods: {
    }
  }
</script>
```

上面按照这里的解析，子组件的html中的{{childData}}的值会随着父组件的值而改变，但是created里面的却不会发生改变(生命周期问题)！如上父组件添加IF判断可解决

2.关于封装上传组件：用饿了么的上传组件，需要封装一层可以通用的遇到一个取之和赋值的问题，可以用v-model的方式解决如下：

```
<ImgUpload v-model="imageUrl"></ImgUpload>
// imageUrl 可以赋值和取值都不耽误，组件内可以用了如下方法
props:["value"] 
this.$emit("input", "")
```

关于如上v-mode方式可查看[Vue奇淫技笔记](https://ithack.github.io/2018/09/26/Vue奇淫技笔记.html)

**3.关于饿了么表单验证问题（此处重点）**

也许有人问一个表单验证怎么成重点了，刚刚也有提到，这个表单在不请求接口的时候我是不知道任何的信息的，不知道这个表单input/radio/checked也不知道label，校验表单是不是必填，是不是数字，多少字符之类的都不知道，只有接口请求回来后才知道这个表单应该渲染成什么样子，校验什么（是否必填，是否数字等）,这不但是动态表单，还是动态dom,这个问题困扰了我一周左右时间，睡觉都在想有什么办法，终于看到了一篇文章启发了我，促使我完成了所有动态生成的表单，input等字段校验的方案，代码如下：

```
/**
 * @Author: kevin
 * Date: 2019/4/12
 * Time: 16:45
 * 规则校验
 *
 */

function mobile(val) {
  if (!/^1(3|4|5|7|8)\d{9}$/.test(val)) {
    return false
  } else {
    return true
  }
}

/**
 * 非0 正整数
 * @param val
 * @returns {boolean}
 */
function integer(val) {
  if (!/^[1-9]\d*$/.test(val)) {
    return false
  } else {
    return true
  }
}
function floatNum(val) {
  if (!/^(([1-9]{1}\\d*)|([0]{1}))(\\.(\\d){0,2})?$/.test(val)) {
    return false
  } else {
    return true
  }
}
exports.install = function(Vue) {
  /**
   * 注意: 定义type 规则时 不用做非空验证
   *    只需要传入 required:true 即可
   * */
  /*验证手机号*/
  const isvalidateMobile = (rule, value, callback) => {
    if (value != null && value != "") {
      if (!mobile(value)) {
        callback(new Error("您输入的手机号不正确!"))
      } else {
        callback()
      }
    } else {
      callback()
    }
  }
  /*请输入正整数*/
  const isvalidateInteger = (rule, value, callback) => {
    if (!integer(value)) {
      callback(new Error("请输入正整数!"))
    } else {
      callback()
    }
  }
  /*请输入两位小数*/
  const isvalidateFloat = (rule, value, callback) => {
    if (value != null && value != "") {
      if (!floatNum(value)) {
        callback(new Error("请输入两位小数!"))
      } else {
        callback()
      }
    } else {
      callback()
    }
  }

  /**
   * 参数 item
   * required true 必填项
   * maxLength 字符串的最大长度
   * min 和 max 必须同时给 min < max type=number
   * type 手机号 mobile
   *   邮箱  email
   *   网址  url
   *   各种自定义类型  定义在 src/utils/validate 中  持续添加中.......
   * */

  Vue.prototype.Validate = function(item) {
    let rules = []
    if (item.required) {
      rules.push({ required: true, message: "该输入项为必填项!", trigger: "blur" })
    }
    if (item.maxLength) {
      rules.push({ min: 1, max: item.maxLength, message: "最多输入" + item.maxLength + "个字符!", trigger: "blur" })
    }
    if (item.min && item.max) {
      rules.push({
        min: item.min,
        max: item.max,
        message: "字符长度在" + item.min + "至" + item.max + "之间!",
        trigger: "blur"
      })
    }
    if (item.type) {
      let type = item.type
      switch (type) {
        case "email":
          rules.push({ type: "email", message: "请输入正确的邮箱地址", trigger: "blur" })
          break
        case "mobile":
          rules.push({ validator: isvalidateMobile, trigger: "blur" })
          break
        case "number":
          rules.push({ validator: isvalidateInteger, trigger: "blur" })
          break
        case "float":
          rules.push({ validator: isvalidateFloat, trigger: "blur" })
          break
        default:
          break
      }
    }
    return rules
  }
}

```

如上就是Validate.js,然后在main.js里引入

```
import Validate from "@U/Validate"
Vue.use(Validate)
```

在动态的form表单里使用如下：

```
<el-form-item
	:label="item.name"
  :prop="'.value'"
  :rules="Validate({ required: item.required, type: attrType[item.attributeType] })"
  >
   <el-input v-if="attrType[item.attributeType] == 'input'" v-model="item.value" />
  </el-form-item>
```

其中item就是从后端返回的json对象，这个item会告诉我需要渲染成input还是其他的控件/默认值/校验等即可实现每一个都按照后端给的渲染和校验了

如上所有问题和vue框架都已上传githua，欢迎访问交流：

[项目包]( https://github.com/fe-cli/vue3.0-template)

[Demo](https://ithack.github.io/demo/vue3xTemplate/index.html#/test)


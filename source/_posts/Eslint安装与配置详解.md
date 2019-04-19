---
title: Eslint安装与配置详解
date: 2019-04-19 10:51:36
tags: [eslint, vue, react]
---

**什么是 ESLint**

ESLint（中文站点）是一个开源的 JavaScript 代码检查工具，使用 Node.js 编写，由 Nicholas C. Zakas 于 2013 年 6 月创建。ESLint 的初衷是为了让程序员可以创建自己的检测规则，使其可以在编码的过程中发现问题而不是在执行的过程中。ESLint 的所有规则都被设计成可插入的，为了方便使用，ESLint 内置了一些规则，在这基础上也可以增加自定义规则。

**一、Eslint安装**

**1.全局安装**

如果你想使 ESLint 适用于你所有的项目，建议全局安装 ESLint

```
$ npm install -g eslint
```



初始化配置文件

```
$ eslint --init
```



**2.局部安装**

```
$ npm install eslint --save-dev
```



初始化配置文件

```
$ ./node_modules/.bin/eslint --init
```



**3.webpack中配置eslint**

需要安装eslint-loader解析.eslint文件

```
{  
  test: /\.(js|jsx|mjs)$/,  
  enforce: 'pre',  
  use: [  
    {  
      options: {  
        formatter: eslintFormatter,  
        eslintPath: require.resolve('eslint'),  
      },  
    	loader: require.resolve('eslint-loader'),  
    },  
  ],  
  include: paths.appSrc, //也可以用exclude排除不需要检查的目录或者用.eslintignore  
},  
```

二、ESlint配置**

<!-- more -->

**1.配置文件类型与优先级顺序**

- .eslintrc.js - 使用 .eslintrc.js 然后输出一个配置对象
- .eslintrc.yaml - 使用 .eslintrc.yaml 或 .eslintrc.yml 去定义配置的结构。
- .eslintrc.yml
- .eslintrc.json - 使用 .eslintrc.json 去定义配置的结构，ESLint 的 JSON 文件允许 JavaScript 风格的注释
- .eslintrc（已弃用）
- package.json - 在 package.json 里创建一个 eslintConfig属性，在那里定义你的配置

**2.plugin属性**

ESLint 支持使用第三方插件（以eslint-plugin-开头的npm包），在使用插件之前，必须使用 npm 安装。如eslint-plugin-react、eslint-plugin-vue等

```
module.exports = {  
  "plugins": [  
  	"react"  
  ],  
  "extends": [  
  	"eslint:recommended"  
  ],  
  "rules": {  
  	"no-set-state": "off"  
  }  
}  
```

**3.extends属性**

一个配置文件可以被基础配置中的已启用的规则继承。可以使用以下规则继承：

（1）"eslint:recommended"

继承Eslint中推荐的（打钩的）规则项

```
module.exports = {  
	"extends": "eslint:recommended",  
  "rules": {  

  }  
}  
```

（2）使用别人写好的规则包（以eslint-config-开头的npm包），如eslint-config-standard

```
module.exports = {  
  "extends": "standard",  
  "rules": {  

  }  
}  

```

（3）使用Eslint插件中命名的配置

```
module.exports = {  
  "plugins": [  
  	"react"  
  ],  
  "extends": [  
    "eslint:recommended",  
    "plugin:react/recommended"  
  ],  
  "rules": {  
  	"no-set-state": "off"  
  }  
}  
```

（4）使用"eslint:all"，继承Eslint中所有的核心规则项

```
module.exports = {  
  "extends": "eslint:all",  
  "rules": {  
    // override default options  
    "comma-dangle": ["error", "always"],  
    "indent": ["error", 2],  
    "no-cond-assign": ["error", "always"],  

    // disable now, but enable in the future  
    "one-var": "off", // ["error", "never"]  

    // disable  
    "init-declarations": "off",  
    "no-console": "off",  
    "no-inline-comments": "off",  
  }  
}  
```

**4.rules属性(根据自己的需要进行配置)**

（1）Eslint部分核心规则

```
"rules": {  
  /** 
  **这些规则与 JavaScript 代码中可能的错误或逻辑错误有关 
  **/  
  "for-direction":"error",//强制 “for” 循环中更新子句的计数器朝着正确的方向移动  
  "getter-return":"error",//强制在 getter 属性中出现一个 return 语句  
  "no-await-in-loop":"error",//禁止在循环中 出现 await  
  "no-compare-neg-zer":"error",//禁止与 -0 进行比较  
  "no-cond-assign":[//禁止在条件语句中出现赋值操作符  
  "error",  
  "always"  
  ],  
  "no-console":[//禁用 console  
  "error"  
  //      { "allow": ["warn", "error"] }  
  ],  
  "no-constant-condition":"error",//禁止在条件中使用常量表达式  
  "no-control-regex":"error",//禁止在正则表达式中使用控制字符  
  "no-debugger":"error",//禁用 debugger  
  "no-dupe-args":"error",//禁止在 function 定义中出现重复的参数  
  "no-dupe-keys":"error",//禁止在对象字面量中出现重复的键  
  "no-duplicate-case":"error",//禁止重复 case 标签  
  "no-empty":"error",//禁止空块语句  
  "no-empty-character-class":"error",//禁止在正则表达式中出现空字符集  
  "no-ex-assign":"error",//禁止对 catch 子句中的异常重新赋值  
  "no-extra-boolean-cast":"error",//禁止不必要的布尔类型转换  
  "no-extra-parens":"error",//禁止冗余的括号  
  "no-extra-semi":"error",//禁用不必要的分号  
  "no-func-assign":"error",//禁止对 function 声明重新赋值  
  "no-inner-declarations":"error",//禁止在嵌套的语句块中出现变量或 function 声明  
  "no-invalid-regexp":"error",//禁止在 RegExp 构造函数中出现无效的正则表达式  
  "no-irregular-whitespace":"error",//禁止不规则的空白  
  "no-obj-calls":"error",//禁止将全局对象当作函数进行调用  
  "no-prototype-builtins":"error",//禁止直接使用 Object.prototypes 的内置属性  
  "no-regex-spaces":"error",//禁止正则表达式字面量中出现多个空格  
  "no-sparse-arrays": "error",//禁用稀疏数组  
  "no-template-curly-in-string":"error",//禁止在常规字符串中出现模板字面量占位符语法  
  "no-unexpected-multiline":"error",//禁止使用令人困惑的多行表达式  
  "no-unreachable":"error",//禁止在 return、throw、continue 和 break 语句后出现不可达代码  
  "no-unsafe-finally":"error",//禁止在 finally 语句块中出现控制流语句  
  "no-unsafe-negation":"error",//禁止对关系运算符的左操作数使用否定操作符  
  "use-isnan":"error",//要求调用 isNaN()检查 NaN  
  "valid-jsdoc":"error",//强制使用有效的 JSDoc 注释  
  "valid-typeof":"error",//强制 typeof 表达式与有效的字符串进行比较  
  /** 
  **最佳实践 
  **/  
  "accessor-pairs":"error",//强制getter/setter成对出现在对象中  
  "array-callback-return":"error",//强制数组方法的回调函数中有 return 语句  
  "block-scoped-var":"error",//把 var 语句看作是在块级作用域范围之内  
  "class-methods-use-this":"error",//强制类方法使用 this  
  "complexity":"error"//限制圈复杂度  
  .....  
}  
```

（2）eslint-plugin-vue中的规则

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
  // 最佳实践  
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
  // 风格问题  
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
}  
```

（3）eslint-plugin-react中的规则

```
/** 
**react规则 
**/  
"react/boolean-prop-naming": ["error", { "rule": "^is[A-Z]([A-Za-z0-9]?)+" }],//bool类型的props强制固定命名  
"react/button-has-type": ["error", {"reset": false}],//强制按钮的type属性必须是"button","submit","reset"三者之一  
"react/default-props-match-prop-types": [2, { "allowRequiredDefaults": false }],//强制所有defaultProps有对应的non-required PropType  
"react/destructuring-assignment": [1, "always"],//强制将props,state,context解构赋值  
"react/display-name": [1, { "ignoreTranspilerName": false }],//react组件中强制定义displayName  
"react/forbid-component-props": [1],//禁止在自定义组件中使用(className, style)属性  
"react/forbid-dom-props": [1, { "forbid": ["style"] }],//禁止在dom元素上使用禁止的属性  
"react/forbid-elements": [1, { "forbid": ["button"] }],//禁止某些元素用于其他元素  
"react/forbid-prop-types": [1],//禁止某些propTypes属性类型  
"react/no-access-state-in-setstate":"error",//禁止在setState中使用this.state  
"react/no-children-prop":[1],//不要把Children当做属性  
"react/no-string-refs":[1],//不要使用string类型的ref  
"react/no-unused-state":[1],//不要在state中定义未使用的变量  
//.....  
"react/jsx-no-undef": [1, { "allowGlobals": false }],//不允许使用未声明的变量  
"react/jsx-key":[1]//遍历使用key  
```

原文引自: https://www.geek-share.com/detail/2747378137.html


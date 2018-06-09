---
title: css变量与calc()方法简单入门demo
date: 2017-05-09 10:31:48
tags: [CSS]
---

CSS变量应该称之为 CSS 自定义属性 ，不过下文为了好理解都称之为 CSS 变量。

一直以来我们都知道，CSS 中是没有变量而言的，要使用 CSS 变量，只能借助 SASS 或者 LESS 这类预编译器。

这个重要的 CSS 新功能，所有主要浏览器已经都支持了。本文主要介绍如何使用它，我们发现css如此强大，这是一个令人激动的革新，下面写个简单的demo来感受下CSS变量的强悍：

<p data-height="265" data-theme-id="dark" data-slug-hash="eWyeoZ" data-default-tab="html" data-user="it_hack" data-embed-version="2" data-pen-title="css变量 " class="codepen">See the Pen <a href="https://codepen.io/it_hack/pen/eWyeoZ/">css变量 </a> by kevin (<a href="http://codepen.io/it_hack">@it_hack</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

<!-- more -->

## 变量声明

声明变量的时候，变量名前面要加两根连词线（`--`）

```
:root {
  --color: red;
}
```

这里我们借助了结构性伪类中的 `:root{ }` 伪类，在全局 `:root{ }` 伪类中定义了一个 CSS 变量，取名为 `--color

**注：**变量名大小写敏感，`--header-color`和`--Header-Color`是两个不同变量。

## 变量使用

`var()`函数用于读取变量。

```
background-color:var(--backgroundColor)
```

`var()`函数还可以使用第二个参数，表示变量的默认值。如果该变量不存在，就会使用这个默认值。

```
background-color: var(--backgroundColor, #7F583F);
```

第二个参数不处理内部的逗号或空格，都视作参数的一部分。

**注**变量值只能用作属性值，不能用作属性名。

```
body{
  --side:margin-top;
  /* 无效 */
  var(--side): 20px;//变量--side用作属性名，这是无效的。
}
```

##### 变量值的类型

如果变量值是一个字符串，可以与其他字符串拼接。如例1中：

```
body:after{
  content: var(--con)" word";
  color:var(--color);
}
```

如果变量值是数值，不能与数值单位直接连用。数值与单位直接写在一起，这是无效的。必须使用`calc()`函数，将它们连接。

```
body {
  --top: 20;
  padding-top: calc(var(--top) * 1px);//calc会在css变量下边讲解
}
```

如果变量值带有单位，就不能写成字符串。

```
/* 无效 */
.body {
  --size: '20px';
  font-size: var(--size);
}

/* 有效 */
.body {
  --size: 20px;
  font-size: var(--size);
}
```

##### CSS 变量的层叠与作用域

CSS 变量是支持继承的，不过这里说成级联或者层叠应该更贴切。

同一个 CSS 变量，可以在多个选择器内声明。读取的时候，优先级最高的声明生效。这与 CSS 的"层叠"（cascade）规则是一致的。通俗一点就是局部变量会在作用范围内覆盖全局变量。

```
:root {
  --color: red;
  --backgroundColor: #F7EFD2;
  --con:'hello';
}
h2{
  --color: #00cc19;
  color:var(--color)
}
```

**注：**值得注意的是 CSS 变量并不支持 !important 声明。

**总结：**:

相较于传统的 LESS 、SASS 等预处理器变量，CSS 变量的优点在于:

1. CSS 变量的动态性，能在页面运行时更改，而传统预处理器变量编译后无法更改
2. CSS 变量能够继承，能够组合使用，具有作用域
3. 配合 Javascript 使用，可以方便的从 JS 中读/写

下面来看看目前为止都有哪些浏览器支持了CSS变量了：

![1](\images\css变量\1.png)

## calc()方法

之前在[大漠](http://www.w3cplus.com)和[阮一峰](http://www.ruanyifeng.com/)的博客中看到过calc的介绍，但一直没有使用和研究，今天接着这个css变量的机会我来一次demo，如有不对和错误的地方还请指出；

calc()从字面我们可以把他理解为一个函数function。其实calc是英文单词calculate(计算)的缩写，是css3的一个新增的功能，用来指定元素的长度。比如说，你可以使用calc()给元素的border、margin、pading、font-size和width等属性设置动态值。为何说是动态值呢?因为我们使用的表达式来得到的值。不过calc()最大的好处就是用在流体布局上，可以通过calc()计算得到元素的宽度。

**calc()能做什么？**
calc()能让你给元素的做计算，你可以给一个div元素，使用百分比、em、px和rem单位值计算出其宽度或者高度，比如说“width:calc(50% + 2em)”，这样一来你就不用考虑元素DIV的宽度值到底是多少，而把这个烦人的任务交由浏览器去计算。

**calc()语法**
calc()语法非常简单，就像我们小时候学加 （+）、减（-）、乘（*）、除（/）一样，使用数学表达式来表示：

```
width: calc(expression);
```

其中”expression”是一个表达式，用来计算长度的表达式。

**calc()的运算规则**
calc()使用通用的数学运算规则，但是也提供更智能的功能：

1. 使用“+”、“-”、“*” 和 “/”四则运算；
2. 可以使用百分比、px、em、rem等单位；
3. 可以混合使用各种单位进行计算；
4. 表达式中有“+”和“-”时，**其前后必须要有空格**，如”widht: calc(12%+5em)”这种没有空格的写法是错误的；
5. 表达式中有“*”和“/”时，其前后可以没有空格，但建议留有空格。



下边我们感受下calc的强大：

<p data-height="265" data-theme-id="dark" data-slug-hash="GmyyVg" data-default-tab="css" data-user="it_hack" data-embed-version="2" data-pen-title="calc()" class="codepen">See the Pen <a href="https://codepen.io/it_hack/pen/GmyyVg/">calc()</a> by kevin (<a href="http://codepen.io/it_hack">@it_hack</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

下面让我们看看都有那些浏览器支持calc()

![2](\images\css变量\2.png)

不知道大家对calc()整明白了没有，要是没有的话，继续动手写几个例子吧。如果您有更好的分享，记得告诉我哟。
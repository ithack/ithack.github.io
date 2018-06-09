---
title: 今天简单聊一下js的深拷贝（deepClone）
date: 2016-08-24 16:24:06
tags: [javascript,深拷贝]
---
javascript深拷贝是初学者甚至有经验的开发者，都会经常遇到问题，有时候面试也回问道这个问题；今天我就来聊一聊这个深拷贝；

# 深拷贝 #
有深拷贝自然就有浅拷贝，很人在接触这个概念的时候，是很懵逼的。

**我们为什么要用深拷贝？**

在很多情况下，我们都需要给变量赋值，给内存地址赋予一个值，但是在赋值引用值类型的时候，只是共享一个内存区域，导致赋值的时候，还跟之前的值保持一直性。
栗子：
<pre>
// 给test赋值了一个对象
var test = {
    a: 'a',
    b: 'b'
};
// 将test赋值给test2
// 此时test和test2是共享了同一块内存对象，这也就是浅拷贝
var test2 = test;
test2.a = 'a2';
test.a === 'a2'// 为true
</pre>
<!--more-->
这个栗子应该很好理解，因为我们都知道在js中有两种数据类型：基本类型、引用类型；那么我们的test对象就是一个引用类型，为什么叫引用类型呢？因为引用类型是以一种传址的方式给变量test的，也就是{a: 'a', b: 'b'}这个对象是存放在内存里的，而我们把这个内存地址给了test，当我们把test对象赋值给test2的时候，也就是test2也用了test的‘址’；所以test和test2指向了同一个内存地址栈中！其中一个改变就是在改变内存中的值！

要实现一个深拷贝函数，就不得不说javascript的数值类型。javascript中有以下基本类型：

| 类型 | 描述 |
|:-----:|:---:|:----------:|
| undefined | 描述 |
| null | 描述 |
| Boolean | Boolean有两种取值true和false |
| String | 它表示文本信息 |
| Number | 它表示数字信息 |
| Object | 它是一系列属性的无序集合， 包括函数Function和数组Array |

使用typeof是无法判断function和array的，这里使用Object.prototype.toString方法。
[默认情况下，每个对象都会从Object上继承到toString()方法，如果这个方法没有被这个对象自身或者更接近的上层原型上的同名方法覆盖(遮蔽)，则调用该对象的toString()方法时会返回"[object type]"，这里的字符串type表示了一个对象类型][1]

<pre>
function type(obj) {
    var toString = Object.prototype.toString;
    var map = {
        '[object Boolean]'  : 'boolean', 
        '[object Number]'   : 'number', 
        '[object String]'   : 'string', 
        '[object Function]' : 'function', 
        '[object Array]'    : 'array', 
        '[object Date]'     : 'date', 
        '[object RegExp]'   : 'regExp', 
        '[object Undefined]': 'undefined',
        '[object Null]'     : 'null', 
        '[object Object]'   : 'object'
    };
    return map[toString.call(obj)];
}
</pre>

## 实现deepClone ##
对于非引用值类型的数值，直接赋值，而对于引用值类型（object）还需要再次遍历，递归赋值。
<pre>
function deepClone(data) {
    var t = type(data), o, i, ni;
    
    if(t === 'array') {
        o = [];
    }else if( t === 'object') {
        o = {};
    }else {
        return data;
    }
    
    if(t === 'array') {
        for (i = 0, ni = data.length; i < ni; i++) {
            o.push(deepClone(data[i]));
        }
        return o;
    }else if( t === 'object') {
        for( i in data) {
            o[i] = deepClone(data[i]);
        }
        return o;
    }
}
</pre>

这里有个点大家要注意下，对于function类型，博主这里是直接赋值的，还是共享一个内存值。这是因为函数更多的是完成某些功能，有个输入值和返回值，而且对于上层业务而言更多的是完成业务功能，并不需要真正将函数深拷贝。

但是function类型要怎么拷贝呢？

其实博主只想到了用new来操作一下，但是function就会执行一遍，不敢想象会有什么执行结果哦！o(╯□╰)o！其它暂时还没有什么好的想法，欢迎大家指导哦！

到这里差不多也就实现完了深拷贝，又有人觉的怎么没有实现浅拷贝呢？

## 浅拷贝？ ##

对于浅拷贝而言，可以理解为只操作一个共同的内存区域！这里会存在危险！

如果直接操作这个共享的数据，不做控制的话，会经常出现数据异常，被其它部分更改。所以应该不要直接操作数据源，给数据源封装一些方法，来对数据来进行CURD操作。

到这里估计就差不多了，但是作为一个前端，不仅仅考虑javascript本身，还得考虑到dom、浏览器等。

## Element类型 ##

来看下面代码，结果会返回啥呢？

<pre>Object.prototype.toString.call(document.getElementsByTagName('div')[0])</pre>

答案是[object HTMLDivElement]

有时候保存了dom元素， 一不小心进行深拷贝，上面的深拷贝函数就缺少了对Element元素的判断。而判断Element元素要使用instanceof来判断。因为对于不同的标签，tostring会返回对应不同的标签的构造函数。

<pre>
function type(obj) {
    var toString = Object.prototype.toString;
    var map = {
        '[object Boolean]'  : 'boolean', 
        '[object Number]'   : 'number', 
        '[object String]'   : 'string', 
        '[object Function]' : 'function', 
        '[object Array]'    : 'array', 
        '[object Date]'     : 'date', 
        '[object RegExp]'   : 'regExp', 
        '[object Undefined]': 'undefined',
        '[object Null]'     : 'null', 
        '[object Object]'   : 'object'
    };
    if(obj instanceof Element) {
        return 'element';
    }
    return map[toString.call(obj)];
}
</pre>

这里大概总结了一下深拷贝，以及怎么实现一个深拷贝。在不同的场景下，要根据业务场景，判断是否需要使用深拷贝。其实实现深拷贝的方法有很多，这里只是用了其中一种方法！参考文章：[http://tostring.site/2016/08/21/javascriptDeepClone/](http://tostring.site/2016/08/21/javascriptDeepClone/)


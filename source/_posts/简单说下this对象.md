---
title: 简单说下this对象
date: 2016-08-10 18:37:10
tags: [Javascript]
---
快下班了，闲来无事就说说this对象把！
在JavaScript中，this关键字是动态绑定的，或称为运行期绑定，这极大地增强的我们程序的灵活性，同时也给初学者带来了很多困惑。本文总结了this的几个使用场景和常见误区。
#全局环境#
在全局环境中使用 this ，它会指向全局对象。
<pre>
	console.log(this);          // window对象
	alert(this === window);    // true
</pre>
#函数调用#
当作为普通函数被调用时，函数内部的的 this 也会指向全局对象。
<pre>
var obj= "window";
function Test(){
    var obj = "fun";
    alert(this.obj);
}
sayName();    // "window"
</pre>
#作为对象的方法调用#
当作为对象内部的方法被调用时，这里 this 指向这个方法所在的对象<!--more-->
<pre>
var name = "window";
var obj = {
  name: "obj",
  sayName: function(){
    alert(this.name);
  }
};
obj.sayName();    // "obj"
</pre>
#作为构造函数调用#
JavaScript 中的构造函数很特殊，如果不使用 new 调用，则和普通函数一样。作为又一项约定俗成的准则，构造函数以大写字母开头，提醒调用者使用正确的方式调用。如果调用正确， this 绑定到新创建的对象上。
<pre>
function Person(name){
    this.name = name,
    this.sayName = function(){
        alert(this.name);
    }
}

var oneObj = new Person("daoko");
oneObj.sayName();    // "darko"
</pre>
#apply和call调用#
apply 和 call 是函数对象的两个方法，它们可以修改函数执行的上下文环境，即 this 绑定的对象。 apply 和 call 的第一个参数就是this绑定的对象，若 apply 和 call 的参数为空，则默认调用全局对象。
<pre>
var name = "window"
var obj = {
  name: "object"
}
function sayName(){
  alert(this.name);
}
sayName();    // 直接调用函数sayName==>this指向window
sayName.call(obj);    // 用call方法修改this的指向为obj
sayName.call();    // 当call方法的参数为空时，默认调用全局对象==>this指向window
</pre>
#特殊情况#
栗1：
<pre>
var name = "window";
var obj = {
  name: "obj",
  sayName: function(){
    var test = function(){
      alert(this.name);    // this绑定到全局对象上
    }
    test();
  }
}
obj.sayName();    // "window"
</pre>
按照我们的理解此时的 this 应该指向 obj 对象，但是实际情况不是这样的，此时的 this 指向全局对象。

这属于 JavaScript 的设计缺陷，正确的设计方式是内部函数的 this 应该绑定到其外层函数对应的对象上，为了规避这一缺陷，我们可以使用变量替代的方法，约定俗成，该变量一般被命名为 that。 在这个栗子中，这样我们创建了一个局部变量 _this 来指向 obj 对象。
<pre>
var name = "window";
var obj = {
  name: "obj",
  sayName: function(){
    var _this = this;    // that指向对象obj
    var test = function(){
      alert(_this.name);
    }
    test();
  }
}
obj.sayName();    // "obj"
</pre>
栗2：方法的赋值表达式
<pre>
var name = "window";
var obj = {
  name: "obj",
  sayName: function(){
    alert(this.name);
  }
}
var test = obj.sayName;
obj.sayName();    // "obj"
test();    // "window"
</pre>
从上面这个栗子中我们可以看到，当把对象 obj 的方法赋值给一个新的变量 test 时，它的this指向发生了变化， test 就向一个普通的函数一样被调用，此时指向全局对象。
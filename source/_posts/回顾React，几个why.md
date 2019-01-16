---
title: 回顾React，几个why
date: 2018-10-23 17:14:01
tags: [react]
---

​      虽然react比Vue早一些，但是我还是先用了Vue；原因很简单当时看了一下React Native和React的API，觉得项目不需要这么复杂的东西，当时还是vue1.x时代，写了大概3个项目后，Vue2.x上线，之后的项目一直在用vue，从1.x到2.x一直踩坑，到目前可以说对Vue很熟练了也做了很多项目！其中也写过一个React，只是看了API，然后上手就写，感觉一直在皮毛的表层应用；

​      最近项目不紧，闲来无事，就在想之前写的react项目，我不能只会用，只关注皮毛，还要多问一些为什么，所以有了今天这篇文章，通过‘十万个为什么’来让自己更加深入的学习了解react，以便自己更加了解react；

​    好了，牢骚不多说，下面进入正题；

##### 第一问：为什么用JSX？

不需要为了React使用JSX，可以直接使用JavaScript，但建议使用 JSX 是因为它能精确，也是常用的定义包含属性的树状结构的语法

1、没有 JSX 的 React：

①　可以使用React.createElement()方法

②　可以使用React.createFactory()方法（工厂方法）

③　可以使用React.DOM.*（React为HTML标签提供的内置工厂方法）

JSX 是一个看起来很像XML的JavaScript语法扩展。

2、JSX的特点：

①　JSX 不是必须用的，可以直接使用JavaScript；

②　JSX 非常小；

③　JSX 类似于HTML，但不完全一样，如HTML对大小写不敏感，JSX对大小写敏感；HTML中不强制要求标签闭合，但JSX要求标签一定要闭合；JSX的组件要以大写字母开头，以便与HTML标签区分。

3、HTML标签 VS React组件

在JSX语法中，遇到HTML标签（以<开头）就用HTML规则解析，遇到代码块（以{开头）就用JavaScript规则解析。

React可以渲染HTML标签或React组件类：要渲染HTML标签，只需在JSX里使用小写字母的标签名；要渲染React组件，只需创建一个大写字母开头的本地变量。即，JSX 使用大、小写来区分React组件类和HTML标签。

注意：由于JSX就是JavaScript，class 和 for 等标识符不建议作为属性名，因此React DOM 使用 className 和 htmlFor作为相应的属性名。



##### 第二问：为什么ReactJS组件名称必须以大写字母开头？

 在JSX中，小写标签名称被认为是HTML标签。但是，带点（属性访问器）的小写标签名不是。

查看[HTML标记与React组件](https://facebook.github.io/react/docs/jsx-in-depth.html#html-tags-vs.-react-components)。

- `<component />`编译为`React.createElement('component')`（html标签）
- `<Component />` 编译 `React.createElement(Component)`
- `<obj.component />` 编译 `React.createElement(obj.component)`

****待续……****
---
title: 关于translate会使内容变得模糊的问题--实战小记
date: 2016-08-22 17:31:44
tags: [CSS3,transform,translate,随笔]
---
刚刚，在调试一个vue的单页面应用时发现一个字体模糊问题！现在来说说问题！

具体页面是这样的，有一个弹框，这个弹框需要水平垂直居中！其实一般情况我们首先想到的是：transform的translate方法!具体方法如：

<pre>
.main{
  position: fixed;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);//或transform: translate3d(-50%, -50%,0)
}
</pre>
这样main这个div就会居中显示了！但是如我一开始说的，这样做这个div里的文字会变得模糊，具体原因我还没了解(需要等这个项目忙完去研究下，很快就这一两天的事！)，所以我们需要改变一下我们的方式这里先说下其中一个也是用CSS3实现的方法，就是比较麻烦，要在main这个div外层再加个div，这个div的class为prant，css如下：
<pre>
.prant{
  height:100%;
  width:100%;position:absolute;z-index:10002;top:0;left:0;
  display: -webkit-flex;
  display: flex;
  -webkit-align-items: center;
  align-items: center;
  -webkit-justify-content: center;
  justify-content: center; 
}
</pre><!--more-->
如上方法是用了CSS3的弹性布局实现的水平垂直居中！

下面我在说一个另一个比较另类的实现方法，这个方法是在上周才从同事哪里得来的，当时的场景并不是水平垂直居中，而是另一个场景，这里先卖个棺材，上个代码给大家看看，我相信很多人回意外，代码如下：
<pre>
.main{
  position:absolute;
  left:0;
  top:0;
  bottom:0;
  right:0;
  margin:auto;
  width:50%;
  height:40px;
}
</pre>

不知道有多少看管觉得这是一段水平垂直居中的代码呢？其实我看到的时候我个人认为这个是布局中的可是当看到效果后我傻了，这个代码确实让main居中了，还是水平垂直居中！难道是我out了吗？

其实我同事告诉我这段代码时的场景不是水平垂直居中，而是另一个场景：又一个pc的滚屏效果页面，每一个page里都是一个水平垂直剧中的背景！问题在于有一个背景的某个位置上有一个连接按钮，这时我要保证不管什么样的屏幕下，这个按钮都在这个背景图的那一个位置上;那么问题来了我的这个背景是根据屏幕宽高自适应居中的，所以这个按钮的位置也不可能写成px或者%单位！我又不想去用js计算这个按钮和背景图的比例问题；然后我就学到了这么一个黑科技，如上代码中，是让某个div水平垂直居中，那么现在各位看官可以改变一下，left或top的值，然后缩小你的浏览器看看效果！不管你有没有惊到，反正当我看到效果后我惊到了！

最有一个居中方法的黑科技我认为很实用，所以分享出来，如果有人知道WHY求告知！
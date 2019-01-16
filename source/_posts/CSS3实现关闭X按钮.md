---
title: CSS3实现关闭X按钮
date: 2016-08-12 10:11:01
tags: [随笔,CSS]
---
昨天做一个小的页面时用到了弹框，一般弹框都有close按钮，有时我们为了偷懒直接用个X来做关闭按钮！但这种情况基本很难看，所以我们需要UI给我设计一个关闭按钮，或者我们要用font-face来做这个关闭，这样做起来美观，但是却有一点麻烦！之前我无意间发现一个好玩的方法，简单好用,今天分享一下；

用CSS3设计出来的样式来呈现出关闭按钮的样子，这样以后想设计简单的对话框，就不需要求美工来帮忙画按钮的图片了。先来看看样式:
<pre>
/* basic style */
.close {
    background: orange;
    color: red;
    border-radius: 12px;
    line-height: 20px;
    text-align: center;
    height: 20px;
    width: 20px;
    font-size: 18px;
    padding: 1px;
}
/* use cross as close button */
.close::before {
    content: "\2716";
}
/* place the button on top-right */
.close {
    top: -10px;
    right: -10px;
    position: absolute;
}
</pre>
然后直接把它加到HTML元素上就可以看到效果了：
<pre>
&lt;div style="height: 100px; width: 100px; border: 1px solid black; position: relative;"&gt;<span class="close"></span>&lt;/div&gt;
</pre>
这主要是利用了在CSS的::before伪属性可以直接添加内容的特点，这样就可以直接设置样式在class上而不用做其它操作。当然这里用::after效果也是一样的。唯一要注意的是如果要指定关闭按钮在特定位置，就需要使用到position:absolut;，相应的其父元素就不能是默认的position:static;。像上边的示例中就使用的是position:relative;以保证按钮呈现在正确的位置上。

对于按钮中的叉按钮，用的是Unicode中的一种标识，以下这些也可以使用：

|                    标识                    | 编码(16进制) | Name |
| :--------------------------------------: | :------: | :--: |
| ![](\images\CSS3实现关闭X按钮\2716.png) |  \2716   | 粗体乘法 |
|                    ✗                     |  \2717   |  交叉  |
|                    ✘                     |  \2718   | 粗体交叉 |
|                    ×                     |  \00D7   | 乘法符号 |
|                    ⨯                     |  \2A2F   | 向量乘积 |
|                    x                     |   \78    | 小写字母 |
|                    X                     |   \58    | 大写字母 |

只是使用了不同字符后，可能居中位置会有偏差需要重新调整。

再结合JavaScript和CSS3的gradient，就能完成一个看起来还不错的对话框了。效果如下图：


![](\images\CSS3实现关闭X按钮\close.png)

其实我们可以完全用字符编码解决关闭X按钮的美观问题，上面是一种不用美工不用font-face的解决方案，在这个基础上我们也可以发散思维，用另一种方法解决，这里这是做一个简单的介绍，我们可以利用css3的旋转属性，父级有个边框，子级有个边框，把子级旋转九十度，和父级交叉，把父级旋转45°这样也是一个关闭X按钮！

如果大家还有什么好的建议和想法，欢迎留言分享！

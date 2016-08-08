---
title: 说一说github上免费搭建自己的博客
tags: [随笔]
categories: [博客]
date: 2016-07-21 15:43:14
---


> 昨天是帝都发大水的日子，我为了上班徒步了半个多小时到公司，为的是不矿工，等到了下午公司人性化的发邮件，当天考勤异常不扣工资，4点下班，我对这么人性的公司说一句太好了，所以今天更新我在github上第一次文章《github搭建博客》

 好了不废话了，下面开始我的搭建工作！

  首先我们先要在本地搭环境：
 
**1.安装Node（必须）**

**2.安装Git（必须）**

**3.申请GitHub（必须）**
  
注：如上三步我就不做详细介绍怎么安装了，这都是程序猿的基本搭配，实在不会可以google，so一把！下面开始我们的搭建安装博客！（win下搭建）

1）正式安装HEXO　

   可以打开CMD或gitBash命令行执行如下`npm install -g hexo` 安装hexo

2）初始化你的本地hexo项目
<!--more-->
   打开你本地自己项目目录，我自己的项目目录：D:/mybolg,用命令行打开，然后在当前项目里初始化hexo

   `hexo init`

   好啦，至此，全部安装工作已经完成！

3）生成静态页面

  现在你的项目中有了hexo的基本架构和目录了我们需要生成静态页到项目

  `hexo generate 或 hexo g `

  现在我们可以本地启动访问操作了
   
  `hexo server`

  好了启动了，我们可以在浏览器中访问地址[http://localhost:4000 ](ithack.github.io ) 了，这里我就成功了，但是有的人可能遇到了报错：

  `ERROR Plugin load failed: hexo-server`

  解决方法，执行命令：

  `npm install hexo-server`

**现在我们本地hexo已经装好了，可以说基本完成了一般了，下面重点来了，我们要在github上搭建博客了！！**

1）建立Github Pages
   
  这一步说简单也简单，说复杂也复杂，下方新手牢记我下面说的每一步！
  
![](http://i1.piimg.com/567571/ea675473d76ce4ee.png)

  一定要注意这一步，看一看你github的用户名，看一看你github的用户名，看一看你github的用户名  重要的是事情说三遍！下边我们开始在github上创建新仓库，创建一个仓库，但是这个仓库是有规则的，其格式必须为：yourusername.github.io而仓库命名格式中的yourusername是你的github用户名。笔者的github用户名是ithack。所以仓库命名格式则是ithack.github.io

  由于笔者有ithack.github.io这个仓库，所以我下述效果图提示已存在。纯属作为演示。

![](http://i1.piimg.com/567571/a27a6a678762b268.png)

 创建仓库成功后继续如下图操作：

![](http://i2.piimg.com/567571/e3942ca0d5c73c1d.png)

![](http://i4.piimg.com/567571/6398d94e955681fa.png)

由于笔者已经有创建成功，所以会有第一个红框内的提示的[http://ithack.github.io](http://ithack.github.io) 的连接,你第一次进来是没有的，只需要点击下边的` Launch automatic page generator` 走下一步即可；

![](http://i2.piimg.com/567571/6554b447ffdb34ef.png)

这一步就不多说了，直接下一步

![](http://i2.piimg.com/567571/dc7f9fd81e1c1c52.png)

这里是选择模版，选择好后点击完成，你的github博客站点就建立好了，现在你可以访问yourName.github.io这个地址了，你会看到你的博客已经初步形成了，这里我们只是用了Github Pages的默认模版，如果你觉得这样就可以了，那么现在你就可以开始你的博客之旅了！

  而我的博客到现在还没有弄完，我们要用的是hexo的博客框架！所以下面我们需要配置hexo并与github绑定，在本地写博文后push to github

# 配置Github和hexo关联 #

  前面我们已经在本地建立了hexo项目，现在我们要把本地和github关联想，我们建立好的东西有：

	_config.yml  node_modules    public      source
    db.json   package.jsonscaffolds   themes
  
  下面我们需要用_config.yml文件，来建立关联，打开_config.yml，翻到最下面，改成我这样子的，**注意： : 后面要有空格**

	deploy:
	type: git
	repository: git@github.com:ithack/ithack.github.io.git
	branch: master
  执行如下命令才能使用git部署
    
	npm install hexo-deployer-git --save

  网上会有很多说法，有的type是github, 还有repository 最后面的后缀也不一样，是github.com.git，我也踩了很多坑，我现在的版本是hexo: 3.1.1，执行命令hexo -vsersion就出来了,貌似3.0后全部改成我上面这种格式了。 忘了说了，我没用SSH Keys如果你用了SSH Keys的话直接在github里复制SSH的就行了，总共就两种协议，相信你懂的。 然后，执行配置命令：

	hexo deploy

 然后再浏览器中输入http://ithack.github.io/就行了，我的github的账户叫ithack,把这个改成你github的账户名就行了

 每次部署的步骤，可按以下三步来进行。

	hexo clean
	hexo generate
	hexo deploy

 一些常用命令：

	hexo new "postName" #新建文章
	hexo new page "pageName" #新建页面
	hexo generate #生成静态页面至public目录
	hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
	hexo deploy #将.deploy目录部署到GitHub
	hexo help  # 查看帮助
	hexo version  #查看Hexo的版本

# 一些基本路径 #
   文章在source/_posts，如果你不怕麻烦的话可以跟我一样直接用vim去编辑，支持markdown语法，你有好的编辑软件，给我也推荐下，感激不尽?。如果想修改头像可以直接在主题的_config.yml文件里面修改，友情链接，之类的都在这里，修改名字在public/index.html里修改，开始打理你的博客吧，有什么问题或者建议，都可以提出来，我会继续完善的。
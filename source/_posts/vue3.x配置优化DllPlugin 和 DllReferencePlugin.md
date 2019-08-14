---
title: vue3配置优化DllPlugin 和 DllReferencePlugin
date: 2019-08-14 10:03:40
tags: [vue, vue3.x, webpack, DllPlugin]
---

#### 前言

目前在做的项目是我从0到1一手搭建起来的，所以比较在整体结构和对架构的优化上思考并实践了很多，每一次的开发需求都先用一部分时间思考项目的打包和搭建怎么样才能更加方便通用化，今天就记录一下在越来愈多的需求和组件写完后怎么才能提高webpack的build速度，可能百度会有很多方案，今天我就来针对当前vue-cli3.x搭建的项目dll文件抽离配置

#### **什么是DllPlugin 和 DllReferencePlugin**

在使用webpack进行打包时候，对于依赖的第三方库，比如vue，vuex等这些不会修改的依赖，我们可以让它和我们自己编写的代码分开打包，这样做的好处是每次更改我本地代码的文件的时候，webpack只需要打包我项目本身的文件代码，而不会再去编译第三方库，那么第三方库在第一次打包的时候只打包一次，以后只要我们不升级第三方包的时候，那么webpack就不会对这些库去打包，这样的可以快速的提高打包的速度。因此为了解决这个问题，DllPlugin 和 DllReferencePlugin插件就产生了。

<!-- more -->

那么对于目前webpack社区来讲，我们希望和自己编写的代码分离开的话，webpack社区提供了2种方案：
**1. CommonsChunkPlugin**
**2. DLLPlugin**

**CommonsChunkPlugin** 插件每次打包的时候还是会去处理一些第三方依赖库，只是它能把第三方库文件和我们的代码分开掉，生成一个独立的js文件。但是它还是不能提高打包的速度。在vue3.x的vue.config配置文件中我们可以从官网找到optimization的配置，此配置很实用，具体的操作这里就不介绍，可官网查看[optimization](https://webpack.js.org/configuration/optimization/#root)配置

**DLLPlugin** 它能把第三方库代码分离开，并且每次文件更改的时候，它只会打包该项目自身的代码。所以打包速度会更快。

**DLLPlugin** 这个插件是在一个额外独立的webpack设置中创建一个只有dll的bundle，也就是说我们在项目根目录下除了有webpack.config.js，还会新建一个webpack.dll.config.js文件。webpack.dll.config.js作用是把所有的第三方库依赖打包到一个bundle的dll文件里面，还会生成一个名为 manifest.json文件。
该manifest.json的作用是用来让 DllReferencePlugin 映射到相关的依赖上去的。

**DllReferencePlugin** 这个插件是在webpack.config.js中使用的，该插件的作用是把刚刚在webpack.dll.config.js中打包生成的dll文件引用到需要的预编译的依赖上来。什么意思呢？就是说在webpack.dll.config.js中打包后比如会生成 vendor.dll.js文件和vendor-manifest.json文件，vendor.dll.js文件包含所有的第三方库文件，vendor-manifest.json文件会包含所有库代码的一个索引，当在使用webpack.config.js文件打包DllReferencePlugin插件的时候，会使用该DllReferencePlugin插件读取vendor-manifest.json文件，看看是否有该第三方库。vendor-manifest.json文件就是有一个第三方库的一个映射而已。

所以说 第一次使用 webpack.dll.config.js 文件会对第三方库打包，打包完成后就不会再打包它了，然后每次运行 webpack.config.js文件的时候，都会打包项目中本身的文件代码，当需要使用第三方依赖的时候，会使用 DllReferencePlugin插件去读取第三方依赖库。所以说它的打包速度会得到一个很大的提升

#### **在项目中如何使用 DllPlugin 和 DllReferencePlugin**

由于是vue3.x的脚手架，所以这里说一下vue.config中的配置方法；其实就算用vue2.x的脚手架也只是目录不同，方法相同，毕竟这些工作是webpack做的

首先在根目录创建一个webpack.dll.conf的js文件代码如下：

```
const path = require('path')
const webpack = require('webpack')
const {CleanWebpackPlugin} = require('clean-webpack-plugin')
// dll文件存放的目录(建议存放到public中)
const dllPath = './public/vendor'
module.exports = {
  mode: 'production',
  entry: {
    vendor: ['vue/dist/vue.runtime.esm.js', 'vuex', 'vue-router', 'element-ui', 'axios', 'moment']
  },
  output: {
    filename: '[name].dll.js',
    path: path.resolve(__dirname, dllPath),
    library: '[name]_[hash]'
  },
  plugins: [
    new CleanWebpackPlugin(),
    new webpack.DllPlugin({
      name: '[name]_[hash]',
      path: path.join(__dirname, dllPath, '[name].manifest.json'),
      context: process.cwd()
    })
  ]

```

**注：不知道是不是项目包的webpack是3.x的缘故，需要在项目中安装webpack-cli;`npm i webpack-cli -D`******

如上代码我们还用到了clean-webpack-plugin插件，每次打包清除上次的文件夹

**DllPlugin 插件有三个配置项参数如下：**
**context(可选)：** manifest文件中请求的上下文，默认为该webpack文件上下文。
**name:** 公开的dll函数的名称，和 output.library保持一致。
**path:** manifest.json 生成文件的位置和文件名称。

然后需要在package中scripts中配置命令`"dll": "webpack -p --progress --config ./webpack.dll.config.js"`命令这样我们就可以在需要的时候把包单独抽离出来了

下面重点来了，vue.config.js配置如下：

```
const AddAssetHtmlPlugin = require('add-asset-html-webpack-plugin')
function resolve(dir) {
  return path.join(__dirname, dir)
}
const dllReference = (config) => {
  config.plugin('vendorDll')
    .use(webpack.DllReferencePlugin, [{
      context: __dirname,
      manifest: require('./public/vendor/vendor.manifest.json')
    }])
  config.plugin('addAssetHtml')
    .use(AddAssetHtmlPlugin, [
      [
        {
          filepath: resolve(__dirname, 'public/vendor/vendor.dll.js'),// dll文件位置
          outputPath: 'vendor',// dll最终输出的目录
          publicPath: '/vendor'// dll 引用路径
        }
      ]
    ])
    .after('html')
}
module.exports = {
	......
	chainWebpack: config => {
    dllReference(config)
  },
}
```

这里说我们用了add-asset-html-webpack-plugin插件，是为了省去手动修改template/html这一步骤

**DllReferencePlugin项的参数有如下：**

**context:** manifest文件中请求的上下文。
**manifest:** 编译时的一个用于加载的JSON的manifest的绝对路径。
**context:** 请求到模块id的映射(默认值为 manifest.content)
**name:** dll暴露的地方的名称(默认值为manifest.name)
**scope:** dll中内容的前缀。
**sourceType:** dll是如何暴露的libraryTarget。

#### 效果

使用DllPlugin打包前

![image-20190814120105164](/images/2019/dllPlugin1.png)

使用DllPlugin打包后

![image-20190814120212299](/images/2019/dllPlugin2.png)

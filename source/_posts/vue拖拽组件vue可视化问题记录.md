---
title: vue拖拽组件vue可视化问题记录问题记录
date: 2018-07-12 14:37:01
tags: [vue,vuedraggable,拖拽]
---
已经停更一年了，平时不是没时间更新，就是不知道记录什么！平时遇到的问题和解决思路都简单的弄了个list作为文章更新点，今天忙里抽闲，在单位的电脑装上blog环境，开始更新一个从零到一纯手动搭建的关于[vue可视化系统，组件拼接页面](https://github.com/ithack/vueCMS)

，从开始的毫无头绪，到目前的一步步踩坑经历了从绝望看到希望，感觉自己变牛逼了（其实并没有）！好了废话不多说，内容比较多和杂，一步一步来；<!--more-->

#### 关于webpack配置

##### **HtmlWebpackPlugin插入顺序**

项目场景设定就要有一个默认模版，所以这里需要打包后生成的js和css等文件插入一个html模版里，这里可能就需要有一个js加载顺序的问题了，所以就有了**webpack配置 HtmlWebpackPlugin插件生成html插入js 的时候 按chunks 顺序**插入解决方案，配置如下：

```
new HtmlWebpackPlugin({
	.......
	chunksSortMode: function (chunk1, chunk2) {//插入顺序
		var order = ['vendor', 'build'];
		var order1 = order.indexOf(chunk1.names[0])
		var order2 = order.indexOf(chunk2.names[0])
		return order1 - order2
	},
})
```

另附上关于htmlWebpackPlugin插件的配置项:

- title: 用来生成页面的 title 元素
- filename: 输出的 HTML 文件名，默认是 index.html, 也可以直接配置带有子目录。
- template: 模板文件路径，支持加载器，比如 html!./index.html
- inject: true | 'head' | 'body' | false  ,注入所有的资源到特定的 template 或者 templateContent 中，如果设置为 true 或者 body，所有的 javascript 资源将被放置到 body 元素的底部，'head' 将放置到 head 元素中。
- favicon: 添加特定的 favicon 路径到输出的 HTML 文件中。
- minify: {} | false , 传递 html-minifier 选项给 minify 输出
- hash: true | false, 如果为 true, 将添加一个唯一的 webpack 编译 hash 到所有包含的脚本和 CSS 文件，对于解除 cache 很有用。
- cache: true | false，如果为 true, 这是默认值，仅仅在文件修改之后才会发布文件。
- showErrors: true | false, 如果为 true, 这是默认值，错误信息会写入到 HTML 页面中
- chunks: 允许只添加某些块 (比如，仅仅 unit test 块)
- chunksSortMode: 允许控制块在添加到页面之前的排序方式，支持的值：'none' | 'default' | {function}-default:'auto'
- excludeChunks: 允许跳过某些块，(比如，跳过单元测试的块) 

更具体的相关配置google查询这里附上一个比较详细的[API](https://www.cnblogs.com/wonyun/p/6030090.html)

##### 关于热启动

我们习惯了`npm run dev` 实际上我们在启动wepack-dev-server的时候，是生成了一个类似temp的缓存文件类似的东西，我们所有打包热更新后的文件都在这个不可见的文件里（我是这么理解的，欢迎指正），我一开始配置webpack的时候并没有配置output.filename这个参数，我认为用不到，导致我启动开发时dist每次热更新都会有一个新的hash的js文件出现，导致最后出口文件里好多过时的js文件，所以在手配webpack务必陪一下filename

```
output: {
    path: path.resolve( './dist/'),
    filename: '[name].js'
  },
```

##### webpack 图片打包

对于webpack图片的loader的配置我想很多人都会配，那么这里说的配置肯定不是普通需求的配置；我们前端开发可能遇到一个场景就是打包后的文件放到一个cdn或一个服务器上，但是引用我们静态资源的地方并不一定是在这两个地方，那么有一个比较尴尬的地方就是，图片的打包后在其他地方用就会出现404的情况，这时候我们需要把所有图片的地址全部换成可访问的域名地址了，所以就有了如下配置：

```
{
      test: /\.(png|jpg|jpeg|gif)$/,
      loader: 'url-loader?limit=8192&name=[name].[ext]&outputPath=img/&publicPath=../'
}
```

**参数说明:**
limit: 在文件小于多少个字节时，将文件编码并返回DataURL。
name: 表示输出的文件名规则，如果不添加这个参数，输出的就是默认值：文件哈希。
outputPath: 表示输出文件路径前缀(此例中意为:在输出文件夹下新建（如果没有）一个名为img的文件夹，把图片放到里面)。
publicPath: 表示打包文件中图片路径的前缀，如果你的图片存放在CDN上，那么你上线时可以加上这个参数，值为CDN地址，这样就可以让项目上线后的资源引用路径指向CDN了。

其实不止url-loader有这些配置，其他的一些loader也可以配置，具体配置可查看loader配置用法

##### webpack+vue组件懒加载配置

这里主要是在一起完成后，我在完善用户体验和架构优化上做的一些工作，第一步就是考虑以后开发组件多了，但是并不是所有页面都需要加载这些组件，所以先做了一个vue组件懒加载的优化，只有当用到这个组件时才会加载这个组件（目前遇到一个问题，还没有找到根源；就是异步组件后拖拽到第二个拖出和第一个相同的组件后会有bug，后期查到原因再做记录，这里只做异步加载组件的介绍）

1. 在webpack配置文件中的output路径配置chunkFilename属性

   ```
   output: {
       path: resolve(__dirname, 'dist'),
       filename: options.dev ? '[name].js' : '[name].js?[chunkhash]',
       chunkFilename: 'chunk[id].js?[chunkhash]',
       publicPath: options.dev ? '/assets/' : publicPath
   },
   ```

   chunkFilename路径将会作为组件懒加载的路径

2. vue组件需要做的事情

   ```
   /全局组件
   Vue.component('AsyncComponent', () => import('./AsyncComponent'))
   
   //局部注册组件
   new Vue({
    // ...
    components: {
    'AsyncComponent': () => import('./AsyncComponent')
    }
   })
   
   // 如果不是default导出的模块
   new Vue({
    // ...
    components: {
    'AsyncComponent': () => import('./AsyncComponent').then({ AsyncComponent }) => AsyncComponent
    }
   })
   ```

   

如上是关于webpack的配置



#### 关于VUE相关问题记录



##### 在vue组件内使用less

一开始用less时按照less的格式写但是并未编译，后来查看资料才知道，必须必须加上lang=‘less’在webpack才会能正常解析（sass同理）；不加的话不能用，完全不会解析，嵌套写法等都会失效

##### 关于axios跨域问题

在使用axios的post跨域时遇到一个问题，axios的post请求不成功即使配置了withCredentials，后端配置了cors也不生效问题；后来查了好多资料才发现，并不是axios的问题而是mockjs的问题；如下是相关issues：

[axios请求不携带cookie](https://github.com/PanJiaChen/vue-element-admin/issues/562)

[withCredentials 冲突](https://github.com/nuysoft/Mock/issues/95)

关于axios的post请求入参问题： 后端的post请求默认接收形式是formData形式，而axios的请求默认是json形式，所以axios的post请求要修改入参格式，官网给出的方式如下： [axios官方解决方案](https://github.com/axios/axios/issues/827) [相关资料](https://www.jianshu.com/p/b22d03dfe006)

简单用了一下qs这个插件，貌似没成功，之后就直接换了个自己的方式如下：

```
axios.defaults.transformRequest = [function (data) {
  let newData = ''
  for (let k in data) {
    newData += encodeURIComponent(k) + '=' + encodeURIComponent(data[k]) + '&'
  }
  return newData
}]
```

针对axios进行了一个二次封装如下：

```
import axios from 'axios'
//http.js
//设置请求baseURL
// axios.defaults.baseURL = '/app'
//设置默认请求头
axios.defaults.headers = {
  "Content-Type": "application/x-www-form-urlencoded; charset=UTF-8",
  'Accept': 'application/json',
}
axios.defaults.timeout = 10000
axios.defaults.withCredentials=true
//添加请求拦截器
axios.interceptors.request.use(config => {
  //在发送请求之前做某事，比如说 设置loading动画显示
  $('#loading').show()
  return config
}, error => {
  //请求错误时做些事
  return Promise.reject(error)
})

//添加响应拦截器
axios.interceptors.response.use(response => {
  //对响应数据做些事，比如说把loading动画关掉
  $('#loading').hide()
  return response.data
}, error => {
  //请求错误时做些事
  return Promise.reject(error)
})

//如果不想要这个拦截器也简单，可以删除拦截器
//axios.interceptors.request.eject(myInterceptor)
// POST发送请求前处理入参的数据为：formData 形式
axios.defaults.transformRequest = [function (data) {
  let newData = ''
  for (let k in data) {
    newData += encodeURIComponent(k) + '=' + encodeURIComponent(data[k]) + '&'
  }
  return newData
}]
axios.jsonp=(url,params)=>{
  // 判断url是否存在以及是否为字符串
  if(!url || typeof url !== 'string') throw new Error('必须传入字符串类型的url地址');
  let param=function(data){
    let url = ''
    for (var k in data) {
      let value = data[k] !== undefined ? data[k] : ''
      url += '&' + k + '=' + encodeURIComponent(value)
    }
    return url ? url.substring(1) : ''
  };
  url += (url.indexOf('?') < 0 ? '?' : '&') + param(params)
  return new Promise((resolve,reject) => {
    // 处理返回的结果
    var oDate=new Date().getTime();
    window['jsonCallBack'+oDate]= (result) => {
      resolve(result)
    }
    // 在页面创建script标签，并将src设置为请求地址，取回数据之后移除script标签
    let JSONP = document.createElement("script");
    JSONP.type = "text/javascript";
    JSONP.src = `${url}&callback=jsonCallBack${oDate}`;
    document.getElementsByTagName("head")[0].appendChild(JSONP);
    setTimeout(() => {
      document.getElementsByTagName("head")[0].removeChild(JSONP)
    },500)
  })
}
axios.oGet=(url,params)=>{
  return axios.get(url,{"params":params})
  if(process.env.NODE_ENV=="development"){
    return axios.jsonp(url+'?format=jsonp',params)
  }else{
    return axios.get(url,{"params":params})
  }
}
//导出使用
export default axios 
```

##### 关于vue中click触发新标签打开方法

```
let newLink=window.open()
newLink.location.href=URL
```



关于vue的可视化生成页面系统一些问题暂时记录到这里，有对vue可视化感兴趣可以看看[可视化](https://github.com/ithack/vueCMS)，其中的说说明还有结构是未优化代码，后期项目稳定后，会更新优化好的代码！有任何问题欢迎大家建议和指正；
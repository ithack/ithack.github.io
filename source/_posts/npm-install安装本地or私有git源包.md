---
title: npm install安装本地/私有git源包
date: 2019-06-28 14:03:40
tags: [npm, package]
---

#### 前言

​	之所以是总结关于npm install的问题，是因为之前在组织组内的一次分享会上我问了小伙伴们一个问题；当时有一个M移动端项目，用到了公司的私有源上的分享插件，而每个人在执行npm install的时候默认淘宝或npm源中下载，所以导致我们自己的私有源要单独用nrm切换到私有源再次npm install一次；当然我们的私有源也很强大基本上的npm源上的包都有，我们可以直接切换到公司镜像源然后npm install一下，这样私有源和npm源的包都能安装，但在极端情况万一npm源某些个别包，公司的源上没有同步上来呢？这种情况我们是不是来回切换？还有之前和后端JAVA合作，JAVA有一个种dubbo接口，这种接口就是输出git包的形式，这种情况我们不能吧dubbo接口的包文件放到公司源里去；

​	其实在package里是支持输入本地包或git 源包的，so 接下来我来梳理总结一下npm install 依赖包安装！

#### package定义

我们都知道要手动安装一个包时，执行 `npm install <package>` 命令即可。这里的第三个参数 package 通常就是我们所要安装的包名，默认配置下 npm 会从默认的源 (Registry) 中查找该包名对应的包地址，并下载安装。但在 npm 的世界里，除了简单的指定包名, package 还可以是一个指向有效包名的 http url/git url/文件夹路径。

阅读 [npm的文档](https://docs.npmjs.com/about-packages-and-modules)， 我们会发现package 准确的定义，只要符合以下其中之一条件，就是一个 package:

| **说明**                                                     | **例子**                                               |
| ------------------------------------------------------------ | ------------------------------------------------------ |
| 一个包含了程序和描述该程序的 package.json 文件 的 **文件夹** | ./local-module/                                        |
| 一个包含了 (a) 的 **gzip 压缩文件**                          | ./module.tar.gz                                        |
| 一个可以下载得到 (b) 资源的 **url** (通常是 http(s) url)     | https://registry.npmjs.org/webpack/-/webpack-3.0.0.tgz |
| 一个格式为 `<name>@<version>` 的字符串，可指向 npm 源(通常是官方源 npmjs.org)上已发布的可访问 url，且该 url 满足条件 (c) | webpack@3.0.0                                          |
| 一个格式为 `<name>@<tag>` 的字符串，在 npm 源上该`<tag>`指向某 `<version>` 得到 `<name>@<version>`，后者满足条件 (d) | webpack@latest                                         |
| 一个格式为 `<name>` 的字符串，默认添加 `latest` 标签所得到的 `<name>@latest` 满足条件 (e) | webpack                                                |
| 一个 **git url**, 该 url 所指向的代码库满足条件 (a)          | git@github.com:webpack/webpack.git                     |

<!-- more -->

####  安装本地包

上面表格的定义意味着，我们在共享依赖包时，并不是非要将包发表到 npm 源上才可以提供给使用者来安装。这对于私有的不方便 publish 到远程源（即使是私有源），或者需要对某官方源进行改造，但依然需要把包共享出去的场景来说非常实用。

**场景1: 本地模块引用**

nodejs 应用开发中不可避免有模块间调用，例如在实践中经常会把需要被频繁引用的配置模块放到应用根目录；于是在创建了很多层级的目录、文件后，很可能会遇到这样的代码:

```
const config = require('../../../../config.js');
```

除了看上去很丑以外，这样的路径引用也不利于代码的重构。并且身为程序员的自我修养告诉我们，这样重复的代码多了也就意味着是时候把这个模块分离出来供应用内其他模块共享了。例如这个例子里的 config.js 非常适合封装为 package 放到 node_modules 目录下，共享给同应用内其他模块。

无需手动拷贝文件或者创建软链接到 node_modules 目录，npm 有更优雅的解决方案。

**方案：**

1: 创建 config 包:
 新增 config 文件夹; 重命名 config.js 为 config/index.js 文件; 创建 package.json 定义 config 包

```
{
    "name": "config",
    "main": "index.js",
    "version": "1.0.0"
}
复制代码
```

2: 在应用层 package.json 文件中新增依赖项，然后执行 `npm install`; 或直接执行第 3 步

```
{
    "dependencies": {
        "config": "file:./config"
    }
}
复制代码
```

3: （等价于第 2 步）直接在应用目录执行 `npm install file:./config`

此时，查看 `node_modules` 目录我们会发现多出来一个名为 `config`，指向上层 `config/` 文件夹的软链接。这是因为 npm 识别 `file:` 协议的url，得知这个包需要直接从文件系统中获取，会自动创建软链接到 node_modules 中，完成“安装”过程。

相比手动软链，我们既不需要关心 windows 和 linux 命令差异，又可以显式地将依赖信息固化到 dependencies 字段中，开发团队其他成员可以执行 `npm install` 后直接使用。

### 安装git仓库包

有些时候，我们一个团队内会有一些代码/公用库需要在团队内**不同项目间**共享，但可能由于包含了敏感内容，或者代码太烂拿不出手等原因，不方便发布到源。

这种情况下，我们可以简单地将被依赖的包托管在私有的 git 仓库中，然后将该  git url 保存到 dependencies 中. npm 会直接调用系统的 git 命令从 git 仓库拉取包的内容到 node_modules 中。

[npm 支持的 git url 格式](https://docs.npmjs.com/files/package.json#git-urls-as-dependencies):

```
<protocol>://[<user>[:<password>]@]<hostname>[:<port>][:][/]<path>[#<commit-ish> | #semver:<semver>]
复制代码
```

git 路径后可以使用 # 指定特定的 git branch/commit/tag, 也可以 #semver: 指定特定的 semver range.

例如：

```
git+ssh://git@github.com:npm/npm.git#v1.2.0
git+ssh://git@github.com:npm/npm#sm:^4.0
git+https://isaacs@github.com/npm/npm.git
git://github.com/npm/npm.git#v6.0.27
复制代码
```

**场景3: 开源 package 问题修复**

使用某个 npm 包时发现它有某个严重bug，但也许最初作者已不再维护代码了，也许我们工作紧急，没有足够的时间提 issue 给作者再慢慢等作者发布新的修复版本到 npm 源。

此时我们可以手动进入 node_modules 目录下修改相应的包内容，也许修改了一行代码就修复了问题。但是这种做法非常不明智！

首先 node_modules 本身不应该放进版本控制系统，对 node_modules  文件夹中内容的修改不会被记录进 git 提交记录；其次，就算我们非要反模式，把 node_modules 放进版本控制中，你的修改内容也很容易在下次 team 中某位成员执行 `npm install` 或 `npm update` 时被覆盖，而这样的一次提交很可能包含了几十几百个包的更新，你自己所做的修改很容易就被淹没在庞大的 diff 文件列表中了。

**方案**:

最好的办法应当是 fork 原作者的 git 库，在自己所属的 repo 下修复问题后，将 dependencies 中相应的依赖项更改为自己修复后版本的 git url 即可解决问题。（Fork 代码库后，也便于向原作者提交 PR 修复问题。上游代码库修复问题后，再次更新我们的依赖配置也不迟。）

#### 总结

 希望如上关于npm install安装本地包和git 源包的问题对小伙伴们有帮助！

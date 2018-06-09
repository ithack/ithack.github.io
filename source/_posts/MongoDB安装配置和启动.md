---
title: MongoDB安装配置和启动
date: 2017-05-11 14:38:05
tags: [MongoDB]
---

这周手里没项目，所以闲来没事就吓研究，之前一直想搞一搞node+mongoDB一直没有机会，所以这周就考虑了一下vue+node+mongodb组合做个后台之类的demo，首先我之前完全没有用过mongodb+node的经验所以基本是从0开始摸索，今天我就来说说mongodb的安装配置和启动；

首先必须要有一套node环境这是必须的，我就不多说了，目前大部分前端多少都要搭建node环境来开发；

下面今天的主角登场[MongoDB](https://www.mongodb.com)，先下载[MongoDB](https://www.mongodb.com/dr/fastdl.mongodb.org/win32/mongodb-win32-x86_64-2008plus-ssl-3.4.4-signed.msi/download)，这里的下载连接是官方64位msi包，有其他系统或需求的可直接到官网自行下载，由于我本机是win7(64-bit)，所以我今天是围绕win7来讲解的；

mongoDB 本地安装如图：

![1](\images\mongodb\1.png)

<!--more-->

![1](\images\mongodb\2.png)

![1](\images\mongodb\3.png)

修改安装路径： D:\MongoDB\Server\3.2， 点击 next !(建议修改不要放到太深的目录中去，后边方便操作)

![1](\images\mongodb\4.png)

![1](\images\mongodb\5.png)

![1](\images\mongodb\6.png)

![1](\images\mongodb\7.png)

![1](\images\mongodb\8.png)

安装目录中主要命令行程序都在bin目录下：

```
mongo.exe：客户端，支持js语法

mongod.exe：服务端

mongodump.exe：备份工具

mongorestore.exe：恢复工具

mongoexport.exe：导出工具

mongoimport.exe：导入工具

mongostat.exe：实时性能监控工具

mongotop.exe：跟踪MongDB实例读写时间工具
```

到这里我们已经把mongodb安装完了，但是还需要手动创建文件夹方便数据库的管理：

d:\mongodb
d:\mongodb\db 数据库目录
d:\mongodb\log 日志存放目录
d:\mongodb\log\mongoDB.log 日志

接下来我们要启动mongoDB服务了

D:/mongoDB的bin目录中有许多命令，启动数据库只需要两个命令mongod和mongo： mongod：是mongoDB数据库进程本身 mongo：是命令行shell客户端
启动mongoDB进程：`mongod --dbpath=D:\mongodb\db`,看到如下界面说明mongodb启动成功了（例图如下），接下来就可以在客户端操作数据库了

![9](\images\mongodb\9.png)

红色箭头指向的是端口号，mongodb的端口号默认都是：27017，可能也有些人没有启动成功，这时候看一下报错如果看不懂可以google或评论中回复问题我给看一下；

到这里还不算完，当mongod.exe启动程序被关闭后，mongoDB客户端就无法连接数据库。想要启动还要去bin文件下敲命令行启动mongodb，为了避免每次都要手动启动数据库，可以将mongDB安装为windows服务，让该服务随windows启动而开启，这样，我们在使用mongoDB的时候直接连接数据库就可以了，省去了手动开启服务的繁琐。将mongoDB安装为windows服务并开启的命令：`D:\mongodb\server\3.2\bin>mongod --dbpath=d:\mongodb\db --logpath=d:\mongodb\log\mongoDB.log --install --serviceName "MongoDB"`

开启服务`net start MongoDB`,停止服务`net stop MongoDB`，这种这个过程中或许开启失败，这个时候可能需要你去`C:\Windows\System32`下找到cmd.exe右击管理员运行，然后一路cd到mongodb\bin目录下运行`mongod --dbpath=d:\mongodb\db --logpath=d:\mongodb\log\mongoDB.log --install --serviceName "MongoDB"`这一命令;

到这里我们的mongoDB基本已经配置启动成功了；

最后多说一嘴推荐两个mongodb比较好的可视化工具：

[mongobooster](https://mongobooster.com/)

[NoSQL Manager](https://www.mongodbmanager.com/)

MongoVUE




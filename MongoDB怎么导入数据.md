---
title: MongoDB怎么导入数据
date: 2018-10-08 22:44:24
tags:  
- MongoDB
categories:
- MongoDB
---
使用`node`的过程中使用了`MongoDB`，在MongoDB中怎么直接将数据导入数据库呢？我们只需要一条命令就行了。

>  mongorestore -h dbhost -d dbname path

其中`dbhost`代表你的host，一般在自己的电脑上的话是`127.0.0.1`，`dbname`代表要导入的数据库，可以是不存在的，默认会创建一个新的数据库，`path`就是你的数据库数据存放的位置。**路径中最好不要有中文，以防导入失败。这行命令不需要进入到数据库里面再执行，直接在cmd中运行就可以了。**

想要导出的话也很简单，只要执行下面的命令：

>  mongodump -h dbhost -d dbname -o dbdirectory

`-o dbdirectory`是指导出到哪个目录。但是导出之前我们需要打开我们的数据路服务：

>  mongod --dbpath yourpath

再执行上面的导出命令就OK啦。

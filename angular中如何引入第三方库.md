---
title: angular中如何引入第三方库
date: 2018-08-29 23:06:42
tags:
- Angular
- Typescript
categories:
- Angular
---

当我们想要在angular项目中引入第三方的库的时候我们应该怎么操作？就拿jquery来说吧：

**第一步**
我们需要在项目目录下安装自己所需要的包：

    npm install --save jquery

`--save`和`--save-dev`的区别就不多描述了，安装完成之后我们就可以在项目下的node_modules下面看到它了。

**第二步**
我们需要把我们下载好的包加入到angular的配置文件中，在6.x之前，这个配置文件的名称是`.angular-cli.json`，但是在6.x之后就不是这个名字了，而是换成了`angular.json`，打开这个文件我们就能看到一系列的配置信息。我们需要将自己下载的包的路径引用到`styles`和`scripts`数组下面：
>![](/img/angular/3.png)
在这里填入正确的路径就可以了。这样jquery或者你需要的第三方包就被加入到angular项目中了。

*但是*我们现在还是不能直接使用jquery，因为angular是使用typescript开发的，而jquery的本质是javascript，Typescript是不能直接使用的。我们还需要安装类型描述文件让Typescript认识jquery。

**第三步**
我们通过命令来安装jquery的类型描述文件：

    npm install @types/jquery(这个名字是你需要安装的包的名字) --save-dev

这样我们就能够使用jquery的命令了。
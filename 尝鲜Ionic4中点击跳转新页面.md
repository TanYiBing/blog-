---
title: 尝鲜Ionic4中点击跳转新页面
date: 2018-09-20 11:15:48
tags:
- Ionic4
- Angular
- Typescript
categories:
- Ionic4
---
Ionic已经出了第四版了，不过还在测试阶段，那我们先来玩玩吧。

首先我们用命令行新建一个页面`news`，我们发现，这一版中，新建的page被直接加入到了路由当中：

>![](/img/ionic/1.png)

也就是说我们可以直接使用路由跳转，接下来我们在home页面中加一个按钮，并且给它个点击事件：

>![](/img/ionic/2.png)

此时我们到`home.page.ts`中去定义这个`goNews()`方法。我们手下需要在构造函数里面注册`NavController`，但是我接下来发现，这个类的`push`方法没定义，那我们去看下源码：

>![](/img/ionic/3.png)

果然是没有push方法了，但是有个`navigateForward`方法好像有那么点意思，试试吧：

>![](/img/ionic/4.png)

果然如此，没问题，现在我们就能够跳转啦。

**官网文档现在还不是写的很清楚，想要正常使用还是要查查源码。**
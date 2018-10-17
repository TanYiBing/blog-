---
title: 在VScode中使用Git
date: 2018-10-09 21:31:16
tags:
- VsCode
- Git
categories:
- VsCode
---
在VsCode中使用Git来将我的修改保存到远程仓库的操作：

1. 首先我们在Github上创建一个仓库，否则我们的项目没办法保存到远程仓库。
2. 接下来我一般是在本地创建一个目录，并且用VsCode打开目录。
3. 然后我们开始将本地目录和远程仓库关联起来，首先在VsCode中运行`git init`命令来将文件夹初始化成Git仓库。
4. 然后我们运行`git remote add origin XXX(你的github仓库地址)`,这样就完成了关联。
5. 接着点击图中的master按钮，就可以看到我们的远程仓库，选择它，并在目录下进行工作。>![](/img/git/1.png)
6. 我们也许需要定义一个`.gitignore`文件来忽略掉一些你不想展示给别人看的，或者不重要的文件。
7. 别忘在工作区工作之后要提交更改，并且同步到远程仓库。
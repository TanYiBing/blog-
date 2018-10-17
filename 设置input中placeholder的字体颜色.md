---
title: 设置input中placeholder的字体颜色
date: 2018-10-10 22:22:46
tags:
- CSS
- HTML
categories:
- CSS
---
在一个有背景的div中创建一个form之后，我发现input的背景默认是白色的，这样的话就无法看到背景图，所以我将input的背景调成透明，并加上边框来凸显出input的存在：

	input {
	    background: transparent;
	    border: #ffffff solid 1px;
	}

这样我们的input框就呈现透明的状态。但是我发现input中的placeholder的颜色是黑色的，在我的背景下面不是特别清晰，所以我们需腰改变placeholder的字体颜色：

	::-webkit-input-placeholder{/*Webkit browsers*/
	    color:#999;
	    font-size:16px;
	}
	:-moz-placeholder{/*Mozilla Firefox 4 to 8*/
	   color:#999;
	   font-size:16px;
	}
	::moz-placeholder{/*Mozilla Firefox 19+*/
	   color:#999;
	   font-size:16px;
	}
	:-ms-input-placeholder{/*Internet Explorer 10+*/
	    color:#999;
	    font-size:16px;
	}

我们只要在：前面加上input或者textarea就可以改变placeholder的字体。它的兼容性如下：

>![](/img/html/1.png)
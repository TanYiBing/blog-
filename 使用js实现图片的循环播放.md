---
title: 使用js实现图片的循环播放
date: 2018-10-14 13:38:01
tags:
- JavaScript 
categories:
- JavaScript
---
今天想要实现很多图片的循环播放，我们首先要把所有的图片放到一行上面，具体怎么写css在这就不啰嗦了，最后的html结构如下：

	<div class="container">
		<ul class="first">
			<li>
				<img src="./images/1.jpg" alt="">
			</li>
			<li>
				<img src="./images/2.jpg" alt="">
			</li>
			<li>
				<img src="./images/3.jpg" alt="">
			</li>
		</ul>
		<ul class="second">
			<li>
				<img src="./images/1.jpg" alt="">
			</li>
			<li>
				<img src="./images/2.jpg" alt="">
			</li>
			<li>
				<img src="./images/3.jpg" alt="">
			</li>
		</ul>
	</div>

这里我设置了两个相同的图片ul是为了更好的实现无缝播放，紧接着js代码如下：

	window.onload = function () {
            var container = document.querySelector('.container');
            var first = document.querySelector('.first');
            var second = document.querySelector('.second');
			// 获得一个ul图片循环需要的长度
            var length = ul1.offsetWidth;
            var x = 0;
			// 当一个循环结束时，将x重置为0，第二个ul其实就是起一个弥补上一个的空白的作用。
            var fun = function () {
                first.style.left = x/100 + 'rem';
                second.style.left = (x + length)/100 + 'rem';
                x--;
                if ((x + length) == 0) {
                    x = 0;
                }
            }
            var loop = setInterval(fun, 10);
			// 鼠标移上去暂停滚动
            container.onmouseover = function () {
                clearInterval(loop);
            }
			// 鼠标移开继续滚动
            container.onmouseout = function () {
                loop = setInterval(fun, 10);
            }
        }  

注释里面解释啦，很简单。           
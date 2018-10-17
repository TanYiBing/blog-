---
title: PHP中Session的清除和销毁
date: 2018-03-26 17:48:30
tags:
- php 
- session
categories:
- php
---
# PHP中session清除和销毁
今天在工作中使用session进行用户的登录验证操作，最后要进行用户注销的操作，也就是session的删除，所以查阅文档发现删除会话主要有删除单个会话、删除所有会话和结束当前会话的三种方式，以下就是三种清除session的具体方法：

###	1.删除单个会话
删除单个会话其实就是删单个会话的变量，和数组的操作一样，直接注销$\_SESSION数组的某个元素即可。
例如：$\_SESSION["session\_name"]变量，可以使用unset()函数，代码如下所示：<br/>
`unset($_SESSION["session_name"]);`<br>
注意：在使用unset()时，要注意$SESSION数组中的元素一定不可以可略，即不可以一次注销整个数组，这样会禁止整个会话的功能，一旦使用会将全局变量$_SESSION全局变量销毁，而且没有办法将其恢复，用户也不可能再注册$_SESSION变量，在php文档中也可以发现警告
>**Caution**请不要使用unset($、_SESSION)来释放整个$\_SESSION， 因为它将会禁用通过全局$\_SESSION去注册会话变量
	
### 2.删除所有会话
如果想吧某个用户在session中注册的所有变量都删除，也就是一次删除所有的会话变量，可以通过将一个空数组赋值给$_SESSION来实现，其代码显示如下：<br/>
`$_SESSION = array();`<br>
也可以使用session\_unset 来释放所有的会话变量,使用方法如下：<br>
`void session_unset ( void )`<br>
该函数没有返回值。

### 3.删除当前会话
如果整个会话已经结束，首先应该注销所有会话变量，然后使用 session\_destroy()函数清除结束当前的会话，并清空会话中的所有资源，彻底销毁 Session，其代码如下显示：<br>
`session_destroy()；`<br>
相对于 session\_start() 函数 （创建 Session 文件），session_destroy()函数用来关闭 Session 的运作 （删除 Session 文件），如果成功则返回 TURE，销毁 Session 资料失败则返回 FALSE。但该函数并不会释放和当前 Session 相关的变量，也不会删除保存在客户端 Cookie 中的 Session ID 。
下面是我在代码中实际使用的方法：<br>
<pre>if(isset($_SESSION['username'])){
    $_SESSION = array();
    if(isset($_COOKIE[session_name()])){
       setcookie(session_name(),'',time()-3600);
    }  
   session_destroy();
}</pre>

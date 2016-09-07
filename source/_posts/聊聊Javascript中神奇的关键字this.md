---
title: 聊聊Javascript中神奇的关键字this
permalink: liao-liao-javascriptzhong-shen-qi-de-guan-jian-zi-this
id: 4
updated: '2016-07-09 23:58:26'
date: 2016-07-09 22:51:16
tags:
categories: javascript
---

转载请注明链接
http://www.brusport.com/liao-liao-javascriptzhong-shen-qi-de-guan-jian-zi-this/
this是javascript中比较tricky的关键字之一.如果你不清楚它的工作的原理的话很难去使用它.

下面我们以this在javacript中的事件处理中的表现来解释this的用法.

==Owener==

owner翻译过来过来就是拥有者的意思.
下面我们主要通过解释下面的这个函数doSomething()中的this指向哪个owner，来解释this的用法.

` function doSomething(){
   this.style.color='#cc0000'
 }
`

在javascript中this往往指向的是执行相应的函数的woner(环境或者宿主).或者指向包含该函数的对象.当我们定义简单的在页面中定义函数doSomething()的时候，这时候对应的owner就是页面本身，或者javascript中的window对象.一般的页面元素拥有onclick属性.它们之间的从属关系如下图:
``` bash
------------ window --------------------------------------
|                                          / \           |
|                                           |            |
|                                          this          |
|   ----------------                        |            |
|   | HTML element | <-- this         -----------------  |
|   ----------------      |           | doSomething() |  |
|               |         |           -----------------  |
|          --------------------                          |
|          | onclick property |                          |
|          --------------------                          |
|                                                        |
----------------------------------------------------------
```
<!--more-->

如果你直接执行doSomething(),则this会直接指向window对象.doSomething函数中会试着去改变window的style.color熟悉。由于window对象不存在style熟悉，所以执行doSomething()会报错.

==Copying==

这里的copy是指,把看起来单独的函数 赋给适当的元素属性上.
如果我们最大程度的利用this,我们必须注意让正确的html元素拥有对应的函数.也就说说，我们必须把doSomething函数赋给对应的html元素的onclick属性.

``` bash
element.onclick = doSomething; //element就是指html中的元素，可以是button,a,甚至div...
```

通过这个赋值语句，函数为赋于了html元素的onclick属性上.( 如果一个函数充当某个对象的属性，只是函数可以叫方法).所以这时候你执行onclick的时候,this指向的是element.由于html的元素拥有color属性，所以执行doSomething不会报错，同事颜色会做相应改变.
``` bash
------------ window --------------------------------------
|                                                        |
|                                                        |
|                                                        |
|   ----------------                                     |
|   | HTML element | <-- this         -----------------  |
|   ----------------      |           | doSomething() |  |
|               |         |           -----------------  |
|          -----------------------          |            |
|          |copy of doSomething()|  <-- copy function    |
|          -----------------------                       |
|                                                        |
----------------------------------------------------------
```

因此通过这种方法，你可以最大程度上使用this.每次函数被调用,this就会指向html对应的元素.html元素拥有doSomething()的副本.

==Referring==

然后如果你使用内联html元素事件注册.如下:

``` bash
<element onclick="doSomething()">
```

只是给onclick指向doSomething的调用引用，而不是通过直接给onclick属性赋予doSomething的副本.这区别是很大的.因为此时的onclick属性不包含正式的doSomething函数，而只是包含soSomething的调用

``` bash
doSomething();
```

所以当执行doSomething()的时候，this再一次指向全局 window对象，再一次会执行错误

``` bash
------------ window --------------------------------------
|                                          / \           |
|                                           |            |
|                                          this          |
|   ----------------                        |            |
|   | HTML element | <-- this         -----------------  |
|   ----------------      |           | doSomething() |  |
|               |         |           -----------------  |
|          -----------------------         / \           |
|          | go to doSomething() |          |            |
|          | and execute it      | ---- reference to     |
|          -----------------------       function        |
|                                                        |
----------------------------------------------------------
```

==The difference==

如果你想使用this去访问处理onclick事件的的html元素,你必须保证this关键字写到onclick属性中.只有这样，让this才指向被onclick所注册的html元素.你可以做如下测试:

``` bash
element.onclick = doSomething;
alert(element.onclick)
```

你讲看到
``` bash
function doSomething()
{
	this.style.color = '#cc0000';
}
```
正如你看到的，this关键字出现在onclick方法中.因此this指向html元素.
但是如果你测试如下:
``` bash
<element onclick="doSomething()">
alert(element.onclick)
```

你将看到:
``` bash
function onclick()
{
	doSomething()
}
```

this关键字没有在onclick方法中，所以它没有指向html元素
==Examples - copying==

下面列出直接将this写进onclick方法的写法:

element.onclick = doSomething
element.addEventListener('click',doSomething,false)
element.onclick = function () {this.style.color = '#cc0000';}
<element onclick="this.style.color = '#cc0000';">

==Examples - referring==

下面列出直接将this指向window对象的写法:

element.onclick = function () {doSomething()}
element.attachEvent('onclick',doSomething)
<element onclick="doSomething()">

需要注意的是attachEvent().微软事件注册模式的主要缺点是attachEvent（）会生成一个指向函数调用的引用而不是复制函数本身.

==Combination==

下面我们说个技巧，如果我们既要想用代码内联形式做事件处理注册，又想对应的处理函数中的 this能正确执行，可以参考下面的做法:

``` bash
<element onclick="doSomething(this)">
function doSomething(obj) {
	// this is present in the event handler and is sent to the function
	// obj now refers to the HTML element, so we can do
	obj.style.color = '#cc0000';
}
```

thx

``` bash
refer to http://www.quirksmode.org/js/this.html
```

`bruce liu                           --2016,7.9.  `

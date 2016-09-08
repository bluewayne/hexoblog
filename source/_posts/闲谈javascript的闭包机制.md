---
title: 闲谈javascript的闭包机制
permalink: xian-tan-javascriptde-bi-bao-ji-zhi
id: 5
updated: '2016-07-11 23:25:02'
date: 2016-07-11 21:17:18
tags:
categories: javascript
---


转载请注明链接 http://www.brusport.com/xian-tan-javascriptde-bi-bao-ji-zhi/

闭包可以让javacript coder写出更好，更创造性，更精简的代码.不管你javacript的经验如何,可以确定的是你将不可避免的时常遇到闭包.也许在你的理解范围内，闭包看起来有点复杂，但是通过阅读本文，闭包将可以被很容易得理解.

==What is a closure?==

闭包的英文名是closure,直译就是关闭，关口.javacript中的闭包就是一个内嵌/内部函数，它可以访问它的直接外层函数的变量.闭包有三个scope chain(其实就是域链):它可以访问自己的域范围里面的变量,它可以范围它的直接外层函数的变量，它还可以访问全局变量(一般指window中的变量).

这个内部函数(闭包)不但可以访问它直接外部函数的变量，还可以访问外部函数的参数.但是要注意的是，内部函数不可以访问外部函数参数的arguments对象。

下面我们举个简单的闭包例子
``` bash
function showName(firstName,lastName){
	return (function makeFullName){
		return 'my name is '+firstName+' '+lastName+' ';
	})();
}

showName('liu',bruce);//my name is liu bruce
```
闭包被广泛的用在Node.js,使用在Node.js的异步,非块结构.jquery也广泛的使用闭包.下面举个经典的jquery闭包应用:
``` bash
$(function(){
 var selections=[];
     $(".niners").click(function(){
      selections.push(this.prop("name"));
   })
})

```
<!--more-->

==Closures’ Rules and Side Effects==

下面我们谈谈闭包的使用准则和副作用.

1.闭包可以访问它的外层函数的变量，即使该外层函数已经返回(执行return):
  关于闭包中最难同时也是最重要的特点之一就是：即使闭包的外层函数已经执行return,但是它还是可以访问外层函数的变量.

``` bash
function celebrityName (firstName) {
    var nameIntro = "This celebrity is ";
    // 内部函数lastName可以直接访问外部函数的变量和参数
   function lastName (theLastName) {
        return nameIntro + firstName + " " + theLastName;
    }
    return lastName;
}
var mjName = celebrityName ("Michael"); // 返回的是lastName
mjName ("Jackson"); // This celebrity is Michael Jackson
// 闭包 (lastName) 被调用
// 如结果所示，闭包还是可以直接访问外部函数的变量和参数
```

2.闭包存储外部函数变量的引用;它不存真实的值.在闭包被正式被调用前修改外部函数的变量值将会发生很多有趣的事情.我们可以把闭包这个强大的特征使用在多个有创意的事情上.比如下面例子，我们可以创建一个private变量.
``` bash
function celebrityID () {
    var celebrityID = 999;
    // 我们将返回一个拥有多个内部函数的对象.
    // 所有的内部函数都可以访问外部函数的变量
    return {
        getID: function ()  {
          return celebrityID;// 这个内部函数将会返回celebrityID变量的值​
        },
        setID: function (theNewID)  {
            celebrityID = theNewID;//这个内部函数将会改变外部函数的变量的值
        }
    }
}
​
​var mjID = celebrityID ();
mjID.getID(); // 999​
mjID.setID(567); // 改变外部函数的变量值
mjID.getID(); // 567: 将会返回最新的外部函数的变量值
```

==Closures Gone Awry==

闭包的错误用法.
由于闭包可以访问外部函数的变量，所以在一个循环中使用闭包经常会导致意外的错误.下面举个闭包在循环中的应用例子:
``` bash
​function celebrityIDCreator (theCelebrities) {
    var i;
    var uniqueID = 100;
    for (i = 0; i < theCelebrities.length; i++) {
      theCelebrities[i]["id"] = function (i)  {
        (function(x){
         return uniqueID + x;
       })();
      }
    }
    return theCelebrities;
}
​var actionCelebs = [{name:"Stallone", id:0}, {name:"Cruise", id:0}, {name:"Willis", id:0}];
​var createIdForActionCelebs = celebrityIDCreator (actionCelebs);
​var stalloneID = createIdForActionCelebs [0];
console.log(stalloneID.id()); // 103
```
有没有发现输出的结果有点出乎意料，为什么结果不是100而是103.之前这个例子中，随着这个异步函数被一次次的被调用，i最好被递增到3.用于闭包是通过引用访问外部函数的变量,而不是通过值.所以调用闭包时，最后的返回值都是103.

为了修改这个因为闭包引起的副作用，我们使用"创建一个匿名函数并立刻执行"的方法:
``` bash
​function celebrityIDCreator (theCelebrities) {
    var i;
    var uniqueID = 100;
    for (i = 0; i < theCelebrities.length; i++) {
      theCelebrities[i]["id"] =(function(x){//x参数是i在调用时传过去的
         return function ()  {
           return uniqueID + x;
         }
      })(i);
    }
    return theCelebrities;
}
​var actionCelebs = [{name:"Stallone", id:0}, {name:"Cruise", id:0}, {name:"Willis", id:0}];
​var createIdForActionCelebs = celebrityIDCreator (actionCelebs);
​var stalloneID = createIdForActionCelebs [0];
console.log(stalloneID.id()); // 103
```

refer to http://javascriptissexy.com/understand-javascript-closures-with-ease/

thx

``` bash
bruce liu                           --2016,7.11.
```




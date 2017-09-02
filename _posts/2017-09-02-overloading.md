---
layout: post
title: JavaScript 中的函数重载
date: 2017-09-02
permalink: /:categories/:month-:day-:year/:title.html
excerpt: 原生 JavaScript 是不支持函数重载的，后定义的函数会覆盖掉前面的同名函数，那么为什么需要函数重载，怎样实现 JavaScript 中的函数重载呢 ......
categories: [tech]
tag:
- JavaScript
- 前端
comments: false
---

### 问题一：什么是函数重载？

重载函数是函数的一种特殊情况，为方便使用，C++允许在同一范围中声明几个功能类似的同名函数，但是这些同名函数的形式参数（指参数的个数、类型或者顺序）必须不同，也就是说用同一个运算符完成不同的运算功能。这就是重载函数。重载函数常用来实现功能类似而所处理的数据类型不同的问题。
{: .notice}

当然，不只是 C++，其它面向对象语言如 Java 也对函数重载有着原生支持。

重载的核心是这些同名函数参数的`个数`，`类型`，以及`顺序`必须不同。

有了强大的函数重载，就可以完成更多的功能。

### 问题二：那为什么 JavaScript 不支持函数重载呢？

这要从具体的语言特性说起：

我们知道，在 Java 中，通过函数签名来唯一确定一个函数。方法签名包括：`方法名`、`参数类型`和`参数顺序`、`参数个数`这几个要素（有的人认为`返回值类型`也应该包括在内）。所以，如果两个方法名称相同，但是只要其他要素(例如参数类型、参数个数)不同，编译器就会认为是不同方法。从而可以存在同名的不同方法，导致了函数重载现象。

然而在 JavaScript 中，函数实际上是功能完整的对象，它是一种引用类型，变量名实际上保存的是指向函数的指针，当指针改变了，前面定义的同名函数自然也就被覆盖掉了。

### 问题三：怎么让 JavaScript 支持函数重载呢？

要想让 JavaScript 支持函数重载，那么它必然需要具备下面两种能力：

* 判断传入参数数量的能力
* 判断传入参数类型的能力

熟悉 JavaScript 语言的朋友肯定知道，在 JavaScript 函数中存在一个 arguments 对象。

arguments对象不是一个 Array。它类似于数组，但除了长度之外没有任何数组属性。
{: .notice}

有了 arguments 这个伪数组，我们便可以对传入参数的个数、类型进行判断了。

### 问题四：有没有更好的解决方案？

<figure>
	<img src="{{ site.url }}/assets/img/tech/JohnResig.jpg">
	<figcaption> jQuery 之父 John Resig 会心一笑 </figcaption>
</figure>

他巧妙地利用了闭包，实现了 JavaScript 函数重载。

我们先不看具体代码怎么实现的，先说怎么用

比如说你想给一个 Person 对象实现这么一个方法，名字叫calculate：
* 当传入一个参数，返回该参数
* 当传入两个参数，返回参数的积
* 当传入三个参数，返回参数的和

这三个函数分别写出来应该是这样：
~~~js
calculate1 (a) {return a;}

calculate2 (a, b) {return a*b;}

calculate3 (a, b, c) {return a+b+c;}
~~~

用 John Resig 给我们提供的 addMethod 方法我们只需要这样写：

~~~js
addMethod(Person.prototype, calculate, calculate1);
addMethod(Person.prototype, calculate, calculate2);
addMethod(Person.prototype, calculate, calculate3);
~~~

恩我们的函数重载，就完成了。

啊？那这个 `addMethod` 方法，具体是怎么实现的呢？

直接看代码：

~~~js
function addMethod(object, name, fn){
    var old = object[ name ];
    object[ name ] = function(){
        if ( fn.length == arguments.length )
            return fn.apply( this, arguments );
        else if ( typeof old == 'function' )
            return old.apply( this, arguments );
    };
}
~~~

这个函数的核心，就是利用 old 变量，将绑定前后的函数连接起来。

undefined <-------- calculate1 <-------- calculate2 <-------- calculate3

这种重载方式也有它的缺点，它只能处理输入参数个数不同的情况，并不能区分参数的类型、名称等其他要素。
{: .notice}

### 参考链接：<a href="https://johnresig.com/blog/javascript-method-overloading/" target="blank">《JavaScript Method Overloading》</a>



## 注:

* 本站所有内容均属个人学习记录所用，一些内容来源网络疏于标注，若您发现有任何侵犯您权益的行为，请立即联系我删除，为您带来不便深表歉意
* 本人尚属前端小白一枚，若您在阅读中有任何技术问题或者发现任何错误，请及时联系我予以斧正，还望您不吝赐教
* **邮箱**: email@zhangqingtian.com
{: .notice}

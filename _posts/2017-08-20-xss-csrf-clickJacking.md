---
layout: post
title: 三种常见web攻击方式，xss、csrf和clickJacking
date: 2017-08-20
permalink: /:categories/:month-:day-:year/:title.html
excerpt: 最近看了360的前端面试题，其中涉及到了很多web安全的知识，突然意识到自己对这方面知之过少，所以花了一下午时间简单了解梳理了web安全相关的基础知识，在这里记录三种比较『纯』前端的攻击方式及其相应的防御方法......
categories: [tech]
tag:
- Web安全
- 前端
comments: false
---


> 最近看了360的前端面试题，其中涉及到了很多web安全的知识，突然意识到自己对这方面知之过少，所以花了一下午时间简单了解梳理了web安全相关的基础知识，在这里记录三种比较『纯』前端的攻击方式及其相应的防御方法

## 一、xss跨站脚本攻击(Cross Site Scripting)

1. 如果网站带有评论功能并将评论内容直接存到服务器中，那么显示评论的时候就可能遭到之前恶意用户恶意评论的攻击
2. 原理主要是通过在评论中输入html标签，如<script></script>标签，就相当于往你的网页中嵌入了一段脚本
3. 理论上，所有可输入的地方没有对输入数据进行处理的话，都会存在xss漏洞

例如在评论框中输入：
~~~ html
<script>
  while(true) {
    alert('Welcome!');
  }
</script>
~~~

如此一来，当再次刷新页面的时候，你的网站就会一直显示"Welcome"

或者：
~~~ html
<script>
  alert(document.cookie);
</script>
~~~

这段脚本将恶意读取你的cookie信息

### 防御方法：
* 要坚持一个原则：永远不要相信用户的输入，对每一条输入都进行验证
* 可以用正则表达式验证输入内容的合法性

## 二、csrf跨站请求伪造(Cross-site Request Forgery)

攻击者盗用你的身份，发送恶意请求
要完成一次csrf攻击，被攻击者需要完成两个操作：
1. 登录收信任网站A，并在本地生成cookie
2. 在与A的会话结束前，访问危险网站B（危险网站B甚至可能是一个存在漏洞的受人信任的网站）

### 防御方法：
* 在客户端表单增加伪随机数（但不能防止对方再利用xss攻击手段获得你的cookie）
* 在每次做类似提交表单的操作时都通过验证码进行验证
* 为每个表单设置不同的伪随机数（但会有并行会话兼容问题）

## 三、点击劫持(Click Jacking)

在我们浏览网页时经常会出现一些令人讨厌的信息，比如一些中奖通知或者色情信息，但在网页中散布这些内容却是不法分子用来诱惑被攻击者点击的常用手段，

点击劫持就是利用透明的iframe或者被覆盖的iframe，通过诱骗用户在该网页上点击某些按钮，触发iframe页面上的点击操作

### 防御方法：

#### 1. 既然点击劫持的核心是iframe的嵌套，那么完全可以通过写一段javascript代码来禁止iframe的嵌套：

~~~ html
<script type="text/javascript">
   if (window!=top) // 判断当前的window对象是否是top对象
   top.location.href =window.location.href; // 如果不是，将top对象的网址自动导向被嵌入网页的网址
</script>
~~~

注：此段代码来自阮一峰老师的文章
<a href="http://www.ruanyifeng.com/blog/2008/10/anti-frameset_javascript_codes.html" target="blank">《防止网页被嵌入框架的代码》</a>

#### 2. X-FRAME-OPTIONS是目前最可靠的方法。

X-FRAME-OPTIONS是微软提出的一个http头，专门用来防御利用iframe嵌套的点击劫持攻击。

并且在IE8、Firefox3.6、Chrome4以上的版本均能很好的支持。

这个头有三个值：

* DENY        // 拒绝任何域加载
* SAMEORIGIN  // 允许同源域下加载
* ALLOW-FROM  // 可以定义允许frame加载的页面地址

## 注:

* 本站所有内容均属个人学习记录所用，一些内容来源网络疏于标注，若您发现有任何侵犯您权益的行为，请立即联系我删除，为您带来不便深表歉意
* 本人尚属前端小白一枚，若您在阅读中有任何技术问题或者发现任何错误，请及时联系我予以斧正，还望您不吝赐教
* **邮箱**: email@zhangqingtian.com
{: .notice}

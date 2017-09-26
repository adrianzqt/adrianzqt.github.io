---
layout: post
title: 【翻译】Node.js 中的未捕获异常
date: 2017-09-26
permalink: /:categories/:month-:day-:year/:title.html
excerpt: 本篇文章是翻译 George Ornbo 在 2012.11.14 的文章 ......
categories: [tech]
tag:
- Node.js
comments: false
---

## Node.js 中的未捕获异常

>处理 Node.js 中的未捕获异常（Uncaught Exceptions）不是很容易

### 目录:
* 未捕获异常带来的问题
* 怎样处理未捕获异常
* 一个没有未捕获异常的应用
* 使你的应用崩溃
* 假装没看见错误？
* 应用崩溃，打印日志，然后重启
* 使用 Domains 模块【译者注：现已废弃】
* 结论

### 1. 未捕获异常带来的问题

由于 Node.js 的单线程特性，未捕获异常是一个应用开发过程中值得注意的问题。Node.js 遵循错误第一，数据第二的回调模式。我们经常在看到这样的例子：当回调函数返回一个错误对象，那就立即抛出这个错误。

~~~js
var fs = require('fs');

fs.readFile('somefile.txt', function (err, data) {
  if (err) throw err;
  console.log(data);
});
~~~

如果你运行这个程序，而且假设你没有 `somefile.txt` 这个文件，一个错误将被抛出。

~~~js
Error: ENOENT, open 'somefile.txt'
~~~

这将导致进程崩溃进而影响到整个APP。
这是有意为之，Node.js 不打算把你的应用和服务分隔开。

### 2. 怎样处理未捕获异常

处理未捕获异常的最佳方式是什么呢？有非常多方法：

* 你的应用不应该有未捕获错误，这很疯狂。
* 你应该让你的应用在崩溃后找到未捕获异常，然后修复它们，这也很疯狂。
* 对错误视而不见，不处理它---这是大多数人做的，然而这糟透了。
* 你应该让你的应用在崩溃后打印出错误日志，然后借用 `upstart` , `forever` , `monit` 之类的东西重启进程。这个方法很实用。
* 【译者注：现已废弃】你应该开始使用 Domains 模块来处理错误，这是必由之路，虽然这还是 Node.js 的一个试验性功能。

现在我们来详细展开这些方法。

### 3. 一个没有未捕获异常的应用

「没有未捕获异常的应用」这个概念对我来说很怪异，任何应用在某个时刻都会有异常而且可能是未捕获的异常。如果你坚持这个观点并把错误扔给用户，那么我觉得你要做好半夜接到电话被告知服务崩溃掉了的准备。


### 4. 使你的应用崩溃

The only defence I can find in this opinion is the fail fast argument. You are going to fix your application quickly if it unavailable. If an application without uncaught exceptions is denial letting your application crash is acceptance. But you are still pushing exception handling onto your users.（原谅我实在想不出怎么翻译这段，如果你有好的想法，请尽快联系我！）

### 5. 假装没看见错误？

很多人这样做：

~~~js
process.on('uncaughtException', function (err) {
  console.log(err);
})
~~~

这很糟糕，当一个未捕获异常被抛出，你应该意识到你的应用处在一个不正常的状态，这种情况下你无法可靠地运行你的程序。

最初提出 process.on 事件的 `Felix Geisendörfer` 现在倡议去除它。

### 6. 应用崩溃，打印日志，然后重启

通过这个方法你可以让你的应用在发生未捕获异常时立即崩溃，然后利用 `forever` 或 `upstart` 这样的工具（几乎可以）立即重启。Node.js 将会把异常写入 `STERR` 所以你可以把异常重定向到一个日志文件稍晚再通过它拿到错误。这种方法的缺点是，对于错误发生在你的代码之外的 `i/o` ，不能提供一种优雅的方法来处理临时停电或者网络 `i/o` 出错的场景。这真是一个利器！--- 重启应用并重试。如果你把这种策略与 `cluster module` 相结合，node 可以自动重启任何抛出错误的 children 并且打印出错误。

~~~js
var cluster = require('cluster');

var workers = process.env.WORKERS || require('os').cpus().length;

if (cluster.isMaster) {

  console.log('start cluster with %s workers', workers);

  for (var i = 0; i < workers; ++i) {
    var worker = cluster.fork().process;
    console.log('worker %s started.', worker.pid);
  }

  cluster.on('exit', function(worker) {
    console.log('worker %s died. restart...', worker.process.pid);
    cluster.fork();
  });

} else {

  var http = require('http');
  http.createServer(function (req, res) {
    res.end("Look Mum! I'm a server!\n");
  }).listen(3000, "127.0.0.1");

}

process.on('uncaughtException', function (err) {
  console.error((new Date).toUTCString() + ' uncaughtException:', err.message)
  console.error(err.stack)
  process.exit(1)
})
~~~

### 7. 使用 `Domains` 模块【译者注：现已废弃】

`Domains` 是 `Node.js v0.8` 版本中新增的一个试验性特性，它使得异常处理变得更加灵活和精确。下面是刚才那个文件不存在的例子，通过使用 `domains` 你可以为一个特定的 domain 触发 error 事件，你还可以针对不同的场景使用不同的异常处理。这使得你根据异常的发生地点来对应地处理它们。如果退出进程像是用榔头敲碎坚果，那么这就像一把精确的手术刀为你提供对程序完全的控制。

~~~js
var domain = require('domain');
var d = domain.create();
var fs = require('fs');

d.on('error', function(err) {
  console.error(err);
});

d.run(function() {
  fs.readFile('somefile.txt', function (err, data) {
    if (err) throw err;
    console.log(data);
  });
});
~~~

### 8. 结论

如果你在产品环境运行 Node.js 你起码应该对如何处理异常有一个想法。目前为止我相信当异常被抛出时，大多数人只是重启应用（也许是优雅地重启），`Domains` 为应用提供了一种更聪明的面对异常的能力，异常处理器可能会选择简单的清理、关闭某些连接，最坏的情况下，退出进程。关键点就在于你有了选择。

我抛下榔头拾起手术刀的时候应该已经到了。

### 原文链接：<a href="https://shapeshed.com/uncaught-exceptions-in-node/" target="blank">《Uncaught Exceptions in Node.js》</a>



## 注:

* 本站所有内容均属个人学习记录所用，一些内容来源网络疏于标注，若您发现有任何侵犯您权益的行为，请立即联系我删除，为您带来不便深表歉意
* 本人尚属前端小白一枚，若您在阅读中有任何技术问题或者发现任何错误，请及时联系我予以斧正，还望您不吝赐教
* **邮箱**: email@zhangqingtian.com
{: .notice}

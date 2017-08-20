---
layout: post
title: 【翻译】React16 中的错误处理
date: 2017-8-07
permalink: /:categories/:month-:day-:year/:title.html
excerpt: 随着 React16 发布的临近，我们想要宣告 React16 将怎样在组件内部处理JavaScript错误，这些变化将包括在 React16 beta 版本中，当然也是 React 16 的一部分......
categories: [tech]
tag:
- React
- 前端
comments: false
---


> 本篇文章是翻译的 React 官方文档，仅作个人学习参考使用，源地址
<a href="https://facebook.github.io/react/blog/2017/07/26/error-handling-in-react-16.html" target="blank">【戳这】</a>

## React16 中的错误处理
随着 React16 发布的临近，我们想要宣告 React16 将怎样在组件内部处理javascript错误，这些变化将包括在 React16 beta 版本中，当然也是 React 16 的一部分，BTW：我们刚释出了 React16 的 beta 版本，快来<a href="https://github.com/facebook/react/issues/10294" target="blank">【试试】</a>吧！

## React15 及更早版本的错误表现
在过去，组件内部的JavaScript错误总是扰乱组件状态以至于在下一次渲染中出现一些奇怪的错误，而且这些错误常常由应用中一些更早的错误引起，然而 React 没有提供一种在组件内部解决这些问题的优雅方案，也不能从错误中恢复

## 认识 Error Boundaries
用户界面某处的 JavaScript 错误不应该使整个 APP 崩溃，为了解决这个问题，React16 引进了一个新的概念 —— Error Boundaries

Error Boundaries 可以捕获在其子组件树里抛出的任何错误，打印这些错误，并且返回一个展示错误的界面而不是崩溃掉的页面。Error Boundaries 可以在渲染过程中、在生命周期方法中、以及其子组件的 constructor 中捕获错误

添加了一个叫做 `componentDidCatch(error, info)` 的新生命周期方法的组件就叫做 Error Boundaries

~~~ruby
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  componentDidCatch(error, info) {
    // Display fallback UI
    this.setState({ hasError: true });
    // You can also log the error to an error reporting service
    logErrorToMyService(error, info);
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
~~~

你仍然可以像正常组件一样使用它

~~~ruby
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
~~~

`componentDidCatch()`方法就像是为组件定制的 JavaScript 中的 `catch {}` 代码块，只有通过 class 定义的组件才能成为 Error Boundaries，在实践中，绝大多数时候你都希望只定义一个 Error Boundary 然后在应用的各个角落使用它

请注意 Error Boundaries 只能捕捉组件树中位于它下方的组件中的错误
{: .notice}

一个 Error Bundary 不能捕获自己内部抛出的错误，如果一个 Error Bundary 没有成功的渲染错误信息，那么这个错误将传播到离它最近的上层 Error Bundary 中，这也类似于 `catch {}` 代码块在 JavaScript 中的工作方式

## 在线示例

你可以试试<a href="https://codepen.io/gaearon/pen/wqvxGa?editors=0010" target="blank">【这个示例】</a>

## Error Boundaries 应该放置在哪儿？

Error Boundaries 的拆分粒度完全取决于你自己，你可以封装顶层的路由组件用来显示错误信息给用户，就像应用崩溃时服务器端框架所做的那样。你也可以在 Error Bundary 中封装一些小工具用来阻止整个应用的崩溃

## 未捕获错误的新行为

这个变化有一个很重要的含义：

对 React16 来说，一个未捕获的错误会导致整个应用不能被挂载
{: .notice}

我们曾就此展开争论，但在实践中我们发现与其展示一个错误的界面，还不如把它完全移除。比如说，一个显示错误的通讯软件可能会导致用户把讯息发送给错误的人。类似的，一个展示错误金额的支付软件还不如什么也不展示

这个变化意味着当你迁移到 React16 ，你可能会发现很多以前没有被发现的错误，增加 Error Boundaries 可以增强你的应用发生错误时的用户体验

Facebook 把侧边栏，信息栏，对话框，和信息输入框放到了不同的 Error Boundaries 中，这样一来，当一个地方发生错误，其余的地方仍然可以正常工作

我们也建议你使用 JS 错误报告服务（或者弄一个你自己的），这样你就可以及时获知应用中未处理的错误，并修复它们

## 组件堆叠信息

开发模式中 React16 会在控制台打印出渲染过程中发生的所有错误，哪怕应用偶然地吞并了它们。除了错误信息和 JavaScript 堆栈，它还提供组件的堆叠信息，所以你就可以精确地知道到底是应用的哪个地方出了错误

<figure>
	<img src="{{ site.url }}/assets/img/tech/error-boundaries-stack-trace.png">
</figure>

你还可以在组件堆叠信息中找到文件名称和代码行数，<a href="https://github.com/facebookincubator/create-react-app" target="blank">【Create React App】</a>项目默认提供这个功能

<figure>
	<img src="{{ site.url }}/assets/img/tech/error-boundaries-stack-trace-line-numbers.png">
</figure>

如果你不打算使用 Create React App ，你也可以把<a href="https://www.npmjs.com/package/babel-plugin-transform-react-jsx-source" target="blank">【这个补丁】</a>手动加入到你的 babel 配置文件中

## 为什么不用现成的 try/catch ？

try/catch 适用于命令：

~~~ruby
try {
  showButton();
} catch (error) {
  // ...
}
~~~

然而，React 组件是陈述性的，它指明要渲染的内容

~~~html
<Button />
~~~

Error Boundaries 保留了 React 的陈述性，如同你所期望一样工作。比如，就算一个`componentDidUpdate`生命周期函数因为一个组件内部很深的`setState`方法导致了错误，它也会正确地被传递到离它最近的 Error Bundary 处

## 和 React15 的不同

React15 包含了非常有限的对 Error Boundaries 的支持，它也有一个不同的方法名字：`unstable_handleError`。这个方法不管用了，你应该从 React16 beta 版首次释出就把它升级到`componentDidCatch`

我们还为你提供了一个<a href="https://github.com/reactjs/react-codemod#error-boundaries" target="blank">【代码模板】</a>












## 注:

* 本站所有内容均属个人学习记录所用，一些内容来源网络疏于标注，若您发现有任何侵犯您权益的行为，请立即联系我删除，为您带来不便深表歉意
* 本人尚属前端小白一枚，若您在阅读中有任何技术问题或者发现任何错误，请及时联系我予以斧正，还望您不吝赐教
* **邮箱**: email@zhangqingtian.com
{: .notice}

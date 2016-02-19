# 路由

在web应用里面，是通过路由和URL来驱动UI的切换的。

在传统web技术栈里，服务器每次渲染一个HTML页面，URL是用户访问应用程序的基本入口。 用户通过URL访问web应用，该请求以HTTP形式传给服务器，然后服务器通过服务端路由加以响应。

和传统不一样的是，Meteor采用 **data on the wire** 的原则，客户端只采用DDP和服务端通信。正常情况下，当程序启动以后，会初始化一系列的订阅来渲染程序。当用户进行交互时，会加载不同的订阅，但是URL可以一直不变。正常情况下，服务端并不是URL驱动的，URL只用来表示用户当前使用的客户端状态。

## 使用FlowRouter

可以通过添加kadira:flow-router来使用FlowRouter。 使用一个路由的基本目的是根据某个URL，执行一个操作，比如

```js
FlowRouter.route('/lists/:_id', {
  name: 'Lists.show',
  action(params, queryParams) {
    console.log("Looking at a list?");
  }
});
```

和常规的服务器端路由不同的是，这里路由切换时，并不需要对服务器做任何额外的请求。

## 路由参数

首先看下面这个路由，有一段内容具备一个`:`前缀，它是路由参数(url parameter)，而URL里面也可以通过`?`符号，引入查询字符串(query string)，提供这些查询字符串，路由会分割成一系列命名的参数，叫做`queryParams`。

```js
'/lists/:_id'
```

## 获取路由信息

FlowRouter有一些可以获得路由信息的API，比如

* `FlowRouter.getRouteName()` 获得路由的名称
* `FlowRouter.getParam(paramName)` 获得单个路由参数的值
* `FlowRouter.getQueryParam(paramName)` 获得单个查询参数的值

## BlazeLayout布局

使用FlowRouter时可以很方便的用 `kadira:blaze-layout` 来做渲染。首先需要定义一个“layout”组件，然后使用`Template.dynamic`来进行组合渲染，具体实例如下


```html
<template name="App_body">
  ...
  {{> Template.dynamic template=main}}
  ...
</template>
```

```js
FlowRouter.route('/lists/:_id', {
  name: 'Lists.show',
  action() {
    BlazeLayout.render('App_body', {main: 'Lists_show_page'});
  }
});
```

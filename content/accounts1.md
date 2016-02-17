# 账户系统

Meteor内置了一个可以直接使用的账户系统及其界面，并基于它实现了多个第三方账号的集成。


## 核心库


### accounts-base

账户系统的核心内容在 **accounts-base** 包里面实现，它包括

* **集合** 一个users数据集及其对应的schema，这个数据集可以通过Meteor.users来访问，而客户端的单例Meteor.userId()和Meteor.user()则可以表示登录状态
* **方法** 一系列辅助方法来跟踪登录状态，登录，登出以及数据验证
* **接口** 用来注册新的登录的API，方便其他代码包和账号系统整合。

 一般的应用程序通过添加**accounts-password**、**accounts-weibo** 等来自动引用**accounts-base**。

### userId

用户登录以后可以一直获得 this.userId 这个变量，并通过它来获取登录状态，这个变量在Method，Publication以及client端代码里面都可以调用。

## 基本使用

可以通过 **accounts-ui** 来快速方便地使用账号系统的基本功能，使用方法非常简单：

1. 添加包 accounts-ui
2. 在Blaze模板里面使用 `{{> loginButtons}}`
3. 选择下面合适的登录方式，它们会自动和accounts-ui集成
  * accounts-password
  * accounts-weibo
  * accounts-google
  * accounts-github
  * accounts-twitter
  * accounts-facebook
  * accounts-meetup


## 使用useraccounts

useraccounts提供了一系列功能更加丰富且可配置的功能包，从而方便程序与其他部分做更加自然的整合。这个系列的包是目前Meteor生态圈最强大的账户系统工具包。

核心包 **useraccounts:core** 是独立于任何模板和路由的，它也内置了几个预置的界面风格，包括 **Bootstrap**、**Semantic UI**、**Material UI** 等 ，如果需要自定义风格，可以使用 **useraccounts:unstyled** 。


### 直接使用

 如果不想做过多的配置，可以直接在代码里面加入下面的模板，即可达到使用的效果

 ```html
 {{> atForm}}
 ```

### 定制模板

对某些应用来说，现成的login模板就够用了，但是更多的情况下是需要做一些定制的。可以通过模板替换的方法来完成这个目的，这里会用到 **aldeed:template-extension** 这个包。

我们首先找到需要被替换的模板，比如

```html
<template name="atPwdFormBtn">
  <button type="submit" class="at-btn submit {{submitDisabled}}" id="at-btn">
    {{buttonText}}
  </button>
</template>
```

然后定义一个新的模板，比如

```html
<template name="override-atPwdFormBtn">
  <button type="submit" class="btn-primary" id="at-btn">
    {{buttonText}}
  </button>
</template>
```

这里需要注意几点：

1. 和原来模板做一样的渲染，使用一样的helpers，比如这里的 `{buttonText}}`
2. 保持所有的id不变， 这些id在事件处理时需要用到，比如这里的 `at-btn`

最后，通过一行指令替换掉那个模板

```js
Template['override-atPwdFormBtn'].replaces('atPwdFormBtn');
```

### 定制路由

除了模板以外，路由也可以自定义。

首先需要配置渲染accounts模板的布局，如下使用 `App_body` 模板作为所有账户相关页面的布局模板，这个模板有一个叫做 `main` 的区域。


```js
AccountsTemplates.configure({
  defaultTemplate: 'Auth_page',
  defaultLayout: 'App_body',
  defaultContentRegion: 'main',
  defaultLayoutRegions: {}
});
```

然后可以配置一些路由

```js
// Define these routes in a file loaded on both client and server
AccountsTemplates.configureRoute('signIn', {
  name: 'signin',
  path: '/signin'
});

AccountsTemplates.configureRoute('signUp', {
  name: 'join',
  path: '/join'
});

AccountsTemplates.configureRoute('forgotPwd');

AccountsTemplates.configureRoute('resetPwd', {
  name: 'resetPwd',
  path: '/reset-password'
});
```

然后则可以渲染login页面如下


```html
<div class="btns-group">
  <a href="{{pathFor 'signin'}}" class="btn-secondary">Sign In</a>
  <a href="{{pathFor 'join'}}" class="btn-secondary">Join</a>
</div>
```

进一步的使用可以参考[`useraccounts:flow-routing`](https://github.com/meteor-useraccounts/flow-routing#routes)的文档，它对blaze和react都有着比较好的支持。

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

进一步路由使用可以参考[`useraccounts:flow-routing`](https://github.com/meteor-useraccounts/flow-routing#routes)的文档，它对blaze和react都有着比较好的支持。

对于useraccounts的进一步使用可以参考[帮助文档](https://github.com/meteor-useraccounts/core/blob/master/Guide.md)。


## 加载和显示用户

### 当前登录用户



**客户端 Meteor.userId()**

一旦用户登录，则可以在客户端获得全局的当前用户信息

* `Meteor.userId()` 可以获得当前用户的id信息
* `Meteor.user()` 等价于 `Meteor.users.findOne({Meteor.userId()})`
* `{{currentUser}}` 在模板里可以通过这个值来获得`Meteor.user()`对应的值

**服务端this.userId**

服务端并没有全局的登录用户信息，但是Meteor会跟踪每个Meteor call的环境，所以在method里面可以使用 `Meteor.userId()`, 该函数的返回值取决于谁调用它。不过不可以在publication里面使用`Meteor.userId()`。

我们推荐在Method的上下文以及publication里面使用`this.userId`，并可以使用这个变量作为参数。 总而言之，可以在服务端用`this.userId`，包括publication里面。


总体而言，可以参考如下两段代码：

```js
// Accessing this.userId inside a publication
Meteor.publish('Lists.private', function() {
  if (!this.userId) {
    return this.ready();
  }

  return Lists.find({
    userId: this.userId
  }, {
    fields: Lists.publicFields
  });
});
```

```js
// Accessing this.userId inside a Method
Meteor.methods({
  'Todos.methods.updateText'({ todoId, newText }) {
    new SimpleSchema({
      todoId: { type: String },
      newText: { type: String }
    }).validate({ todoId, newText }),

    const todo = Todos.findOne(todoId);

    if (!todo.editableBy(this.userId)) {
      throw new Meteor.Error('Todos.methods.updateText.unauthorized',
        'Cannot edit todos in a private list that is not yours');
    }

    Todos.update(todoId, {
      $set: { text: newText }
    });
  }
});
```

### `Meteor.users`集合

Meteor自带了保存用户的MongodB数据集`users`，可以在代码里面通过访问`Meteor.users`来访问，有几个注意事项：

* 在发布user数据时，不要发布key等数据。
* 当通过email或者user来查找用户时，不要直接查找数据库，而是通过这两个函数来达到大写不敏感的查询结果：`Accounts.findUserByUsername`、`Accounts.findUserByEmail`。


## 自定义用户数据

在实际应用中时常需要针对用户自定义一些数据，在关系型数据库里面，往往是通过独立的表来保存这些数据的。不过MongoDB并不善于做级联查询，我们经常把这些数据直接保存在`users`这个集合中以提高程序的效率。

### 把数据添加在`user`的顶级字段里

添加自定义字段的最优方式是在`Meteor.users`集合里添加一个全新命名的顶级字段。比如如下服务端代码添加了一个邮寄地址：


```js
const newMailingAddress = {
  addressCountry: 'US',
  addressLocality: 'Seattle',
  addressRegion: 'WA',
  postalCode: '98052',
  streetAddress: "20341 Whitworth Institute 405 N. Whitworth"
};

Meteor.users.update(userId, {
  $set: {
    mailingAddress: newMailingAddress
  }
});
```

### 用户注册时添加字段

有的时候需要在创建用户时初始化某些值，这个工作可以在`Accounts.onCreateUser`里进行，比如

```js
// Generate a todo list for each new user
Accounts.onCreateUser((options, user) => {
  // Generate a user ID ourselves
  user._id = Random.id(); // Need to add the `random` package

  // Use the user ID we generated
  Lists.createListForUser(user._id);

  // Don't forget to return the new user object at the end!
  return user;
});
```

### 不要使用profile

用户注册以后有一个默认的`profile`字段，因为它字段很容易在客户端写入，所以人们常常喜欢把自定义字段放在这里。但这并不是一个好习惯，有很多安全隐患。

### 拒绝客户端修改user数据

出于安全考虑，尽量避免在客户端修改user数据，可以通过如下代码做出强制约束

```js
Meteor.users.deny({
  update() { return true; }
});
```

### 发布自定义数据

为了获得自定义数据，需要从服务器端发布它们，但是由于user数据里面保存了一些重要数据，所以需要明确哪些数据被发布了。下面例子，发布了initials数据：

```js
Meteor.publish('Meteor.users.initials', function ({ userIds }) {
  // Validate the arguments to be what we expect
  new SimpleSchema({
    userIds: { type: [String] }
  }).validate({ userIds });

  // Select only the users that match the array of IDs passed in
  const selector = {
    _id: { $in: userIds }
  };

  // Only return one field, `initials`
  const options = {
    fields: { initials: 1 }
  };

  return Meteor.users.find(selector, options);
});
```

## 权限

有2种权限形式
* 基于角色的权限
* 基于文档的权限

### 基于角色的权限

最常见的角色权限包是`alanning:roles`， 如下代码给用户赋予了特定的权限


```js
// Give Alice the 'admin' role
Roles.addUsersToRoles(aliceUserId, 'admin', Roles.GLOBAL_GROUP);

// Give Bob the 'moderator' role for a particular category
Roles.addUsersToRoles(bobsUserId, 'moderator', categoryId);
```

如果需要判断用户是否允许删除一个post，则可以通过如下代码验证

```js
const forumPost = Posts.findOne(postId);

const canDelete = Roles.userIsInRole(userId,
  ['admin', 'moderator'], forumPost.categoryId);

if (! canDelete) {
  throw new Meteor.Error('unauthorized',
    'Only admins and moderators can delete posts.');
}

Posts.remove(postId);
```

### 基于文档的权限

如下所示，我们首先通过文档的owner来判断用户是否有权限

```js
Lists.helpers({
  // ...
  editableBy(userId) {
    if (!this.userId) {
      return true;
    }

    return this.userId === userId;
  },
  // ...
});
```

然后根据用户是否允许修改来进行判断

```js
const list = Lists.findOne(listId);

if (! list.editableBy(userId)) {
  throw new Meteor.Error('unauthorized',
    'Only list owners can edit private lists.');
}
```


## 练习

1. 为项目添加基本的登录
2. 使用useraccounts来时实现复杂的登录系统
3. 使用useraccounts和FlowRouter来时实现基于React的登录系统
4. 自定义字段及其发布

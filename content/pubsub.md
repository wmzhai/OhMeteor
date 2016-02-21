# 发布订阅

在传统的Web应用里，客户端和服务端是通过‘request-response’方式进行通信的。 客户端一般是通过RESTful的HTTP请求服务器，并获得HTML或者JSON格式数据的反馈。而Meteor基于则DDP协议建立，允许数据做双向的通信，建立一个Meteor应用并不需要构建REST接口，而是创建一个人publication，然后就可以将数据从服务器端推送到客户端了。

* **发布publication**  发布是一个服务端的命名API，他可以构建数据集发送到客户端。
* **订阅subscription**  客户端可以点阅服务端的发布，并获取数据；第一次初始化时会获得一批数据，而后面当数据发生变化时，获得的数据也会获得更新。

## 定义发布

发布需要在服务端代码里定义，比如

```js
Meteor.publish('Lists.public', function() {
  return Lists.find({
    userId: {$exists: false}
  }, {
    fields: Lists.publicFields
  });
});
```

上述代码有2个注意点：
1. 这个发布的名称是 `Lists.public`，客户端可以通过名称订阅
2. 发布返回的是Mongo的游标`cursor`, 而且仅仅返回指定的部分字段

##  发布的参数

每个发布可以使用2类参数

1. `this`, 包含当前DDP连接的信息。 比如，可以通过`this.userId`获得当前用户的`_id`
2. 通过`Meteor.subscribe`传入的参数。

特别注意的是： 以为我们需要通过`this`参数来获得上下文信息，所以我们需要用`function() {}` 形式的函数，而不是ES6的`()=>{}`写法。 （这一点在未来的API里面可能获得改进）

下面看一段代码：

```js
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

这段代码只会将用户自己的list发布给他。 当用户没有登录是，直接调用 `this.ready()` 以表示已经将所有数据发送完毕。 如果不返回cursor，也不调用`this.ready()`，这个订阅就一直不会变成ready，会永远处于loading状态。

下面是一个具体的带参数发布及其订阅的例子：

```js
Meteor.publish('Todos.inList', function(listId) {
  // We need to check the `listId` is the type we expect
  new SimpleSchema({
    listId: {type: String}
  }).validate({ listId });

  // ...
});
```

```js
Meteor.subscribe('Todos.inList', list._id);
```

## 订阅数据

在客户端通过名称调用`Meteor.subscribe()`来订阅数据，完成订阅后，客户端会打开一个订阅，然后服务端会自动发送数据到客户端。 这个函数会返回一个"subscription handle"句柄，这个句柄有一个`.ready()`的属性，当订阅ready时，它的值是true。

```js
const handle = Meteor.subscribe('Lists.public');
```

### 停止订阅

当订阅完成时，你必须调用`.stop()`方法来结束订阅，否则会浪费额外的资源。 不过如果调用是在一个reactive上下文中的话，系统最终会在合适的时候自动调用`this.stop()`。

### 在哪写订阅

写订阅最好的地方是和使用数据最近的地方，这样的程序更容易理解，这意味着在实际工作中最好把订阅写作 *组件* 里。 在Blaze里，可以放在模板的 `onCreated()` 回调里：

```js
Template.Lists_show_page.onCreated(function() {
  this.getListId = () => FlowRouter.getParam('_id');

  this.autorun(() => {
    this.subscribe('Todos.inList', this.getListId());
  });
});
```

上述代码里面有2个重要的技术点：
1. 调用 `this.subscribe()` (而不是 `Meteor.subscribe`), 然后会给模板示例附加一个特殊的 `subscriptionsReady()` 函数，当所有订阅完成时，这个函数返回true。
2. 调用 `this.autorun` 将设置一个reactive的上下文，每当`this.getListId()` 变化时会重新初始化订阅。

### 获取数据

订阅数据会将数据放置到客户端的数据集里，需要在界面里面使用这些数据时，需要查询客户端的数据集。这里有一些相关的规则如下：

**本地也需要使用查询条件**

因为在订阅的过程中会添加查询条件，往往就容易在本地使用数据时忘记添加查询条件，但是这种用法可能会有一些问题。 考虑如下情况
1. 还有一个其他的订阅，也订阅了同样的数据集，结果在本地会放置在同一个本地数据库里，查询条件以及对应的数据可能不一样。
2. 当订阅发生变化的时候，有一个数据加载的阶段，数据是发生变化的。

**在订阅数据附近获取他**

这样有利于从代码的角度理解数据的来源。另外，一个比较常见的编程模式是在父组件里面获取数据，然后再传递给他的纯子组件。

### 全局订阅

有些数据是你在任何场合都需要使用的，这种数据可能适合全局订阅。但是即使是这种情况，也可以构造一个布局组件，然后在这个组件里面进行订阅。

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

# Method


## 什么是Method

Method是一种客户端远程调用(remote procedure call)服务端接口的技术手段。

从本质来说，Method是服务器的API端点(endpoint)，可以在服务端定义一个Method以及他对应的客户端的部分，然后使用一些数据调用它，写入数据库，最终通过回调函数返回数据。
Method紧密地和数据加载系统整合，并支持OptimisticUI。


## 定义和调用Method

### 基本Method

**定义**

可以通过内置的Meteor.methods API来定义Method，而Method总是可以在客户端及服务端加载以支持OptimisticUI。

下面这个例子使用`aldeed:simple-schema`包来做Method的参数校验

```js
Meteor.methods({
  'Todos.methods.updateText'({ todoId, newText }) {
    new SimpleSchema({
      todoId: { type: String },
      newText: { type: String }
    }).validate({ todoId, newText });

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

**调用**

可以在客户端及服务端使用`Meteor.call`来调用Method，需要注意的是，我们仅是需要定义一个从客户端调用服务端的方法的时候，才会使用Method，如果仅仅是在服务端调用服务端的功能，使用常规的JavaScript函数就行了，而不是一个Method。

下面这个例子展示了如何在客户端调用Method

```js
Meteor.call('Todos.methods.updateText', {
  todoId: '12345',
  newText: 'This is a todo item.'
}, (err, res) => {
  if (err) {
    alert(err);
  } else {
    // success!
  }
});
```

如果调用出错，Method会通过回调函数的第一个参数抛出一个error，如果调用成功，Method会通过回调函数的第二个参数返回结果。

### 高级Method

通过多年的改进，Method具备一些不是很直观的高端特性，完整地使用这些特性需要比较合适的boilerplate。我们首先介绍针对每个特性需要的编码，然后演示一个包装这些特性的package。

一个理想的Method需要具备如下特点：

1. 合理的验证代码
2. 容易测试
3. 容易使用一个定制的userid调用
4. 通过JS Module而不是字符串调用Method
5. 通过返回值来获得插入文档的id
6. 如果客户端验证失败，则不用调用服务器端Method，以免浪费服务器计算资源。

**定义**


```js
// Define a namespace for Methods related to the Todos collection
// Allows overriding for tests by replacing the implementation (2)
Todos.methods = {};

Todos.methods.updateText = {
  name: 'Todos.methods.updateText',

  // 独立验证代码，从而它可以独立运行 (1)
  validate(args) {
    new SimpleSchema({
      todoId: { type: String },
      newText: { type: String }
    }).validate(args)
  },

  // 独立Method嗲吗，从而它可以独立被调用 (3)
  run({ todoId, newText }) {
    const todo = Todos.findOne(todoId);

    if (!todo.editableBy(this.userId)) {
      throw new Meteor.Error('Todos.methods.updateText.unauthorized',
        'Cannot edit todos in a private list that is not yours');
    }

    Todos.update(todoId, {
      $set: { text: newText }
    });
  },

  // Call Method by referencing the JS object (4)
  // Also, this lets us specify Meteor.apply options once in
  // the Method implementation, rather than requiring the caller
  // to specify it at the call site.
  call(args, callback) {
    const options = {
      returnStubValue: true,     // (5)
      throwStubExceptions: true  // (6)
    }

    Meteor.apply(this.name, [args], options, callback);
  }
};

// 在Meteor的DDP系统里面注册方法
Meteor.methods({
  [Todos.methods.updateText.name]: function (args) {
    Todos.methods.updateText.validate.call(this, args);
    Todos.methods.updateText.run.call(this, args);
  }
})
```

**调用**

调用Method和调用一个普通的饿JavaScript函数一样方便

```js
// Call the Method
Todos.methods.updateText.call({
  todoId: '12345',
  newText: 'This is a todo item.'
}, (err, res) => {
  if (err) {
    alert(err);
  } else {
    // success!
  }
});

// Call the validation only
Todos.methods.updateText.validate({ wrong: 'args'});

// Call the Method with custom userId in a test
Todos.methods.updateText.run.call({ userId: 'abcd' }, {
  todoId: '12345',
  newText: 'This is a todo item.'
});
```

上述代码可以产生一个更好的开发流程，可以更好地处理Method的不同部分以及更加简单的测试。但是这个方法会写非常多的代码。

### 使用mdg:validated-method实现高级Method

`mdg:validated-method`这个包封装了上述流程的很多部分，从而方面定义一个具备丰富特性的Method。

上一个例子的具体代码简写如下，可以用同样的方法进行调用，但是Method的定义简单了很多，而且表意更加明确。

```js
Todos.methods.updateText = new ValidatedMethod({
  name: 'Todos.methods.updateText',
  validate: new SimpleSchema({
    todoId: { type: String },
    newText: { type: String }
  }).validator(),
  run({ todoId, newText }) {
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

## 错误处理

正常JavaScript函数里的错误会抛出一个Error对象，在Meteor Method里面也类似，不同的是这个error对象会通过websocket返回到client端。

### 抛出错误

Meteor引入了2种错误：Meteor.Error和ValidationError，这些Error和普通的JavaScript Error会在不同的条件下使用：

**一般错误**

如果错误仅是服务器内部的，不需要返回给客户端时，抛出一个普通的JavaScript错误对象即可。 这种情况下客户端会获得一个没有任何细节的不透明的服务端错误。

**Meteor.Error**

当服务器无法完成客户端的请求，在需要抛出一个带描述的`Meteor.Error`对象给客户端，它具备3个参数：error，reason和details。
1. **error** 是一个简短唯一的机器可读客户端课理解的错误编码字符串。最好将Method名称设置成前缀。
2. **reason** 一个简短的针对开发者的描述。需要具备足够的能够debug这个错误的信息。这个信息不应该直接打印到客户端。
3. **detail** 是可选的， 提供一些额外的信息帮助客户端理解错误内容。 一般，这个信息可以和error信息组合起来打印出有用的报错信息给终端用户。

**ValidationError**

当参数验证错误时，会抛出一个ValidationError。如果使用了`mdg:validated-method`的`aldeed:simple-schema`组合，这个错误是会自动抛出的。
具体内容可以参考`mdg:validation-error`的[文档](https://atmospherejs.com/mdg/validation-error)。

### 处理错误

调用Method的时候，任何它抛出的错误都会在回调函数里返回，这时需要判断是哪种异常，并显示给用户合适的信息。

```js
Todos.methods.updateText.call({
  todoId: '12345',
  newText: 'This is a todo item.'
}, (err, res) => {
  if (err) {
    if (err.error === 'Todos.methods.updateText.unauthorized') {
      // Displaying an alert is probably not what you would do in
      // a real app; you should have some nice UI to display this
      // error, and probably use an i18n library to generate the
      // message from the error code.
      alert('You aren\'t allowed to edit this todo item');
    } else {
      // Unexpected error, handle it in the UI somehow
    }
  } else {
    // success!
  }
});
```

## Method加载数据

因为Method可以被用作通用的RPC，所以它也可以代替发布来获取数据，他们各自有自己的优缺点。

Method可以用来获取可能是经过复杂计算的不跟随服务器数据变化的数据。不过这种方法的缺点在于，这种数据不会被自动放到minimongo里面，这样则需要手工管理这些数据。可以使用本地数据集来管理和显示从Method中而不是通过发布订阅获取的数据。

首先，需要创建一个本地数据集(local collection)，这样的数据集仅存在于客户端而不是服务端。

```js
// In client-side code, declare a local collection
// by passing `null` as the argument
ScoreAverages = new Mongo.Collection(null);
```

然后，如果使用Method获取数据，则可以放到这个数据集里

```js
function updateAverages() {
  // Clean out result cache
  ScoreAverages.remove({});

  // Call a Method that does an expensive computation
  Games.methods.calculateAverages.call((err, res) => {
    res.forEach((item) => {
      ScoreAverages.insert(item);
    });
  });
}
```

然后就可以在UI组件里面和访问普通MongoDB数据集一样访问了。 这样的数据不会自动更新，每当我们需要新数据时，必须手工调用updateAverages。


## 调用Method的生命周期

1. 客户端模拟运行Method
2. 发送一个method DDP消息到服务器
3. 服务端运行Method
4. 返回值发送给客户端
5. 任何被Method影响的DDP发布都会更新
6. updated消息发给客户端，采用服务端结果更新数据，回调执行

## Method对比REST

* Method使用同步调用，但是并没有阻塞
* Method总是按照顺序运行















-

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

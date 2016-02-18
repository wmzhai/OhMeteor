# Blaze模板

Blaze是Meteor内嵌的实时渲染库，通过Spacebar来写模板，最终编译成JavaScript的UI组件进行渲染。

## Spacebars

Spacebars是基于HTML的模板语言，用来渲染实时变化的上下文数据，它在HTML基础上扩展了mustache tag`{{ }}`。

```html
<template name="Todos_item">
  <div class="list-item {{checkedClass todo}} {{editingClass editing}}">
    <label class="checkbox">
      <input type="checkbox" checked={{todo.checked}} name="checked">
      <span class="checkbox-custom"></span>
    </label>

    <input type="text" value="{{todo.text}}" placeholder="Task name">
    <a class="js-delete-item delete-item" href="#">
      <span class="icon-trash"></span>
    </a>
  </div>
</template>
```

这个模板需要一个键值为`todo`的对象作为数据上下文(data context)，然后通过mustache tag来访问其属性，比如`{{todo.text}}`。


另外，这里有一个带参数的模板helper `{{checkedClass todo}}`，这个helper定义在一个独立的JavaScript文件里：

```js
Template.Todos_item.helpers({
  checkedClass(todo) {
    return todo.checked && 'checked';
  }
});
```

在这个Helper里面，`this`是helper调用时的当前数据上下文。 因为这个上下文不是很清晰，所以可以将需要的数据作为参数传递到helper里面。

## Blaze模板最佳实践

### 数据上下文验证

为了确保模板能够获得正确的数据，需要对数据上下文进行验证。 通常可以在Blaze的onCreated函数里面创建，如下

```js
Template.Lists_show.onCreated(function() {
  this.autorun(() => {
    new SimpleSchema({
      list: {type: Function},
      todosReady: {type: Boolean},
      todos: {type: Mongo.Cursor}
    }).validate(Template.currentData());
  });
});
```

任何时候，只要数据发生变化，`autorun()`就会自动执行以重新验证数据。

### 子模板参数给数据上下文命名

最好能够给子模板参数一个固定的名字，主要原因有两个：

1. 当在子模板里面使用数据时，`{{todo.title}}`比`{{title}}`更加清晰
2. 当未来需要更多参数时更加灵活

作为一个良好的习惯，做好能够给子模块提供一个数据上下文，而不是直接让他集成父模板。

### 最好用`{{#each .. in}}`

为了清晰起见，最好使用`{{#each todo in todos}}`而不是早期的`{{#each todos}}`相关写法。后一种写法将整个数据上下文设置到todo一个对象里面，导致访问代码块以我的内容比较麻烦。

### 为helper传递数据

在helper里面，相对于通过this访问数据而已，最好是将参数直接从模板传递过去。

### 使用template实例

可以使用template来保存内部功能和状态， 在模板的回调函数里面，可以通过`this`来访问模板实例，而在事件处理函数和helper里面可以通过`Template.instance()`来访问。在事件处理函数里，这个变量往往也被作为第二个参数传入。

作为一个最佳实践，我们鼓励在helper中使用`instance`这个变量，并在每个相关helper的最开始处加以赋值和声明。比如：


```js
Template.Lists_show.helpers({
  todoArgs(todo) {
    const instance = Template.instance();
    return {
      todo,
      editing: instance.state.equals('editingTodo', todo._id),
      onEditingChange(editing) {
        instance.state.set('editingTodo', editing ? todo._id : false);
      }
    };
  }
});

Template.Lists_show.events({
  'click .js-cancel'(event, instance) {
    instance.state.set('editing', false);
  }
});
```

### 为状态使用reactive dict

`reactive-dict`包方便定义一个简单的`key-value`字典。可以在`onCreated`回调里面创建 state字典，并将他赋值给模板实例。

```js
Template.Lists_show.onCreated(function() {
  this.state = new ReactiveDict();
  this.state.setDefault({
    editing: false,
    editingTodo: false
  });
});
```

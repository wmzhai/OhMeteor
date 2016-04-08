# 代码规范

读完本章后，您将了解

1. 为什么需要一致的代码规范
2. 我们针对JavaScript代码的规范是什么
3. 如何设置ESLint进行代码自动检测
4. Meteor相关的代码规范及模式

## 为什么要规范代码

程序员在代码规范上的争论花费了很多时间，具体例如使用单引号还是双引号，以及大括号的放置位置，以及缩进的空格数等。而这些问题跟代码的质量是有着密切的关系的，在一个组织内统一代码规范是有着巨大的好处的。

### 容易阅读代码

一般情况下并不是逐个单词地阅读代码，大多数情况下，只需要看一下代码的形状以及高亮，就大致了解代码的功能了。如果代码的风格是一致的，读起来就会容易很多。

```js
// 这个代码有很大的误导性，因为两个语句看起来像是在一个条件里面
if (condition)
  firstStatement();
  secondStatement();
```

```js
// 这个就比较清楚了
if (condition) {
  firstStatement();
}

secondStatement();
```

### 自动错误检查

风格统一的代码更加容易用工具进行错误检查。

### 更深入的理解


一般很难一次学习一种编程语言的所有内容，使用社区推荐的代码规范，能够促使比较高效地避免陷阱。也意味着，你不需要学习所有的语法细节，就可以直接开始工作了，随着时间的退役可以进一步深入的了解。



## JavaScript规范指南

在Meteor社区里，我们坚信JavaScript是构建web应用最佳的语言。 而JavaScript本身也在不断的改进，ES6标准统一了JavaScript社区，下面是我们关于如何在你的应用里使用ES6的建议。

### 使用`ecmascript`包

ECMAScript作为跨浏览器的JavaScript标准，已经达到了每年更新一次的程度，最新的标准是ES6，这个标准对语言做出了很大的改进。Meteor的`ecmascript`包通过babel将ES6代码解释成所有浏览器都懂的传统JavaScript。 因为它与传统的JavaScript完全兼容，所以即使您不想使用任何新特性时也没有任何问题。另外，我们这个包也支持source map这样高级的功能，所以你可以使用任何传统的开发工具来调试代码，而不必查看编译输出。

默认情况下，所有新建的Meteor项目都会使用`ecmascript`包，并自动编译所有具有`.js`后缀的文件。具体可以参考[ecmascript包支持的ES6特性列表 ](https://docs.meteor.com/#/full/supportedES6features)。
为了得到完整的支持，您也应该使用`es5-shim`包，它也是一个默认添加的包。者意味着您可以使用诸如`Array#forEach`这样的运行时特性而不必考虑浏览器支持性。

进一步信息可以参考如下文章：

- [Getting started with ES6 and Meteor](http://info.meteor.com/blog/ES6-get-started)
- [Set up Sublime Text for ES6](http://info.meteor.com/blog/set-up-sublime-text-for-meteor-es6-ES6-and-jsx-syntax-and-linting)
- [How much does ES6 cost?](http://info.meteor.com/blog/how-much-does-ES6-cost)

### 遵循一个JavaScript规范指南

我们建议选择遵循一个JavaScript规范指南，并使用工具去强制这个指南。一个常见的选项是[Airbnb style guide](https://github.com/airbnb/javascript)的基础上对ES6和React的扩展。

## 使用ESLint检查代码

Lint是针对常见错误和样式问题对代码自动检查代码的过程。比如，ESLint可以检查出变量的拼写错误，以及是否有一些无效代码的存在。我们建议使用[Airbnb eslint configuration](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb)来遵循Airbnb的规范。

下面是在不同开发阶段设置自动Lint的指南，一般来说，需要尽可能多地运行lint，因为他是查找微小错误最快捷方便的手段。

### 安装运行ESLint

在应用程序中设置ESLint，可以安装如下npm包：

```
npm install --save-dev eslint eslint-plugin-react eslint-plugin-meteor eslint-config-airbnb
```

您可以在`package.json`里加入一个`eslintConfig`小节，以指定采用Airbnb配置，并使用[ESLint-plugin-Meteor](https://github.com/dferber90/eslint-plugin-meteor).
您也可以使用任何您需要改变的规则，并加入一个lint的npm指令:

```
{
  ...
  "scripts": {
    "lint": "eslint .",
    "pretest": "npm run lint --silent"
  },
  "eslintConfig": {
    "plugins": [
      "meteor"
    ],
    "extends": [
      "airbnb/base",
      "plugin:meteor/guide"
    ],
    "rules": {}
  }
}
```

使用`"airbnb/base"`作为基本配置，并在一个React项目里采用`"airbnb"`，执行lint时只需要执行

```bash
npm run lint
```

更详细信息可以参考ESLint主页上的[Getting Started](http://eslint.org/docs/user-guide/getting-started)。

### WebStorm集成

Lint是最快的发现代码潜在bug的方法，运行linter往往比运行程序和单元测试更快，所以最好是能够随时运行它。在IDE里面设置Lint一开始看起来总是报错，比较扰人，不过长期来看，这个方法有助于你养成良好的代码习惯。在WebStorm中设置ESLint可以参考[these instructions for using ESLint](https://www.jetbrains.com/webstorm/help/eslint.html)。 当你安装完ESLint包并设置好`package.json`以后, 点击Apply以enable ESLint功能。


## Meteor代码规范

这一节讨论Meteor相关的代码规范，也可以将部分规范应用到其他JavaScript程序里。


### 集合

集合(Collections)应该使用PascalCase的复数名词，而和集合对应的数据库集合名词，应该与它名词完全一致。比如：

```js
// Defining a collection
Lists = new Mongo.Collection('Lists');
```

数据库里面的字段名称应该采用camelCased，与JavaScript里的变量名一致。

```js
// Inserting a document with camelCased field names
Widgets.insert({
  myFieldName: 'Hello, world!',
  otherFieldName: 'Goodbye.'
});
```

### 方法和发布

方法和发布名称应该是camelCase的，并且用它们的module作为namespace

```js
// in imports/api/todos/methods.js
updateText = new ValidatedMethod({
  name: 'todos.updateText',
  // ...
});
```

注意上面代码使用[ValidatedMethod package recommended in the Methods article](methods.html#validated-method)，如果你不使用那个package，也可以使用传递给`Meteor.methods`一样的名字。

下面是命名习惯如何应用到发布上的:

```js
// Naming a publication
Meteor.publish('lists.public', function listsPublic() {
  // ...
});
```

### 文件，导出和包

应该使用ES6的`import`和`export`特性来管理代码，这样能够更深入地了解代码不同部分的依赖。

代码中每个文件应该代表一个单独的逻辑模块，每个class，UI组件和collection，独立一个文件是一个比较好的习惯，这样能够避免单个模块里面有多个不相关的东西。不过有时也会有一些例外，比如你需要一个不在外部使用的小组件的时候。

当文件仅有单个类型和UI组件时，文件名应该和它定义的东西一致。比如导出一个class的文件

```js
export default class ClickCounter { ... }
```

这个类应该在一个叫做`click_counter.js`的文件里面定义，导入代码如下：

```js
import ClickCounter from './click_counter.js';
```

注意，导入时使用相对路径的，需要把文件后缀放进去。

在比较早的版本里， `api.export`允许导出多个内容，这样很多可以看到一些非默认导出，比如

```js
// You'll need to deconstruct here, as Meteor could export more symbols
import { Meteor } from 'meteor/meteor';

// This will not work
import Meteor from 'meteor/meteor';
```

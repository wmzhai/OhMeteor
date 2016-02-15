# 开发环境

本章介绍基本的开发环境设置。

## 前提

* **会翻墙** 首先要会翻墙，推荐shadowsocks和proxychains4，后者主要是解决命令行翻墙的。
* **会英语** 最新的技术文档以及技术社区的通用文字都是英文，需要能够熟练阅读英文技术资料。
* **会搜索** 尽量用Google，别用百度。

## 操作系统

* **MacOS** 首推MacOS作为Meteor开发的操作系统，各种资源都比较全，命令行支持也和linux系统一样好，而且这也是唯一支持iOS开发的操作系统。
* **Ubuntu** 如果没有MacOS，可以使用Ubuntu LTS来进行开发，具体的设置过程可以参考[Meteor环境安装指南](https://github.com/wmzhai/setup-meteor-machine/blob/master/README.md)。
* **Windows** 虽然Meteor号称支持Windows开发，但是一定不要去做尝试，特别是新手，你会遇到很多问题，而且很少有人愿意去帮你解决。如果你只有Windows电脑，那么就用Ubuntu的虚拟机吧。

## 集成开发环境

* **Webstorm** 推荐使用Webstorm作为Meteor的开发环境，比较傻瓜，不用任何配置就有了丰富的面向Meteor开发的功能，包括调试、git和Emmet等。
* **其他** 对Sublime和Atom比较熟悉的可以采用这些编辑器，不过很多功能需要自己安装插件，相对而言Atom的未来可能会更加好一些。

## 浏览器

Chrome是唯一的选择，不管是Mac还是Ubuntu。

## 安装Meteor

Meteor的安装比较简单，只需要一条指令

```
curl https://install.meteor.com/ | sh
```

安装完成以后，尝试创建一个例子并运行

```
meteor create hello
cd hello
meteor
```

等项目启动以后 ，用Chrome访问 http://localhost:3000即可。

## 断点调试

Webstorm和Chrome的整合支持meteor的断点调试。

## 简单部署

对于一个Meteor项目，在项目路径下执行

```
meteor deploy my_app_name.meteor.com
```

就会在meteor的服务器上做一个简单的部署，可以用如下网址访问：
http://my_app_name.meteor.com

## 版本管理

推荐用git作为版本管理工具，共有项目可以托管在[Github](https://github.com)上。私有项目，可以购买Github的私仓。如果将项目托管在自己的服务器上，可以安装[Gitlab](https://about.gitlab.com/downloads/)。

## 练习

安装好上述开发环境，然后按照[入门教程](https://www.meteor.com/tutorials/blaze/creating-an-app)操作一遍。

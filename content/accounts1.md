# 账户系统

Meteor内置了一个可以直接使用的账户系统及其界面，并基于它实现了多个第三方账号的集成。

账户系统的核心内容在**accounts-base**包里面实现，一般的应用程序通过添加**accounts-password**、**accounts-weibo** 等来自动引用**accounts-base**。

## 基本使用

### userId

用户登录以后可以一直获得 this.userId 这个变量，并通过它来获取登录状态，这个变量在Method，Publication以及client端代码里面都可以调用。

### accounts-base

**accounts-base** 包是Meteor账号系统的核心，它包括

1. **user collection** 这个数据集及其对应的schema，这个数据集可以通过


## accounts-ui



## useraccounts

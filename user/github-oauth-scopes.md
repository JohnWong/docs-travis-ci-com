---
title: Travis CI的GitHub API使用范围
layout: en
permalink: /user/github-oauth-scopes/
---
当你登录到Travis CI，我们请求一些权限来访问你的数据。

本页提供了我们请求以及原因的概述。

### 开源项目使用Travis CI

我们目前在<https://travis-ci.org>请求如下权限。

注意，对于开源项目，我们并没有任何对你的源代码或者资料的写权限。

确保检查[GitHub API的文档](/user/github-oauth-scopes/)来了解我们使用的范围的额外详情。

* `user:email`

    我们同步你的电子邮件地址用来给你发送构建提醒。目前并不会有任何其他用途。

    我们请求这一权限是因为没有它，我们可能没有办法发送构建提醒给你。你的电子邮件地址可能从GitHub资料中隐藏，这样会对我们隐藏。

* `read:org`

    当你登录了Travis CI时，我们展示了你所有的库，包括那些你所在的组织里的库。

    没有这个范围，GitHub API将会隐藏所有你是私有成员的组织。因此要确保我们展示你所有的库给你，我们需要这个范围。

    注意这个范围允许访问私有和公共库的基本信息，但是并不访问任何存储在其中的数据与代码。

* `repo_deployment`

    给我们访问[即将到来的部署API](http://developer.github.com/v3/repos/deployments/)，目前使用预览模式。

    这个范围目前使用并不被活跃，但是将来将会。

* `repo:status`

    在每次构建后，我们更新其提交的状态，这与测试pull request最相关。这一范围给我们在构建开始与结束时更新提交状态的权限。

* `write:repo_hook`

    在Travis CI上构建一个新的库非常容易，在你的资料中启用它并推送一个新提交。

    我们需要这个API范围，从而可以在GitHub新提交或者pull request的时候能够通知更新我们需要的webhook。此外你的账号需要有你希望启用的库的管理权限。

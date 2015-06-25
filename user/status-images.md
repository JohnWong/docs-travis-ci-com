---
title: 嵌入状态图片
layout: en
permalink: /user/status-images/
---

你可以嵌入状态图片（也被称作是徽章或者图标）到你的README或者网站来展示你的构建的状态。

例如这个徽章展示了`travis-web`库的构建状态：
[![Build Status](https://travis-ci.org/travis-ci/travis-web.svg?branch=master)](https://travis-ci.org/travis-ci/travis-web)

状态图片的URL展示在你的Travis CI库的页面：

1. 点击右上角的状态图片来打开一个对话框，里面包含了状态图的markdown、html等通用模版。

	![](http://s3itch.paperplanes.de/statusimage_20140320_112129.jpg)

2. 在对话框中个选择分支与模版。

3. 将文本复制粘贴到你的README或者网站website。

公共库的构建状态图在Travis CI中公开可用。

[私有库](https://travis-ci.com)的构建状态图包括了一个安全token。

![](http://s3itch.paperplanes.de/Travis_CI__Hosted_Continuous_Integration_That_Just_Works_20140320_112255_20140320_112334.jpg)

这个token只用来访问构建状态图，但是我们推荐你不要再一个公开可用的网站使用。

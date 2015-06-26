---
title: Atom构建Feeds
layout: en
permalink: /user/build-feeds/
---

得到构建更新的一种方式是Atom feeds。

你可以在你喜爱的RSS阅读器中阅读它们，或者用脚本以编程方式利用它们。

### Atom Feeds

Travis CI上的每个库都有其自己的Atom feed，包含了运行在其中的所有构建，包括pull request与普通提交。

feed直接从你的[API](https://api.travis-ci.org)获取。获取一个库的构建的典型URL是`https://api.travis-ci.org/repos/travis-ci/travis-ci/builds`。这默认返回用JSON表示，但是你可以通过添加`.atom`后缀来得到Atom feed。

对于上面的库，URL将会是`https://api.travis-ci.org/repos/travis-ci/travis-ci/builds.atom`

在Travis CI Pro中，对于私有库，你需要一个token来订阅源。The [API endpoint](https://api.travis-ci.com)也不同。

token必须作为`token`参数附加到URL后。你可以在"Profile" tab下[你的资料](https://magnum.travis-ci.com/profile/)中找到token。

![]({{ "/images/token.jpg" | prepend: site.baseurl }})

一个URL示例是
`https://api.travis-ci.com/repos/travis-ci/billing/builds.atom?token=<token>`

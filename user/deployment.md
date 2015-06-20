---
title: 部署
layout: en
permalink: /user/deployment/
---

### 支持的提供者

到下列提供者的持续集成当前即开即用：

{% include deployments.html %}

### 部署到多个提供者

部署到多个提供者可以通过作为列表添加不同的provider到`deploy`部分来使用。例如如果你希望部署到cloudControl与Heroku，你的`deploy`部分看起来会像这样：

    deploy:
      - provider: cloudcontrol
        email: "YOUR CLOUDCONTROL EMAIL"
        password: "YOUR CLOUDCONTROL PASSWORD"
        deployment: "APP_NAME/DEP_NAME"
      - provider: heroku
        api_key "YOUR HEROKU API KEY"

### 使用`on:`进行条件发布

可以通过为每个部署提供者设置`on:`来控制部署。.

    deploy:
      provider: s3
      access_key_id: "YOUR AWS ACCESS KEY"
      secret_access_key: "YOUR AWS SECRET KEY"
      bucket: "S3 Bucket"
      skip_cleanup: true
      on:
        branch: release
        condition: $MY_ENV = super_awesome

当`on:`部分指定的所有条件都满足时，到这个提供者的部署将会执行。

通用选项是：

1. 库的**`repo`**名称与所有者（例如`travis-ci/dpl`）。

1. 分支的**`branch`**名称。如果省略了，默认是特定的`app`分支或者`master`。如果分支名称不能提前知道，你可以指定`all_branches: true`而不是`branch: **`，并使用其他条件来控制你的部署。

1. **`jdk`**，**`node`**，**`perl`**，**`php`**，**`python`**，**`ruby`**，**`scala`**，**`go`**：对于支持多个版本的语言运行时，你可以限制部署，使其只发生在匹配预期版本的工作中。

1. **`condition`**：你可能用这个选项设置了bash条件。它必须是一个字符串值，将会被放入`if [[ <condition> ]]; then <deploy>; fi`这样形式的bash表达式里。它可以是复杂的，也可以只有一个。比如`$CC = gcc`。

1. **`tags`**：设置为`true`时，应用会在一个tag设置到提交上时部署。这会引起`branch`条件被忽略。

#### 使用`on:`的条件发布的例子：

这个例子在0.11版本的Node.js运行测试时，将`staging`分支部署到Nodejistu。

    deploy:
      provider: nodejitsu
      user: ...
      api_key: ...
      on:
        branch: staging
        node: '0.11' # this should be quoted; otherwise, 0.10 would not work

下一个例子只在`$CC`设置为`gcc`时部署到S3。

    deploy:
      provider: s3
      access_key_id: "YOUR AWS ACCESS KEY"
      secret_access_key: "YOUR AWS SECRET KEY"
      skip_cleanup: true
      bucket: "S3 Bucket"
      on:
        condition: "$CC = gcc"

这个例子当设置了tag并且Ruby版本是2.0.0时部署到Github Release。

    deploy:
      provider: releases
      api-key: "GITHUB OAUTH TOKEN"
      file: "FILE TO UPLOAD"
      skip_cleanup: true
      on:
        tags: true
        rvm: 2.0.0

### 其他提供者

我们致力于支持其他Paas提供者。如果你的应用的提供者并不在列表内，你希望Travis CI自动部署你的应用，请[联系我们](mailto:support@travis-ci.com)。

如果你希望贡献或者试验[部署工具](https://github.com/travis-ci/dpl)，确保你使用GitHub的edge版本：

    deploy:
      provider: awesome-experimental-provider
      edge: true

### Pull Requests

注意pull request构建会完全忽略部署步骤。

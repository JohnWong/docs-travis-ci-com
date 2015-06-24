---
title: Validating .travis.yml files
layout: en
permalink: /user/travis-lint/
---

在提交前验证你的`.travis.yml`文件来减少常见构建错误，例如

* `.travis.yml`中的无效[YAML](http://yaml-online-parser.appspot.com/)
* 缺失的`language`键
* Ruby、PHP、OTP等的不支持的[运行时版本](/user/ci-environment/)
* 弃用的特性或运行时别名

### 使用lint.travis-ci.org

你可以通过进入一个到你的库的链接或者将你的`.travis.yml`的内容粘贴到表格中来使用[web app](http://lint.travis-ci.org)。

### 使用travis-lint命令行工具

要安装`travis-lint`命令行工具，需要Ruby 1.8.7+与RubyGems：

    gem install travis-lint

要运行`travis-lint`：

    # from any directory
    travis-lint [path to your .travis.yml]

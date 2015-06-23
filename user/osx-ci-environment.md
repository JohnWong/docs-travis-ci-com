---
title: OS X构建环境
layout: en
permalink: /user/osx-ci-environment/
---

### 本指南覆盖什么内容

本指南解释了在Travis OS X CI环境（通常引用为“持续集成环境”）中可用的包、工具与设置。

## 概览

Travis CI在每次构建前建立虚拟机快照，并在运行构建结束后回滚。这提供了一些好处：

* 托管操作系统不会受测试套件影响
* 在两次运行之间没有状态保留
* 没有密码的sudo可用
* 测试套件可以创建数据库，添加RabbitMQ vhosts或用户等。

测试套件可用的环境被称为*Travis CI
环境*。

## CI环境操作系统

Travis CI使用OS X 10.9.5。

## 所有虚拟机映像的通用环境

### Homebrew

Homebrew已经安装，并会在虚拟机每次更新的时候更新。推荐在使用Homebrew安装任何东西之前先运行`brew update`。

#### 升级包注意事项

当使用`brew upgrade`升级一个包的时候，如果已经安装了最新版本的包，那么命令将会失败（因此升级并未发生）。

根据你升级包的方式，可能会引起构建错误：

```
$ brew upgrade xctool
Error: xctool-0.1.16 already installed
The command "brew upgrade xctool" failed and exited with 1 during .

Your build has been stopped.
```

或者可能引起命令未找到：

```
xctool: command not found
```

这是Homebrew那边故意的行为，但是你可以通过先运行`brew outdated`命令检查需要升级的包来避免：

    before_install:
      - brew update
      - brew outdated <package-name> || brew upgrade <package-name>

例如，如果你总是想要最新版本的xctool，你可以运行：

    before_install:
      - brew update
      - brew outdated xctool || brew upgrade xctool


### 编译与构建工具链

* apple-gcc42
* autoconf 2.69
* automake 1.14
* maven 3.2
* mercurial 2.9
* pkg-config 0.28
* subversion 1.8.10
* wget 1.15
* xctool 0.2.1
* cmake

### 语言

* go 1.3.1
* node 0.10.32

### 服务

* postgis 2.1.3
* postgresql 9.3.5

### Xcode

安装了Xcode 6.1以及iOS 7.0，7.1与8.1的模拟器与SDK。命令行工具也安装了。

### 运行时

每个worker 都有至少一个版本的Ruby，Java与Python来适应在构建时需要这些运行时中的某个的项目。

### 环境变量

* `CI=true`
* `TRAVIS=true`
* `USER=travis` (**不要依赖这个值**)
* `HOME=/Users/travis` (**不要依赖这个值**)

此外，Travis CI设置了你在构建时可以使用的环境变量，例如用来给构建打标签，或者运行构建后部署。

* `TRAVIS_BRANCH`：对于非pull request触发的构建，这是当前构建使用的分支的名称；而对于pull request触发的构建，这是pull request的目标分支的名称（在许多情况下将会是`master`）。
* `TRAVIS_BUILD_DIR`：正在构建的库拷贝到worker上的绝对路径。
* `TRAVIS_BUILD_ID`：Travis CI内部使用的当前构建的id。
* `TRAVIS_BUILD_NUMBER`：当前构建的号码（例如“4”）。
* `TRAVIS_COMMIT`：当前构建正在测试的提交。
* `TRAVIS_COMMIT_RANGE`：在推送或者pull request中包含的提交范围。
* `TRAVIS_JOB_ID`：Travis CI内部使用的当前工作的id。
* `TRAVIS_JOB_NUMBER`：当前工作的号码（例如“4.1”）。
* `TRAVIS_PULL_REQUEST`：如果当前工作是一个pull request，是pull request的数字；否则是“false”。
* `TRAVIS_SECURE_ENV_VARS`：安全环境变量是否在使用。值是“true”或“false”。
* `TRAVIS_REPO_SLUG`：当前正在构建的库的slug（以`owner_name/repo_name`的形式）。（例如“travis-ci/travis-build”）。
* `TRAVIS_OS_NAME`：在多操作系统构建中，这个值指示工作正在运行的平台。值当前是`linux`与`osx`，未来可能会扩展。
* `TRAVIS_TAG`：如果当前构建是在一个标签上，这包含了标签的名称。

### Maven版本

提供Apache Maven 3。

### Ruby版本/实现

* 系统（2.0.0）————你需要使用`sudo`在Ruby下安装gem
* 1.9.3
* 2.0.0（默认）
* 2.1.2
* 2.1.3

为每个用户预装了使用[RVM](http://rvm.io/)构建的Ruby。

### Bundler版本

目前是1.7版本（通常是最新的）

### 全局gem集合中的gem

* bundler
* rake
* cocoapods

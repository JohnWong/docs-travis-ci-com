---
title: 准备开始
layout: en
permalink: /user/getting-started/
---

在你的Github上托管的代码库上使用Travis CI的简明教程。对这些完全陌生？试试[新手使用Travis CI](/user/for-beginners) instead. 

Or if you want a more complete guide to a particular language, pick one of these: 

{% include languages.html %}

<!-- 
[C](/user/languages/c) | [C++](/user/languages/cpp) | [Clojure](/user/languages/clojure) | [C#](/user/languages/csharp/) | [D](/user/languages/d) | [Erlang](/user/languages/erlang) | [F#](/user/languages/csharp/) | [Go](/user/languages/go) | [Groovy](/user/languages/groovy) | [Haskell](/user/languages/haskell) | [Java](/user/languages/java) | [JavaScript (with Node.js)](/user/languages/javascript-with-nodejs) | [Julia](/user/languages/julia) | [Objective-C](/user/languages/objective-c) | [Perl](/user/languages/perl) | [PHP](/user/languages/php) | [Python](/user/languages/python) | [Ruby](/user/languages/ruby) | [Rust](/user/languages/rust) | [Scala](/user/languages/scala) | [Visual Basic](/user/languages/csharp/).
-->

### 开始使用Travis CI:

1. 用你的GitHub账号[登陆Travis CI](https://travis-ci.org/auth)，允许Github的[访问权限确认](/user/github-oauth-scopes).

2. 一旦你登录了，我们将从Github同步你的代码库，到你的[资料页](https://travis-ci.org/profile)开启你想要构建Travis CI的代码库的。

	> 注意：你只能开启拥有管理权限的代码库的Travis CI构建。

2. 添加一个 `.travis.yml` 文件到你的代码库来告诉Travis CI构建什么：

   ```yaml
   language: ruby
   rvm:
    - "1.8.7"
    - "1.9.2"
    - "1.9.3"
    - rbx
   # uncomment this line if your project needs to run something other than `rake`:
   # script: bundle exec rspec spec
   ```
   
   这个例子告诉Travis CI这是一个使用Ruby语言，使用 `rake` 构建的项目。Travis CI使用三个版本的Ruby和最新版的Rubinius。

2. 添加 `.travis.yml` 文件到git，提交并推送，来触发一次Travis CI构建：

	> 注意：Travis只基于你添加代码库到Travis后推送的第一个提交上运行一次构建。

2. 检查[构建状态](https://travis-ci.org/repositories)页来查看你的构建通过还是失败了。

你可能需要通过[安装依赖](/user/installing-dependencies)或者[建立一个数据库](/user/database-setup/)来[定制你的构建](/user/customizing-the-build)。或者你只是想要关于[测试环境](user/ci-environment/)的更多信息？

对于任何类型的问题，欢迎加入我们的IRC频道[#travis on chat.freenode.net](irc://chat.freenode.net/%23travis)。

<!--

### Some basic **.travis.yml** examples:


#### C

    language: c
    compiler:
      - gcc
      - clang
    # Change this to your needs
    script: ./configure && make

Learn more about [.travis.yml options for C projects](/user/languages/c/)


#### C++

    language: cpp
    compiler:
      - gcc
      - clang
    # Change this to your needs
    script: ./configure && make

Learn more about [.travis.yml options for C++ projects](/user/languages/cpp/)


#### Clojure

For projects using Leiningen 1:

    language: clojure
    jdk:
      - oraclejdk7
      - openjdk7
      - openjdk6

For projects using Leiningen 2:

    language: clojure
    lein: lein2
    jdk:
      - openjdk7
      - openjdk6


Learn more about [.travis.yml options for Clojure projects](/user/languages/clojure/)

#### C#, F#, and Visual Basic

    language: csharp
    solution: solution-name.sln

Learn more about [.travis.yml options for C# projects](/user/languages/csharp/)

#### Dart

    language: dart
    dart:
      - stable
      - dev
      - "1.8.0"

Learn more about [.travis.yml options for Dart projects](/user/languages/dart/)

#### Erlang

    language: erlang
    otp_release:
      - R15B02
      - R15B01
      - R14B04
      - R14B03

Learn more about [.travis.yml options for Erlang projects](/user/languages/erlang/)

#### Haskell

    language: haskell

Learn more about [.travis.yml options for Haskell projects](/user/languages/haskell/)


#### Go

    language: go

Learn more about [.travis.yml options for Go projects](/user/languages/go/)



#### Groovy

    language: groovy
    jdk:
      - oraclejdk7
      - openjdk7
      - openjdk6


Learn more about [.travis.yml options for Groovy projects](/user/languages/groovy/)

#### Haxe

    language: haxe
    haxe:
      - "3.2.0"
      - development

Learn more about [.travis.yml options for Haxe projects](/user/languages/haxe/)

#### Java

    language: java
    jdk:
      - oraclejdk8
      - oraclejdk7
      - openjdk7
      - openjdk6


Learn more about [.travis.yml options for Java projects](/user/languages/java/)

#### Julia

    language: julia
    julia:
      - release
      - nightly

Learn more about [.travis.yml options for Julia projects](/user/languages/julia/)

#### Node.js

     language: node_js
     node_js:
       - "0.10"
       - "0.8"
       - "0.6"

Learn more about [.travis.yml options for Node.js projects](/user/languages/javascript-with-nodejs/)

#### Objective-C

     language: objective-c

Learn more about [.travis.yml options for Objective-C projects](/user/languages/objective-c/)

#### Perl

    language: perl
    perl:
      - "5.16"
      - "5.14"
      - "5.12"

Learn more about [.travis.yml options for Perl projects](/user/languages/perl/)

#### PHP

    language: php
    php:
      - "5.5"
      - "5.4"
      - "5.3"

Learn more about [.travis.yml options for PHP projects](/user/languages/php/)

#### Python

    language: python
    python:
      - "3.3"
      - "2.7"
      - "2.6"
    # command to install dependencies, e.g. pip install -r requirements.txt --use-mirrors
    install: PLEASE CHANGE ME
    # command to run tests, e.g. python setup.py test
    script:  PLEASE CHANGE ME

Learn more about [.travis.yml options for Python projects](/user/languages/python/)

#### R

    language: r

Learn more about [.travis.yml options for R projects](/user/languages/r/)

#### Ruby

    language: ruby
    rvm:
      - "1.8.7"
      - "1.9.2"
      - "1.9.3"
      - jruby-18mode # JRuby in 1.8 mode
      - jruby-19mode # JRuby in 1.9 mode
      - rbx
    # uncomment this line if your project needs to run something other than `rake`:
    # script: bundle exec rspec spec

Learn more about [.travis.yml options for Ruby projects](/user/languages/ruby/)

#### Scala

     language: scala
     scala:
       - "2.9.2"
       - "2.8.2"
     jdk:
       - oraclejdk7
       - openjdk7
       - openjdk6


Learn more about [.travis.yml options for Scala projects](/user/languages/scala/)

-->

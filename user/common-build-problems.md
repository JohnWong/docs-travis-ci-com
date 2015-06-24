---
title: 常见构建问题
layout: en
permalink: /user/common-build-problems/
---

<div id="toc"></div>

## 我的测试被打断但是昨天还是好的

主要代码没有任何变更但是测试突然打断的一个非常常见的原因是上游依赖的变更。

这可能是一个Ubuntu包或者你的项目语言的任何依赖，比如RubyGems，NPM包，Pip，Composer等。

要找出是否是这个原因，重启一次曾经是绿色的变更，例如上次已知有效的那个。如果那次构建也突然失败了，那么很可能是一个依赖更新并引起了中断。

确保检查了构建日志中的依赖列表，通常输出中包含版本，并检查是否有什么东西变更了。

有时可能是非直接依赖更新引起的。

在找出更新的依赖后，将其锁定为上次已知的版本。

此外我们定期更新我们的构建环境，这将会带来语言和运行的服务的较新版本。确保关注我们的[changelog](http://changelog.travis-ci.com)来得到最近的更新。

## 我的构建脚本被杀而没有任何错误

有时你会看到一个构建脚本引起了错误，日志中的消息类似于`Killed`。

这通常是由于脚本或者某个程序在构建沙盒中耗尽了内存，目前是3GB。另外可以使用2个核，bursted。

依赖于正在使用的工具，这可能是由一些东西引起：

* Ruby测试套件使用了太多内存
* 测试并行运行，使用了太多进程或者线程（例如使用`parallel_test`gem）
* g++需要太多内存来编译文件，例如包含了许多模版

对于同时运行的并行进程，尝试减少数量。2-4个进程不错，超过这么多，资源可能耗尽。

对于Ruby进程，检查你本地机器的内存消耗，这大概会展示类似的原因。可能是内存泄漏或者定制的垃圾回收设定引起的，例如尽量晚地延迟清理。降低这些数字可能有用。

## Ruby：即使构建失败，RSpec也返回0

在某些情况下，在运行`rake rspec`或者甚至直接运行rspec时，即使构建失败，命令也返回0。这通常是由于一些RubyGem覆盖了另一个RubyGem的`at_exit`处理程序，在这里是RSpec的。

临时修复方案是在你的代码里安装`at_exit`处理程序，像[这篇文章](http://www.davekonopka.com/2013/rspec-exit-code.html)指出的那样。

    if defined?(RUBY_ENGINE) && RUBY_ENGINE == "ruby" && RUBY_VERSION >= "1.9"
      module Kernel
        alias :__at_exit :at_exit
        def at_exit(&block)
          __at_exit do
            exit_status = $!.status if $!.is_a?(SystemExit)
            block.call
            exit exit_status if exit_status
          end
        end
      end
    end

如果你的项目使用[Code Climate integration](/user/code-climate/)或者Simplecov，这个问题在Simplecov的0.8分支出现。修复方法是降级到最近的0.7发布，直到修复了这个问题。

## Capybara：我遇到了关于元素未找到的错误

在涉及JavaScript的情况下，你可能偶然发现指出一个元素缺失的错误，一个按钮，链接或者一些其他由异步JavaScript更新或者创建的资源。

这可以看出Selenium或者其某个驱动的超时设置太低。

Capybara的超时设置你可以增加到至少15秒：

    Capybara.default_wait_time = 15

Poltergeist有它自己的超时设置：

    Capybara.register_driver :poltergeist do |app|
      Capybara::Poltergeist::Driver.new(app, timeout: 15)
    end

如果你在最初增加了它之后依然遇到超时，那么将一次测试运行设置为高很多的值。如果错误依然存在，那么可能在页面内有更深层次的问题，例如编译资源。

## Ruby：安装debugger_ruby-core-source library失败

不幸的是Ruby库有着甚至因为补丁级别的发布而导致中断的历史。这一般是由于依赖的库，比如linecache活着其他Ruby调试库。

我们建议将这些库移动到你的Gemfile中独立的组中，然后再Travis安装非这些组的RubyGems。由于这些库只对本地开发有用，在你构建的安装进程中你甚至会得到加速。

    # Gemfile
    group :debug do
      gem 'debugger'
      gem 'debugger-linecache'
      gem 'rblineprof'
    end

    # .travis.yml
    bundler_args: --without development debug

## Mac：代码签名错误

在Mavericks下，代码签名与keychain应用的条款修改了很多。

有问题的签名可能是指出身份未找到与“用户交互不允许”的错误信息。

keychain必须被标记为默认的keychain，必须明确地解锁，并且构建需要确保在到达构建的鉴定节点前keychain不被锁。下面的命令集合处理这种情况：

    KEY_CHAIN=ios-build.keychain
    security create-keychain -p travis $KEY_CHAIN
    # Make the keychain the default so identities are found
    security default-keychain -s $KEY_CHAIN
    # Unlock the keychain
    security unlock-keychain -p travis $KEY_CHAIN
    # Set keychain locking timeout to 3600 seconds
    security set-keychain-settings -t 3600 -u $KEY_CHAIN

## Mac：运行CocoaPods出错

目前CocoaPods的使用可能由于一些原因失败。

### 需要较新版本的CocoaPods

大多数Pods目前需要CocoaPods 0.32.1，但是我们仍然预装了0.21。如果你遇到这个错误，添加如下内容到你的`.travis.yml`：

    before_install:
      - gem install cocoapods -v '0.32.1'

### CocoaPods未找到

CocoaPods目前并不在所有可用的Ruby上安装，不幸的是这意味着在使用默认版本2.0.0的Ruby时将会失败。

要避免这个问题，你可以如上所示手动安装CocoaPods，或者在你的`.travis.yml`中切换到Ruby 1.9.3，这样将会没有任何问题地工作：

    rvm: 1.9.3

### CocoaPods段错误失败

在Ruby 2.0.0，CocoaPods会因为一个段错误崩溃。

你可以通过使用Ruby 1.9.3来避免，这样就不会显示这些问题。添加如下内容到你的`.travis.yml`：

    rvm: 1.9.3

## 系统：未安装所需的语言包

Travis CI构建环境目前只安装了en_US语言包。如果你遇到类似“Error: unsupported locale
setting”的错误，那么你可能需要在你的测试运行时安装另一个语言包。

可以通过在你的`.travis.yml`中添加如下内容来完成：

    before_install:
      - sudo apt-get update && sudo apt-get --reinstall install -qq language-pack-en language-pack-de

上面的额外内容将会重新安装en_US与de_DE语言包。

## Linux：apt由于404错误安装包失败

这通常是由于老的包数据库引起的，可以通过在你的`.travis.yml`添加如下内容来修复：

    before_install:
      - sudo apt-get update


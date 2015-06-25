---
title: 在Travis CI上使用Code Climate
layout: en
permalink: /user/code-climate/
---
[Code Climate](https://www.codeclimate.com)是一个持续测量与监控代码质量的托管平台。

它关注你的[代码整体质量](https://codeclimate.com/tour)，但是它也可以[追踪测试覆盖率](https://codeclimate.com/tour/test-coverage)。为了那个目的，它可以彻底与Travis CI集成。

Code Climate支持Ruby，JavaScript与PHP（目前是beta）项目。

作为Travis CI用户，[你可以得到前三个月20%减免](https://codeclimate.com/partners/travisci)！

## 用Code Climate测量测试覆盖率

测试覆盖率集成当前对于Ruby，JavaScript和PHP项目可用，可以在私有与开源项目中使用，并对开源项目免费。

### Ruby

对于Ruby项目你需要添加一个称为[`code-climate-reporter`](https://github.com/codeclimate/ruby-test-reporter)的库到你的Gemfile：

    gem "codeclimate-test-reporter", group: :test, require: nil

接下来你的`test_helper.rb`或`spec_helper.rb`上部require reporter：

    require "codeclimate-test-reporter"
    CodeClimate::TestReporter.start

作为最后一步，你需要告诉Travis CI用来传送覆盖率结果的token。你可以在Code Climate上你的库的设置中找到这个token。然后你可以将其添加到你的`.travis.yml`：

    addons:
      code_climate:
        repo_token: adf08323...

### JavaScript

JavaScript项目可以使用[Code Climate的JavaScript
reporter库](https://www.npmjs.org/package/codeclimate-test-reporter)测量测试覆盖率。

覆盖率数据应该使用Lcov格式来生成，例如使用[istanbul库](https://www.npmjs.com/package/istanbul)。

你可以在你的.travis.yml中指出库的token，它将作为一个环境变量自动导出：

    addons:
      code_climate:
        repo_token: aff33f...

要将测试覆盖率数据报告给Code Climate，你只需要在你的测试后运行如下内容：

    after_script:
      - codeclimate < lcov.info

### PHP

PHP项目可以使用[Code Climate的PHP报告者](https://github.com/codeclimate/php-test-reporter)来收集代码覆盖率数据。

要为你的项目建立起来，可以按照[README](https://github.com/codeclimate/php-test-reporter#usage)中的指引。

可以在你的.travis.yml中指出库的token

    addons:
      code_climate:
        repo_token: aff33f...

假设报告者是你的Composer bundle的一部分，需要添加如下内容到你的.travis.yml，从而报告覆盖率数据给Code Climate：

    after_script:
      - vendor/bin/test-reporter

## 常见问题

#### 我在传输覆盖率结果时遇到一个错误

如果你的项目在使用WebMock来模拟HTTP请求，你将需要明确地将Code Climate API加入白名单，否则你将会遇到覆盖率结果不会传输的错误。这里是如何将Code Climate加入白名单：

    WebMock.disable_net_connect!(allow: %w{codeclimate.com})

这么做的一种方法是将如下内容添加到你的`spec_helper.rb`：

    # whitelist codeclimate.com so test coverage can be reported
    config.after(:suite) do
      WebMock.disable_net_connect!(:allow => 'codeclimate.com')
    end

#### 我希望在并行构建中使用Code Climate

Code Climate当前并不从多个测试运行中收集测试覆盖率结果。这意味着你还不能在`parallel_test`这样的库或者使用我们构建矩阵的并行构建中有效地使用它。

#### 虽然rspec失败了但是我的构建成功了

由于一个[Simplecov的0.8发布分支的bug](https://github.com/colszowka/simplecov/issues/281)，rspec的退出代码被Simplecov覆盖，使得一个失败的构建看起来成功了。

在Simplecov修复这个问题前，建议使用没有这个问题的最新的0.7发布来替代。

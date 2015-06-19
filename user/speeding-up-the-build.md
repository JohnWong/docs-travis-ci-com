---
title: 加速构建
layout: en
permalink: /user/speeding-up-the-build/
---

Travis CI实现了一些有助于加速你的构建的优化，比如数据库文件的内存文件系统，还有一些能够更多地改进构建时间的事情可以做。

## 在虚拟机间并行化你的构建

要加速你的测试套件，你可以使用Travis CI的[构建矩阵](/user/build-configuration/#The-Build-Matrix)特性将其拆分几部分。

假设你希望将你的单元测试与集成测试拆分为两个不同的构建工作。它们将会并行运行，并完全利用你账户的可用构建能力。

这里的例子是在你的.travis.yml中如何利用这个特性：
    
    env:
      - TEST_SUITE=units
      - TEST_SUITE=integration

然后你使用新的环境变量修改你的脚本命令来确定要运行的脚本。

    script: "bundle exec rake test:$TEST_SUITE"

Travis CI将会依据环境变量确定构建矩阵，并安排运行两个构建。

这个设置的奇妙之处在于单元测试用例套件通常会在集成测试套件之前完成，这会给你基本测试覆盖率的快速可视反馈。

依赖于你的测试套件的大小与复杂度，你可以拆分更细。你可以将不同关注点的集成测试分到不同的子目录下，在一个构建矩阵的的不同阶段运行。

    env:
      - TESTFOLDER=integration/user
      - TESTFOLDER=integration/shopping_cart
      - TESTFOLDER=integration/payments
      - TESTFOLDER=units

然后你可以调整你的脚本命令为每个子目录运行rspec。

    script: "bundle exec rspec $TESTFOLDER"

例如，Rails项目使用构建矩阵特性来为每个数据库创建分开的工作来测试，依据关注来拆分测试。一个集合只为railties运行测试，另一个为actionpack，actionmailer，activesupport，一整个集合针对不同数据库运行activerecord测试。更多例子参见他们的[.travis.yml
file](https://github.com/rails/rails/blob/master/.travis.yml)。


注意在试用期间，你在私有库上只有一个并发构建可以使用，因此在你注册了付费订阅后才能看到提升。

## 在虚拟机上并行化你的构建

Travis CI虚拟机运行在1.5的虚拟内核上。严格说来这并不是并发，并发允许并行化许多，但是它可以给你的测试用例上提供一个很好的加速。

在一个虚拟机上测试套件的并行化依赖于你使用的语言与测试运行程序，因此你不得不研究你的选择。在Travis CI我们主要使用Ruby与RSpec，这意味着我们可以使用[parallel_tests](https://github.com/grosser/parallel_tests) gem。如果你使用Java，你可能使用内建特性来[使用JUnit来并行运行测试](http://incodewetrustinc.blogspot.com/2009/07/run-your-junit-tests-in-parallel-with.html)。

为了让你知道我们在谈论的是什么测试，我尝试在`travis-core`上并行运行测试，我能够看到使用4个工作后时间从26分钟下降到19分钟。

## 在多个虚拟机上并行化RSpec与Cucumber

如果你希望在多个虚拟机上并行化RSpec与Cucumber测试来从命令行得到更快反馈，那么你可以尝试[knapsack](https://github.com/ArturT/knapsack) gem。它将会在虚拟机间拆分测试，确保测试在每个虚拟机上将会运行可比较的时间（每个工作将会花费类似的事件）。你可以使用你的矩阵特性来设置knapsack。

### RSpec并行化示例

    script: "bundle exec rake knapsack:rspec"
    env:
      global:
        - MY_GLOBAL_VAR=123
        - CI_NODE_TOTAL=2
      matrix:
        - CI_NODE_INDEX=0
        - CI_NODE_INDEX=1

这样的配置将会生成包含下面2行ENV的矩阵：

    CI_NODE_TOTAL=2 CI_NODE_INDEX=0 MY_GLOBAL_VAR=123
    CI_NODE_TOTAL=2 CI_NODE_INDEX=1 MY_GLOBAL_VAR=123

### Cucumber并行化示例

    script: "bundle exec rake knapsack:cucumber"
    env:
      global:
        - MY_GLOBAL_VAR=123
        - CI_NODE_TOTAL=2
      matrix:
        - CI_NODE_INDEX=0
        - CI_NODE_INDEX=1

### RSpec与Cucumber并行化示例

如果你希望同时并行化RSpec与Cucumber测试，那么这样在`.travis.yml`定义脚本：

    script:
      - "bundle exec rake knapsack:rspec"
      - "bundle exec rake knapsack:cucumber"

你可以在[knapsack docs](https://github.com/ArturT/knapsack#info-for-travis-users)发现更多例子。

## 缓存依赖

对于大项目来说，为一个项目安装依赖会耗费相当长时间。为了让它更快，你可以尝试缓存依赖。

一可以使用我们的[内建缓存](/user/caching/)或者在S3上建立你自己的。如果你希望建立你自己的并且你使用Ruby与Bundler，检出[the great WAD project](https://github.com/Fingertips/WAD)。

对于其他语言，你可以直接使用s3工具来上传下载依赖。

---
title: 定制构建
layout: en
permalink: /user/customizing-the-build/
---

<div id="toc"></div>

Travis CI提供了一个默认的构建环境和一个每种编程语言的一系列默认步骤。你可以在 `.travis.yml` 中定制这个过程的任意一步。Travis CI使用你代码库的根目录下的 `.travis.yml` 文件来了解你的项目和你希望如何执行构建。`.travis.yml` 可以非常精简或者有许多定制。你的 `.travis.yml` 文件可能包含的信息类型的例子：

* 你的项目所使用的编程语言
* 在你每次构建前你希望执行什么命令或者脚本（例如，安装或者克隆你的项目的依赖）
* 使用什么命令来运行你的测试套件
* 提醒你构建失败的电子邮件，Campfire和IRC房间

## 构建的生命周期

Travis CI的一次构建由两步组成：

1. **install**: 安装任何所需的依赖
2. **script**: 运行构建脚本

你可以在安装这一步之前运行自定义的脚本（`before_install`），和脚本这一步之前的（`before_script`）或者之后的（`after_script`）。

在`before_install`这一步中，你可以安装你的项目所需的额外的依赖，比如Ubuntu包或者自定义服务。

你可以在构建成功或者失败时使用`after_success`（比如构建文档或者部署到自定义的服务器）或者`after_failure`（比如上传日志文件）选项来执行额外的步骤。在 `after_failure` 和 `after_success`，你可以使用环境变量`$TRAVIS_TEST_RESULT`获取构建结果。

完整的构建生命周期，包括了在检查git库和改变库目录后的三个可选的部署步骤，它们是：

1. `before_install`
2. `install`
3. `before_script`
4. `script`
5. `after_success` or `after_failure`
6. `after_script`
7. OPTIONAL `before_deployment`
8. OPTIONAL `deployment`
9. OPTIONAL `after_deployment`


## 定制安装步骤

默认的依赖安装命令取决于项目语言。比如Java构建使用Maven或者Gradle，取决于库中存在出哪个构建文件。Ruby项目在库中存在一个Gemfile时使用Bundler。

你可以指定你自己的脚本来安装 `.travis.yml` 中你的项目所需要的任何依赖：

    install: ./install-dependencies.sh

> 当使用自定义脚本时，他们应该是可执行的（比如使用 `chmod +x`）并且包含了有效的一行shebang，比如 `/usr/bin/env sh`，`/usr/bin/env ruby`或者`/usr/bin/env python`。

你也可以提供多个步骤，比如同时安装ruby和node依赖：

    install:
      - bundle install --path vendor/bundle
      - npm install

当其中一步失败了，构建立即停止并标记为[错误](#Breaking-the-Build)。

## 定制构建步骤

默认构建命令依赖于项目语言。Ruby项目使用 `rake`，是大多数Ruby项目的共同特征。

你可以覆盖 `.travis.yml` 中的默认构建步骤：

    script: bundle exec thor build

你也可以指定多个脚本命令：

    script:
      - bundle exec rake build
      - bundle exec rake builddoc

当其中一个构建命令返回了一个非零的推出状态码，Travis CI构建也会运行随后的构建命令，并累积构建结果。

在上面的例子中，如果 `bundle exec rake build` 返回推出状态码1，后面的命令 `bundle exec rake builddoc` 仍然会运行，但是构建的结果将会是失败。

如果你的第一步是运行单元测试，紧接着是集成测试，你可能仍然希望在单元测试失败的时候仍然看集成测试是否成功。

你可以通过使用一点命令行魔法来改变这种行为，运行后续所有命令但是当第一个命令返回非零推出状态码时构建仍然是失败的。这里是 `.travis.yml` 里的代码片段。

    script: bundle exec rake build && bundle exec rake builddoc

这个例子（注意 `&&`）在 `bundle exec rake build` 失败的时候会立即失败。

### 注意 $?

`script` 中的每个命令都由一个特殊的bash函数处理。这个函数操作 `$?` 来产生适于显示的日志。因此你不应该依赖 `script` 部分的 `$?` 的值来改变构建行为。更多技术讨论参见[这个GitHub issue](https://github.com/travis-ci/travis-ci/issues/3771)。

## 阻断构建

如果前四步中的任何一个命令返回非零退出状态码，Travis CI会认为构建被阻断。

当`before_install`，`install`或者`before_script`阶段中的任何一步失败返回非零退出状态码，构建就会被标记为 **errored**。

当`script`阶段的任何步骤失败返回非零退出状态码，构建会被标记为 **failed**。

> 注意，`script` 部分与其他步骤相比有不同的语义。当 `script` 中定义的一个步骤失败了，构建并不会立即结束，而是继续执行剩下的步骤直到构建失败。

目前，`after_success` 或者 `after_failure` 都不会对构建结果有任何影响。我们有计划改变这种行为。

## 部署你的代码

构建生命周期的一个可选的阶段是部署。这一步并不能被覆盖，而是使用我们的持续部署代码到Heroku，Engine Yard或者其他支持的平台的部署提供者来定义。

你可以使用 `before_deploy` 在一次部署前运行一些步骤。这个命令中的非零返回状态码将会把构建标记为 **errored**。

如果有任何想要在部署之后运行的步骤，你可以使用 `after_deploy` 阶段。

## 指定运行时版本

Travis CI的一个主要特性是在多个运行时喝版本下运行测试套件的容易性。在 `.travis.yml` 文件中指定使用什么语言和运行时来运行你的测试套件。

{% include languages.html %}

## 使用apt来安装包

如果你的依赖需要原生库，那么你可以使用 **无密码的sudo来安装他们**：

```yml
before_install:
	- sudo apt-get update -qq
	- sudo apt-get install -qq [packages list]
```

> 注意，这个特性在运行在[基于容器的Worker](/user/ci-environment/#Virtualization-environments)的构建上不可用。

所有的虚拟机都保存快照，并在每次构建后恢复到其初始状态。

### 使用第三方PPA

如果你需要的原生库在Ubuntu的官方库里不可用，有一个[第三方PPA](https://launchpad.net/ubuntu/+ppas)可供你使用。

## 构建超时

由于测试套件或者构建脚本挂起很常见，Travis CI对每个工作都有指定的时间限制。如果一个脚本或者测试套件耗费了超过50分钟的时间（或者在travis-ci.com耗费超过120分钟），或者超过10分钟没有日志输出，它将会被终止，构建日志中会写入一条消息。

一些常见的构建挂起原因：

* 等待键盘输入或者其他类型的人机交互
* 并发问题（死锁，活锁等）
* 安装耗费很长时间来编译的原生扩展

> 构建并没有超时；构建将会执行每一个工作直到这个工作超时。

## 构建指定的分支

Travis CI使用触发这次构建的git提交所在的分支的 `.travis.yml` 文件。你可以通过使用白名单盒黑名单来告诉Travis构建多个分支。

### 白名单或黑名单分支

使用白名单指定哪些分支需要构建，或者你不希望构建的黑名单分支：

    # blacklist
    branches:
      except:
        - legacy
        - experimental

    # whitelist
    branches:
      only:
        - master
        - stable

如果你两者都指定了，`only` 比 `except` 优先级更高。 `gh-pages` 分支默认不构建，除非你将它添加到白名单。

> 注意，由于历史原因， `.travis.yml` 需要出现在你项目的 *所有活跃分支* 中。

### 使用正则表达式

你可以使用正则表达式来指定分支的白名单或者黑名单哪：

    branches:
      only:
        - master
        - /^deploy-.*$/

分支列表中任何分支使用 `/` 包围的名称都被当作一个正则表达式，并且可以包含任何量词、锚点或者任何[Ruby正则表达式](http://www.ruby-doc.org/core-1.9.3/Regexp.html)支持的字符类。

后一个 `/` 之后指定的选项（例如 `i` 表示大小写不敏感匹配）并不支持，但是可以替换为行内给出。例如 `/^(?i:deploy)-.*$/` 匹配 `Deploy-2014-06-01` 和其他以任何大小写组合的 `deploy-` 开始的分支和标签（tag)。 

## 跳过一次构建

如果不你不希望为某一个特殊的提交运行一次构建，原因是例如你所修改的是README，添加 `[ci skip]` 到提交消息中。

在提交消息任何地方包含 `[ci skip]` 的提交将会被Travis CI忽略。

## 构建矩阵

当你把 *Runtime*、*Environment* 和 *Exclusions/Inclusions* 这三个主要配置选项组合起来的时候，你会有一个所有可能组合的矩阵。

下面是一个扩展到 *56个独立（7 * 4 * 2）* 构建矩阵的示例配置。

    rvm:
      - 1.8.7
      - 1.9.2
      - 1.9.3
      - rbx-2
      - jruby
      - ruby-head
      - ree
    gemfile:
      - gemfiles/Gemfile.rails-2.3.x
      - gemfiles/Gemfile.rails-3.0.x
      - gemfiles/Gemfile.rails-3.1.x
      - gemfiles/Gemfile.rails-edge
    env:
      - ISOLATED=true
      - ISOLATED=false

你可以可以定义构建矩阵的一些排除的组合：

    matrix:
      exclude:
        - rvm: 1.8.7
          gemfile: gemfiles/Gemfile.rails-2.3.x
          env: ISOLATED=true
        - rvm: jruby
          gemfile: gemfiles/Gemfile.rails-2.3.x
          env: ISOLATED=true

> 请考虑下Travis CI是一个开源服务，我们依靠社区提供的worker容器。因此请只在你 *确实需要* 的时候指定一个大的矩阵。

### 排除构建

如果你想要从矩阵中排除的构建拥有相同的矩阵参数，你可以只指定这些并省略变化的部分。

Suppose you have:

```yml
language: ruby
rvm:
	- 1.9.3
	- 2.0.0
	- 2.1.0
env:
	- DB=mongodb
	- DB=redis
	- DB=mysql
gemfile:
	- Gemfile
	- gemfiles/rails4.gemfile
	- gemfiles/rails31.gemfile
	- gemfiles/rails32.gemfile
```

这将会导致一个3×3×4的构建矩阵。要排除所有 `rvm` 的值是 `2.0.0` 并且 
`gemfile` 的值是 `Gemfile` 的工作，你可以这样写：

```yml
matrix:
	exclude:
	- rvm: 2.0.0
		gemfile: Gemfile
```

这也等价于：

```yml
matrix:
	exclude:
	- rvm: 2.0.0
		gemfile: Gemfile
		env: DB=mongodb
	- rvm: 2.0.0
		gemfile: Gemfile
		env: DB=redis
	- rvm: 2.0.0
		gemfile: Gemfile
		env: DB=mysql
```

### 明确包含构建

也可以使用 `matrix.include` 来为矩阵添加一些条目：

    matrix:
      include:
        - rvm: ruby-head
          gemfile: gemfiles/Gemfile.rails-3.2.x
          env: ISOLATED=false

这将会给已经形成的构建矩阵增加一个特殊的工作。

如果你希望只在最新版本的运行时上测试一个依赖的最新版本，那么这将会很有用。

你可以使用这种方法来创建一个只包含指定组合的工作矩阵。例如：

    language: python
    matrix:
      include:
        - python: "2.7"
          env: TEST_SUITE=suite_2_7
        - python: "3.3"
          env: TEST_SUITE=suite_3_3
        - python: "pypy"
          env: TEST_SUITE=suite_pypy
    script: ./test.py $TEST_SUITE

创建一个构建矩阵，包含在每个版本的Python上运行测试套件的3个工作。

### 允许失败的行

你可以在构建矩阵中定义允许失败的行。允许的失败是你的构建矩阵中允许失败但不引起整个构建失败的条目。这使得你可以添加实验中的和预备的构建来测试你并未准备好正式支持的版本或配置。

在构建矩阵中使用键值对来定义允许的失败：

    matrix:
      allow_failures:
        - rvm: 1.9.3

### 快速完成

如果构建矩阵中的一些行允许失败，那么直到这些构建矩阵全部完成，构建才会被标记为完成。

要设置构建尽快完成，像这样添加 `fast_finish: true` 到你的 `.travis.yml` 的 `matrix` 部分：

    matrix:
      fast_finish: true

现在，构建将会在一个工作失败或者剩下的工作都允许失败时完成。

## 实现复杂的构建步骤

如果你有一个难以在 `.travis.yml` 中配置的复杂的构建环境，可以考虑把这些步骤移动到一个分开的shell脚本中。脚本可以使你的代码库中的一部分，可以从 `.travis.yml` 方便地调用。

考虑一个场景，你想要运行更多复杂的测试场景，但是只针对非来自pull request的构建。一个shell脚本可能是：

```sh
#!/bin/bash
set -ev
bundle exec rake:units
if [ "${TRAVIS_PULL_REQUEST}" = "false" ]; then
	bundle exec rake test:integration
fi
```

注意上面的 `set -ev` 。 `-e` 标志会导致脚本在一个命令返回一个非零退出码时尽快退出。如果你希望你的无论什么情况脚本早点退出，那么这将是方便的。如果你的复杂安装脚本符合一个失败的命令并不会引起安装失败，那么这也是有用的。

`-v` 标记将会使shell在执行它们之前打印所有的行，这将帮助我们辨别哪个步骤失败了。

假设上面的脚本存储在你代码库的 `scripts/run-tests.sh` 中，并且也有合适的权限（在检入它之前运行 `chmod ugo+x scripts/run-tests.sh`），你可以在 `.travis.yml` 中调用它：

    script: ./scripts/run-tests.sh

### 这如何工作？（或者为什么你不应该在构建步骤中使用 `exit` )

在构建生命周期中指定的步骤会被编译进入一个独立的bash脚本，由worker执行。

当覆盖这些步骤时，不要使用shell的内建命令 `exit`。这么做将会引起不给Travis一个运行后续任务而终止构建流程的风险。

在一个构建过程中会被唤起的自定义脚本中使用 `exit` 是允许的。

## 自定义主机名

如果你的构建需要设置自定义主机名，你可以在你的 ｀.travis.yml.｀ 中指定一个主机名或者一个主机名列表。Travis CI将会在 `/etc/hosts` 中为IPv4和IPv6自动设置主机名。

    addons:
      hosts:
        - travis.dev
        - joshkalderimis.com


## 构建问答

### Travis CI在构建中不保留任何状态

Travis CI使用虚拟主机快照来确保构建之间不会留下任何状态。如果你通过写什么到数据存储，创建文件或者通过apt安装一个包来修改CI环境，这并不会影响接下来的构建。

### SSH

Travis CI通过隔离的虚拟机的SSH来运行所有的命令。修改SSH会话状态的命令是“黏”的，并且会在整个构建中保留。例如如果你 `cd` 到一个特殊的目录，接下来所有的命令将会从那里执行。这可能是好的（比如依次构建子工程）或者影响可能在当前目录寻找文件的工具比如 `rake` or `mvn`。

### Git子模块

在代码库的跟目录下存在 `.gitmodules` 文件时，Travis CI自动初始化并升级子模块。

这可以通过如下设置关闭:

    git:
      submodules: false

如果你的项目的Git子模块需要一些Travis CI盒子并不支持的特殊选项，你可以关闭自动集成并使用 `before_install` 钩子来初始化或升级它们。

例如：

    before_install:
      - git submodule update --init --recursive

如果存在嵌套的子模块（子模块的子模块），那么这将会包含它们。

### 为子模块使用公共网址

如果你的项目使用Git子模块，确保使用公共Git网址。例如在GitHub，将

    git@github.com:someuser/somelibrary.git

替换为

    git://github.com/someuser/somelibrary.git

否则Travis CI构建时无法克隆你的项目，因为它们并没有你的私有SSH key。

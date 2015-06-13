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

> Note that the `script` section has different semantics to the other
steps. When a step defined in `script` fails, the build doesn't end right away,
it continues to run the remaining steps before it fails the build.

Currently, neither the `after_success` nor `after_failure` have any influence on the build result. We have plans to change this behaviour.

## Deploying your Code

An optional phase in the build lifecycle is deployment. This step can't be overridden, but is defined by using one of our continuous deployment providers to deploy code to Heroku, Engine Yard, or a different supported platform.

You can run steps before a deploy by using the `before_deploy` phase. A non-zero exit code in this command will mark the build as **errored**.

If there are any steps you'd like to run after the deployment, you can use the `after_deploy` phase.

## Specifying Runtime Versions

One of the key features of Travis CI is the ease of running your test suite against multiple runtimes and versions. Specify what languages and runtimes to run your test suite against in the `.travis.yml` file:

{% include languages.html %}

## Installing Packages Using apt

If your dependencies need native libraries to be available, you can use **passwordless sudo to install them**:

```yml
before_install:
	- sudo apt-get update -qq
	- sudo apt-get install -qq [packages list]
```

> Note that this feature is not available for builds that are running on [Container-based workers](/user/ci-environment/#Virtualization-environments)

All virtual machines are snapshotted and returned to their intial state after each build. 

### Using 3rd-party PPAs

If you need a native dependency that is not available from the official Ubuntu repositories, there might be a [3rd-party PPAs](https://launchpad.net/ubuntu/+ppas) that you can use.

## Build Timeouts

Because it is very common for test suites or build scripts to hang, Travis CI has specific time limits for each job. If a script or test suite takes longer than 50 minutes (or 120 minutes on travis-ci.com), or if there is not log output for 10 minutes, it is terminated, and a message is written to the build log.

Some common reasons why builds might hang:

* Waiting for keyboard input or other kind of human interaction
* Concurrency issues (deadlocks, livelocks and so on)
* Installation of native extensions that take very long time to compile

> There is no timeout for a build; a build will run as long as all the jobs do as long as each job does not timeout.

## Building Specific Branches

Travis CI uses the `.travis.yml` file from the branch specified by the git commit that triggers the build. You can tell Travis to build multiple branches suing blacklists or whitelists.

### Whitelisting or blacklisting branches

Specify which branches to build using a whitelist, or blacklist branches that you do not want to be built:

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

If you specify both, `only` takes precedence over `except`. By default, `gh-pages` branch are not built unless you add it to the whitelist.

> Note that for historical reasons `.travis.yml` needs to be present *on all active branches* of your project.

### Using regular expressions ###

You can use regular expressions to whitelist or blacklist branches:

    branches:
      only:
        - master
        - /^deploy-.*$/

Any name surrounded with `/` in the list of branches is treated as a regular expression and can contain any quantifiers, anchors or character classes supported by [Ruby regular expressions](http://www.ruby-doc.org/core-1.9.3/Regexp.html).

Options that are specified after the last `/` (e.g., `i` for case insensitive matching) are not supported but can be given inline instead.  For example, `/^(?i:deploy)-.*$/` matches `Deploy-2014-06-01` and other
branches and tags that start with `deploy-` in any combination of cases.

## Skipping a build

If you don't want to run a build for a particular commit, because all you are
changing is the README for example, add `[ci skip]` to the git commit message. 

Commits that have `[ci skip]` anywhere in the commit messages are ignored by 
Travis CI.

## Build Matrix

When you combine the three main configuration options of *Runtime*, *Environment* and *Exclusions/Inclusions* you have a matrix of all possible combinations. 

Below is an example configuration for a build matrix that expands to *56 individual (7 * 4 * 2)* builds.

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

You can also define exclusions to the build matrix:

    matrix:
      exclude:
        - rvm: 1.8.7
          gemfile: gemfiles/Gemfile.rails-2.3.x
          env: ISOLATED=true
        - rvm: jruby
          gemfile: gemfiles/Gemfile.rails-2.3.x
          env: ISOLATED=true

> Please take into account that Travis CI is an open source service and we rely on worker boxes provided by the community. So please only specify an as big matrix as you *actually need*.

### Excluding Builds 

If the builds you want to exclude from the matrix share the same matrix
parameters, you can specify only those and omit the varying parts.

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

This results in a 3×3×4 build matrix. To exclude all jobs which have `rvm` value `2.0.0` *and*
`gemfile` value `Gemfile`, you can write:

```yml
matrix:
	exclude:
	- rvm: 2.0.0
		gemfile: Gemfile
```

Which is equivalent to:

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

### Explicity Including Builds

It is also possible to include entries into the matrix with `matrix.include`:

    matrix:
      include:
        - rvm: ruby-head
          gemfile: gemfiles/Gemfile.rails-3.2.x
          env: ISOLATED=false

This adds a particular job to the build matrix which has already been populated.

This is useful if you want to, only test the latest version of a dependency together with the latest version of the runtime.

You can use this method to create a job matrix containing only specific combinations. 
For example,

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

creates a build matrix with 3 jobs, which runs test suite for each version
of Python.

### Rows that are Allowed To Fail

You can define rows that are allowed to fail in the build matrix. Allowed
failures are items in your build matrix that are allowed to fail without causing
the entire build to fail. This lets you add in experimental and
preparatory builds to test against versions or configurations that you are not
ready to officially support.

Define allowed failures in the build matrix as key/value pairs:

    matrix:
      allow_failures:
        - rvm: 1.9.3

### Fast finishing

If some rows in the build matrix that are allowed to fail, the build won't be marked as finished until they have completed.

To set the build to finish as soon as possible, add `fast_finish: true` to the `matrix` section of your `.travis.yml` like this:

    matrix:
      fast_finish: true

Now, a build will finish as soon as a job has failed, or when the only jobs left allow failures.


## Implementing Complex Build Steps

If you have a complex build environment that is hard to configure in the `.travis.yml`, consider moving the steps into a separate shell script. 
The script can be a part of your repository and can easily be called from the `.travis.yml`. 

Consider a scenario where you want to run more complex test scenarios, but only for builds that aren't coming from pull requests. A shell script might be:

```sh
#!/bin/bash
set -ev
bundle exec rake:units
if [ "${TRAVIS_PULL_REQUEST}" = "false" ]; then
	bundle exec rake test:integration
fi
```

Note the `set -ev` at the top. The `-e` flag causes the script to exit as soon as one command returns a non-zero exit code. This can be handy if you want whatever script you have to exit early. It also helps in complex installation scripts where one failed command wouldn't otherwise cause the installation to fail.

The `-v` flag makes the shell print all lines in the script before executing them, which helps identify which steps failed.

Assuming the script above is stored as `scripts/run-tests.sh` in your repository, and with the right permissions too (run `chmod ugo+x scripts/run-tests.sh` before checking it in), you can call it from your `.travis.yml`:

    script: ./scripts/run-tests.sh

### How does this work? (Or, why you should not use `exit` in build steps)

The steps specified in the build lifecycle are compiled into a single bash script and executed on the worker.

When overriding these steps, do not use `exit` shell built-in command.
Doing so will run the risk of terminating the build process without giving Travis a chance to
perform subsequent tasks.

Using `exit` inside a custom script which will be invoked from during a build is fine. 

## Custom Hostnames

If your build requires setting up custom hostnames, you can specify a single host or a
list of them in your .travis.yml. Travis CI will automatically setup the
hostnames in `/etc/hosts` for both IPv4 and IPv6.

    addons:
      hosts:
        - travis.dev
        - joshkalderimis.com


## Build FAQ

### Travis CI Preserves No State Between Builds

Travis CI uses virtual machine snapshotting to make sure no state is left between builds. If you modify CI environment by writing something to a data store, creating files or installing a package via apt, it won't affect subsequent builds.

### SSH

Travis CI runs all commands over SSH in isolated virtual machines. Commands that modify SSH session state are "sticky" and persist throughout the build.
For example, if you `cd` into a particular directory, all the following commands will be executed from it. This may be used for good (e.g. building subprojects one
after another) or affect tools like `rake` or `mvn` that may be looking for files in the current directory.

### Git Submodules

Travis CI automatically initializes and updates submodules when there's a `.gitmodules` file in the root of the repository.

This can be turned off by setting:

    git:
      submodules: false

If your project requires some specific option for your Git submodules which Travis CI does not support out of the box, then you can turn the automatic integration off and use the `before_install` hook to initializes and update them.

For example:

    before_install:
      - git submodule update --init --recursive

This will include nested submodules (submodules of submodules), in case there are any.


### Use Public URLs For Submodules

If your project uses Git submodules, make sure you use public Git URLs. For example, on GitHub, instead of

    git@github.com:someuser/somelibrary.git

use

    git://github.com/someuser/somelibrary.git

Otherwise, Travis CI builders won't be able to clone your project because they don't have your private SSH key.


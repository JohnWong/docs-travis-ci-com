---
title: Environment Variables
layout: en
permalink: /user/environment-variables/
---

一个常用的定制构建流程的方法是使用环境变量，可以在你的构建的任何阶段访问它。

<div id="toc"></div>

* [.travis.yml](#Defining-Variables-in-.travis.yml)中定义的变量绑定到某一个提交上。修改它们需要一次新提交，重启一次老的构建会使用旧的变量。它们在fork的库中自动可用。在`.travis.yml`中定义这些变量：

	+ 构建运行所需并且不包含敏感数据。例如Ruby的测试套件可能需要将`$RACK_ENV`设置为`test`。
	+ 每个分支不同的
	+ 每个工作不同的

* 定义在[repository settings](#Defining-Variables-in-Repository-Settings)中的变量对于所有构建来说相同。当你重新运行一个老的构建，它使用最新的值。这些变量在fork的库中并不自动可用。在Repository Settings中定义这些变量：

	+ 每个库不同的
	+ 包含敏感数据，比如第三方认证信息。

* 使用[Encrypted variables](#Encrypted-Variables)存放敏感数据，比如身份验证token。


> 如果你在`.travis.yml`与Repository Settings中定义了相同名称的变量，则优先使用`.travis.yml`中的。如果你将一个变量定义为加密的与不加密的，那么在文件中后定义的将会优先使用。

并没有在所有Travis CI环境中出现的一个[默认环境变量的完全列表](#Default-Environment-Variables)。

## 在.travis.yml中定义环境变量

要在你的`.travis.yml`中定义一个环境变量，添加`env`行，例如：

    env:
    - DB=postgres

> 注意环境变量值可能需要放入引号中。例如，如果你在其中需要星号（*）：
> 
> ````
> env:
> PACKAGE_VERSION="1.0.*"
> ````

### 每一条定义多个变量

如果你在`env`数组中每一项指定了多个变量（变量矩阵），每一条都会触发一次构建。

    rvm:
      - 1.9.3
      - rbx
    env:
      - FOO=foo BAR=bar
      - FOO=bar BAR=foo

这个配置将会触发**4次独立构建**：

1. Ruby 1.9.3与`FOO=foo`并且`BAR=bar`
2. Ruby 1.9.3与`FOO=bar`并且`BAR=foo`
3. 最新版Rubinius（rbx）与`FOO=foo`并且`BAR=bar`
4. 最新版Rubinius（rbx）与`FOO=bar`并且`BAR=foo`


### 全局变量

有时你可能希望使用矩阵中全局的环境变量，例如插入到每个矩阵行。它们可能包含密钥，token，URI或者其他每次构建需要的数据。在这种情况下你可以使用`global`与`matrix`键来区分两种情况而不必手动将这样的键加入到矩阵中的每一行。例如：

    env:
      global:
        - CAMPFIRE_TOKEN=abc123
        - TIMEOUT=1000
      matrix:
        - USE_NETWORK=true
        - USE_NETWORK=false

出发了下面`env`行的构建：

    USE_NETWORK=true CAMPFIRE_TOKEN=abc123 TIMEOUT=1000
    USE_NETWORK=false CAMPFIRE_TOKEN=abc123 TIMEOUT=1000

## Repository Settings中定义变量

要在Repository Settings中定义变量，确保你已经登陆，导航到正在考虑的repository，在齿轮菜单中选择“Settings”，点击“Environment Variables”部分的“Add new variable”。
  
<figure>
  <img src="{{ "/images/settings-env-vars.png" | prepend: site.baseurl }}">
  <figcaption>Repository Settings中的环境变量</figcaption>
</figure>

这些新环境变量的值在日志中`export`行默认隐藏。这是为了符合你的`.travis.yml`中[加密变量](#Encrypted-Variables)的行为。

类似地，我们在从其他库的pull request出发的非信任的构建中并不提供这些值。

作为一个web界面的替代选择，你可以使用命令行界面的[`env`](https://github.com/travis-ci/travis.rb#env)命令。

## 加密变量

可以加密变量，这样他们的内容在构建相应的`export`行不会显示。这是用来为Travis CI构建提供敏感数据，比如API认证信息。加密变量也不会添加到非信任构建中，比如来自另一个库的pull request。

一个`.travis.yml`文件包含的加密变量类似下面这样：

    env:
      global:
        - secure: <long encrypted string here>
        - TIMEOUT=1000
      matrix:
        - USE_NETWORK=true
        - USE_NETWORK=false
        - secure: <you can also put encrypted vars inside matrix>

> 加密的环境变量在来自fork的pull request中不可用，这是由于暴露这样的信息给未知的代码存在安全风险。

### 使用公共密钥加密变量

travis gem可以使用附加到你的库的公共密钥来加密环境变量：

    gem install travis
    cd my_project
    travis encrypt MY_SECRET_ENV=super_secret

要自动添加加密的环境变量到你的`.travis.yml`：

    travis encrypt MY_SECRET_ENV=super_secret --add env.matrix

> 加密和解密的密钥与库相关联。如果你fork一个库并将其添加到Travis CI，它将会有与原库不同的键。

The encryption scheme is explained in more detail in [Encryption keys](/user/encryption-keys).

### 便利变量

为了使加密环境变量更加容易，下面的环境变量时可用的。

* `TRAVIS_SECURE_ENV_VARS` “true”或者“false” 取决于环境变量是否可用
* `TRAVIS_PULL_REQUEST` 如果当前工作是一个pull request，那么变量是pull request的号码，否则是“false”

## 默认环境变量

下面的默认环境变量对所有的构建都可用。

* `CI=true`
* `TRAVIS=true`
* `CONTINUOUS_INTEGRATION=true`
* `DEBIAN_FRONTEND=noninteractive`
* `HAS_JOSH_K_SEAL_OF_APPROVAL=true`
* `USER=travis` (**不要依赖这个值**)
* `HOME=/home/travis` (**不要依赖这个值**)
* `LANG=en_US.UTF-8`
* `LC_ALL=en_US.UTF-8`
* `RAILS_ENV=test`
* `RACK_ENV=test`
* `MERB_ENV=test`
* `JRUBY_OPTS="--server -Dcext.enabled=false -Xcompile.invokedynamic=false"`
* `JAVA_HOME`被设置为适当的值

此外Travis CI设置你在你的构建中可以使用的环境变量，例如用来为构建打标签或者运行构建后的部署。


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

特定语言的构建暴露了用来运行构建的当前版本的额外环境变量。是否设置取决于你所使用的语言。

* `TRAVIS_DART_VERSION`
* `TRAVIS_GO_VERSION`
* `TRAVIS_HAXE_VERSION`
* `TRAVIS_JDK_VERSION`
* `TRAVIS_JULIA_VERSION`
* `TRAVIS_NODE_VERSION`
* `TRAVIS_OTP_RELEASE`
* `TRAVIS_PERL_VERSION`
* `TRAVIS_PHP_VERSION`
* `TRAVIS_PYTHON_VERSION`
* `TRAVIS_R_VERSION`
* `TRAVIS_RUBY_VERSION`
* `TRAVIS_RUST_VERSION`
* `TRAVIS_SCALA_VERSION`

下列环境变量在Objective-C构建中可用。

* `TRAVIS_XCODE_SDK`
* `TRAVIS_XCODE_SCHEME`
* `TRAVIS_XCODE_PROJECT`
* `TRAVIS_XCODE_WORKSPACE`

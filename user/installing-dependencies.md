---
title: 安装依赖
layout: en
permalink: /user/installing-dependencies/
---

一些构建不仅仅需要语言库的一个集合，他们需要默认为安装的额外的服务或者库。想了解我们构建环境的默认设置，请参考构建环境

你可以完全控制你的测试所运行于的虚拟机，因此你可以定制它来满足需求。

<div id="toc"></div>

## 安装Ubuntu包

我们的Linux环境当前基于Ubuntu 12.04 LTS。你可以安装所有的包。你可以从它们的包库，包括安全的和移植的。

<div class="note-box">
注意，这个特性在运行于<a href="/user/workers/container-based-infrastructure">基于容器的worker</a>的构建中不可用，尽管
可以使用<a href="/user/apt/">APT</a>插件。
</div>

要使用Ubuntu包，添加一些类似下面例子的东西到你的.travis.yml：

    before_install:
      - sudo apt-get update -qq
      - sudo apt-get install -y libxml2-dev

有两件事需要注意。在安装一个包之前，确保运行'apt-get update'。我们定期升级我们的构建环境来包含最新的安全补丁和升级，但是新的包定期发布会导致我们的包索引过时。我们建议在安装一个Ubuntu包之前升级来避免由于包需要升级而打破我们的构建。

第二件需要注意的事情是当运行apt-get install时`-y`参数的使用。由于你的构建运行时没有人机交互或人工干预，你应该确保不会由于apt-get需要输入而挂起。指定这个标志确保他不会像通常那样向你请求权限。

### apt-get upgrade的一句话

我们推荐你不要运行apt-get upgrade，它会升级每个apt-get发现更新版本的包。由于我们默认安装非常少的包，这将会最终下载安装高达500MB的包。

这将会显著增加构建时间，因此我们通常建议你不要在你的构建中使用它。

如果你需要升级指定的包，你可以运行一个正常的'apt-get install'，它将会安装最新可用的版本。

## 从自定义APT库安装包

你可能从一个我们构建系统并未默认配置的已经存在的库中找到一些包。你可以很容易地添加自定义库和Launchpad PPA作为你构建的一部分。

假设你需要RethinkDB作为你构建的一部分。有一个[可用的Launchpad PPA](http://www.rethinkdb.com/docs/install/ubuntu)。你可以通过添加如下一些步骤到你的.travis.yml来添加这个库：

    before_script:
      - sudo add-apt-repository ppa:rethinkdb/ppa -y
      - sudo apt-get update -q
      - sudo apt-get install rethinkdb

对于没有托管在Launchpad的库，你可能不得不将添加GnuPG key作为你配置过程的部分。

如果一个项目运行在其自己的APT库，比如[Varnish](http://varnish-cache.org)，你需要一个额外的添加库的GnuPG key的步骤。

这个例子为Ubuntu 12.04添加了Varnish 3.0的APT库到本地可用的APT资源列表，并安装`varnish`包。

    before_script:
      - curl http://repo.varnish-cache.org/debian/GPG-key.txt | sudo apt-key add -
      - echo "deb http://repo.varnish-cache.org/ubuntu/ precise varnish-3.0" | sudo tee -a /etc/apt/sources.list
      - sudo apt-get update -qq
      - sudo apt-get install varnish

## 不使用APT来安装包

对于一些项目，可能有可用的Debian/Ubuntu包，但是没有匹配的APT库。安装仍然简单，只需要额外的下载步骤。

假设你的项目需要pngquant工具来压缩PNG文件，如下是如何下载并安装.deb文件：

    before_install:
      - wget http://pngquant.org/pngquant_1.7.1-1_i386.deb
      - sudo dpkg -i pngquant_1.7.1-1_i386.deb

如果你通过这种方式安装包，确保它们在我们当前的Linux平台Ubuntu 12.04下可用。

## 从源代码安装项目

一些包可能只能从一个代码包安装。构建可能需要较新的没有可用的Ubuntu包的版本或者工具或者库。

你可以很容易地把这些构建步骤放到你的.travis.yml或者更加推荐的运行一个脚本来处理安装进程的方式。

这里是一个从一个二进制库安装CasperJS的简单的例子：

    before_script:
      - wget https://github.com/n1k0/casperjs/archive/1.0.2.tar.gz -O /tmp/casper.tar.gz
      - tar -xvf /tmp/casper.tar.gz
      - export PATH=$PATH:$PWD/casperjs-1.0.2/bin/

注意，当你要升级`$PATH`，这部分不能移动到一个shell脚本中，因为这样将会只升级运行脚本的子进程的变量。

要从源代码安装一些东西，你可以遵循类似的步骤。这里是一个下载，编译，安装protobufs库的例子。

    install:
      - wget https://protobuf.googlecode.com/files/protobuf-2.4.1.tar.gz
      - tar -xzvf protobuf-2.4.1.tar.gz
      - cd protobuf-2.4.1 && ./configure --prefix=/usr && make && make install

这个脚本可以很好地提取到一个shell脚本中，我们将它命名为`install-protobuf.sh`：

    #!/bin/sh
    set -ex
    wget https://protobuf.googlecode.com/files/protobuf-2.4.1.tar.gz
    tar -xzvf protobuf-2.4.1.tar.gz
    cd protobuf-2.4.1 && ./configure --prefix=/usr && make && sudo make install

一旦它被添加到库中，你可以从你的.travis.yml运行它：

    before_install:
      - ./install-protobuf.sh

## 安装Mac包

在我们的Mac平台上，如果需要，你拥有所有的开发工具来手动下载编译安装包。

首先你应该在[Homebrew](http://brew.sh)寻找可用的资源，因为它是已经预装随时可以使用的。

使用Homebrew而不是手动下载编译安装有一些好处。对于许多包来说，有可用的二进制包不必在安装时编译它们。即使其中某个包需要从源码编译，Homebrew也可以为你管理依赖和安装进程。使用它有助于保持你的.travis.yml最小化。

假设你需要安装beanstalk来进行你的测试，你可以在你的.travis.yml中使用下面一系列命令：

    before_install:
      - brew update
      - brew install beanstalk

注意额外的`brew update`命令，类似于`apt-get update`，来确保本地安装的Homebrew已经有最新包的索引。

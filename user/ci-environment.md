---
title: 构建环境
layout: en
permalink: /user/ci-environment/
---

### 本指南覆盖什么内容

本指南在Travis CI环境（通常引用为“命令行环境”）中可用的包、工具与设置。

<div id="toc"></div>

## 概览

Travis CI在为每次构建提供vanilla构建环境的隔离的虚拟机上运行。

这样做的好处是构建之间没有状态留存，提供一个整洁的开始并确保你的测试运行在一个下载编译安装的环境中。

构建可以访问多种数据存储与消息传送的服务，并可以安装它们运行时需要的任何东西。

## 虚拟环境

每次构建都运行在下面三个虚拟环境中的一个：

* 标准的（默认环境）
* 基于容器的（更新的环境，`sudo`命令不可用）。
* Objective-C项目使用的OS X

下表概述了虚拟环境之间的差异：

<div class="header-row header-column">
<table><thead>
<tr>
<th></th>
<th>标准的</th>
<th>基于容器的</th>
<th>OS X</th>
</tr>
</thead><tbody>
<tr>
<td>.travis.yml</td>
<td><em>默认的</em></td>
<td><code>sudo: false</code></td>
<td><code>language: objective-c</code> or <code>os: osx</code></td>
</tr>
<tr>
<td>允许<code>sudo</code>，<code>setuid</code>与<code>setgid</code></td>
<td>yes</td>
<td>no</td>
<td>N/A</td>
</tr>
<tr>
<td>启动时间</td>
<td>比基于容器的稍慢</td>
<td>比标准的稍快</td>
<td>N/A</td>
</tr>
<tr>
<td>文件系统</td>
<td>SIMFS，大小写敏感返回目录项顺序随机</td>
<td>AUFS</td>
<td>HFS+，大小写不敏感，返回目录项按字母顺序排列</td>
</tr>
<tr>
<td>缓存可用</td>
<td>仅限私有的</td>
<td>仅限公共的</td>
<td>N/A</td>
</tr>
<tr>
<td>操作系统</td>
<td>Ubuntu 12.04 LTS服务器版64位</td>
<td>Ubuntu 12.04 LTS服务器版64位</td>
<td>OS X Mavericks</td>
</tr>
<tr>
<td>内存</td>
<td>3 GB</td>
<td>3 GB</td>
<td></td>
</tr>
<tr>
<td>内核</td>
<td>高达2个，bursted</td>
<td>高达2个，bursted</td>
<td></td>
</tr>
</tbody></table>
</div>

所有的[教育优惠](https://education.travis-ci.com/)构建都适用基于容器的基础设施。

## 网络

虚拟机运行测试时IPv6是启用的。它们并没有外部IPv4地址但是完全能够与任何外部IPv4服务沟通。

IPv6栈尤其会对Java服务有一些影响，可能需要设置标志`java.net.preferIPv4Stack`来在服务显示没有启动或者无法通过网络到达时强制JVM使用IPv4栈：`-Djava.net.preferIPv4Stack=true`。

大多数服务在使用`localhost`或`127.0.0.1`与本地主机通信时正常工作。

## 所有虚拟机映像的通用环境

### 版本控制

所有虚拟机映像都预装了下列工具

 * 来自[git-core PPA](https://launchpad.net/~git-core/+archive/ubuntu/v1.8)的Git 1.8发布版
 * Mercurial（官方Ubuntu包）
 * Subversion（官方Ubuntu包）


### 编译器与构建工具链

GCC，Clang，make，autotools，cmake，scons。


### 网络工具

curl，wget，OpenSSL，rsync


### Go

Go编译器/构建工具。

### 运行时

每个worker都有至少一个版本的

* Ruby
* OpenJDK
* Python
* Node.js
* Go编译器/构建工具

来适应在构建时可能需要的这些运行时之一的项目。

特定语言的worker有各个语言的多个运行时（例如Ruby worker有大约10个Ruby版本/实现）。

### 数据存储

* MySQL
* PostgreSQL
* SQLite
* MongoDB
* Redis
* Riak
* Apache Cassandra
* Neo4J Community Edition
* ElasticSearch
* CouchDB

### Firefox

所有的虚拟机安装了最新版的Firefox，当前Linux环境是31.0，OS X是25.0。

如果你需要指定版本的Firefox，在构建的`before_install`阶段使用Firefox插件来安装它。

例如，要安装17.0版本，添加如下内容到你的`.travis.yml`文件：

    addons:
      firefox: "17.0"

请注意插件只能在64位的Linux环境下工作。

### 消息传送技术

* [RabbitMQ](http://rabbitmq.com)
* [ZeroMQ](http://zeromq.org/)

### Headless浏览器与测试工具

* [xvfb](http://en.wikipedia.org/wiki/Xvfb) (X Virtual Framebuffer)
* [PhantomJS](http://phantomjs.org/)

### 环境变量

在每个构建环境有[默认环境变量列表](/user/environment-variables#Default-Environment-Variables)可用。

### 库

* OpenSSL
* ImageMagick

### apt配置

apt使用`DEBIAN_FRONTEND`环境变量与apt配置文件，配置为不需要确认（假设默认使用-y开关）。这意味着`apt-get install -qq`可以使用而不加-y标记。

### 组成员

用户执行属于从用户派生的一个主要组（`$USER`）的构建。如果你的项目需要额外成员身份来运行构建，依照这些步骤：

1. 设置环境。这可以在构建生命周期内构建脚本执行前的任何时间完成。
    - 设置并导出环境变量。
    - 添加`$USER`到期望的次级组：`sudo usermod -a -G SECONDARY_GROUP_1,SECONDARY_GROUP_2 $USER`。你可能使用`-g`修改用户的主要组。
1. 你的`script`看起来像是：

```bash
script: sudo -E su $USER -c 'COMMAND1; COMMAND2; COMMAND3'
```

这将会把环境变量向下传递到一个以`$USER`运行的`bash`进程，同时保持环境变量已经定义且属于上文`usermod`中指出的次级组。

### 构建系统信息

在构建日志中，相关软件版本（包括可用语言版本）显示在“构建系统信息”中。

## Go虚拟机映像

下面的别名是可用的，并推荐使用从而在映像升级时将阻力减到最小：

* `go1`, `go1.0` → 1.0.3
* `go1.1` → 1.1.2
* `go1.2` → 1.2.2

## JVM（Clojure, Groovy, Java, Scala）虚拟机映像

### JDK

* Oracle JDK 7 (oraclejdk7)
* OpenJDK 7 (openjdk7)
* OpenJDK 6 (openjdk6)
* Oracle JDK 8 (oraclejdk8)

OracleJDK 7是默认的，相比于Ubuntu库的OpenJDK 7，我们的补丁等级要新很多。Sun/Oracle JDK 6并不提供，因为它在2012年秋季已经到达生命尽头。

如果你在JVM映像选择了`jdk`，那么`$JAVA_HOME`将被正确设置。

### Maven版本

使用Apache Maven 3.2.x。Maven被配置为使用主要的与http://maven.travis-ci.org的oss.sonatype.org镜像。

### Leiningen版本

travis-ci.org包含了标准的（“uberjar”）位于`/usr/local/bin/lein1`的Leiningen 1.7.x和位于`/usr/local/bin/lein2`的Leiningen 2.4.x。
默认是2.4.x；`/usr/local/bin/lein`是到`/usr/local/bin/lein2`的符号连接。

### SBT版本

因为有了强大的[sbt-extras](https://github.com/paulp/sbt-extras)可供使用，travis-ci.org潜在提供了任何版本的Simple Build Tool（sbt或SBT）。为了减少构建时间，sbt的最常用版本已经预装（例如0.13.5或0.12.4），但是`sbt`命令可以动态探测和安装你的Scala项目需要的版本。

### Gradle版本

Gradle 2.0。

## Erlang虚拟机影响

### Erlang/OTP发布版

Erlang/OTP发布使用[kerl](https://github.com/spawngrid/kerl)构建。


### Rebar

travis-ci.org提供了最新版本的Rebar。如果一个代码库的`./rebar`（项目根目录）下有rebar二进制打包，那么将会使用它为非预先提供的版本。

## Node.js虚拟机映像

### Node.js版本

Node运行时使用[nvm](https://github.com/creationix/nvm)构建。

### SCons

Scons

## Haskell虚拟机映像

### Haskell平台版本

[Haskell平台](http://hackage.haskell.org/platform/contents.html) 2012.02与GHC 7.0，7.4，7.6与7.8。


## Perl虚拟机映像

### Perl版本

Perl版本是通过[Perlbrew](http://perlbrew.pl/)安装的。这些以`-extras`后缀结束的运行时使用`-Duseshrplib`与`-Duseithreads`标志编译。这些也有带`-shrplib`后缀的别名。

### 预装的模块

cpanm (App::cpanminus)
Dist::Zilla
Dist::Zilla::Plugin::Bootstrap::lib
ExtUtils::MakeMaker
LWP
Module::Install
Moose
Test::Exception
Test::Kwalitee
Test::Most
Test::Pod
Test::Pod::Coverage

## PHP虚拟机映像

### PHP版本

PHP运行时使用[php-build](https://github.com/CHH/php-build)构建。

[hhvm](https://github.com/facebook/hhvm)可用。并且nighly构建按需安装（作为`hhvm-nightly`）。

### XDebug

已经支持。

### 扩展

    [PHP Modules]
    bcmath
    bz2
    Core
    ctype
    curl
    date
    dom
    ereg
    exif
    fileinfo
    filter
    ftp
    gd
    gettext
    hash
    iconv
    intl
    json
    libxml
    mbstring
    mcrypt
    mysql
    mysqli
    mysqlnd
    openssl
    pcntl
    pcre
    PDO
    pdo_mysql
    pdo_pgsql
    pdo_sqlite
    pgsql
    Phar
    posix
    readline
    Reflection
    session
    shmop
    SimpleXML
    soap
    sockets
    SPL
    sqlite3
    standard
    sysvsem
    sysvshm
    tidy
    tokenizer
    xdebug
    xml
    xmlreader
    xmlrpc
    xmlwriter
    xsl
    zip
    zlib

    [Zend Modules]
    Xdebug

## Python虚拟机映像

### Python版本

每个Python都有一个来自`pip`与`distribute`的隔离的虚拟环境，并在构建运行前激活。

*不支持*Python 2.4与Jython，并且将来也没有计划支持它们。

### 预装的pip包

* nose
* py.test
* mock
* wheel

除了pypy与pypy3的所有版本也安装了`numpy`。

## Ruby（也就是通用的）虚拟机映像

### Ruby版本/实现

[travis-ci.org不再提供Ruby 1.8.6与1.9.1](https://twitter.com/travisci/status/114926454122364928).

Ruby使用[RVM](http://rvm.io/)构建.RVM为每个用户安装，并且由`~/.bashrc`引入。

可以根据需要按需安装其他版本的RVM。例如要测试Rubinius 2.2.1，你可以使用`rbx-2.2.1`，RVM将会按需下载二进制。

### Bundler版本

目前是1.7.x版本（通常是最新的）

### 全局gem集合中的gem

* bundler
* rake


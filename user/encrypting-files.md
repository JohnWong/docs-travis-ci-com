---
title: 加密文件
layout: en
permalink: /user/encrypting-files/
---

**请注意加密文件在[来自fork的pull request](/user/pull-requests#Security-Restrictions-when-testing-Pull-Requests)中不可用。**

<div id="toc"></div>

## 准备

要依照本指南的例子，你需要安装Travis CI [命令行客户端](https://github.com/travis-ci/travis.rb#readme)：

    $ gem install travis

确保你已经[登录](https://github.com/travis-ci/travis.rb#login):

    $ travis login

如果你在使用Travis Pro，你将不得不使用`--pro`标志来登录：

    $ travis login --pro

更多信息参考其[安装指南](https://github.com/travis-ci/travis.rb#installation)。

## 自动加密

假设：

* 库在Travis CI设置过
* 你安装了**1.7.0**或更新版本的Travis CI命令行客户端并设置好了（你已经登陆）
* 你有一个库的本地拷贝，并且在这份拷贝的工作目录下打开一个终端
* 在库中有一个称作super_secret.txt文件，你在Travis CI中需要但并不希望发布其内容到GitHub。

`travis encrypt-file`命令将会使用对称加密（AES-256）来加密一个文件，并切存储secret到一个安全变量。它将会输出你在构建脚本中可以使用命令来解密文件。

{% highlight console %}
$ travis encrypt-file super_secret.txt
encrypting super_secret.txt for rkh/travis-encrypt-file-example
storing result as super_secret.txt.enc
storing secure env variables for decryption

Please add the following to your build scirpt (before_install stage in your .travis.yml, for instance):

    openssl aes-256-cbc -K $encrypted_0a6446eb3ae3_key -iv $encrypted_0a6446eb3ae3_key -in super_secret.txt.enc -out super_secret.txt -d

Pro Tip: You can add it automatically by running with --add.

Make sure to add super_secret.txt.enc to the git repository.
Make sure not to add super_secret.txt to the git repository.
Commit all changes to your .travis.yml.
{% endhighlight %}

你也可以使用`--add`来让它自动添加解密命令道你的`.travis.yml`。

{% highlight console %}
$ travis encrypt-file super_secret.txt --add
encrypting super_secret.txt for rkh/travis-encrypt-file-example
storing result as super_secret.txt.enc
storing secure env variables for decryption

Make sure to add super_secret.txt.enc to the git repository.
Make sure not to add super_secret.txt to the git repository.
Commit all changes to your .travis.yml.
{% endhighlight %}

### 加密多个文件

注意这个方法[只对一个文件有效](https://github.com/travis-ci/travis.rb/issues/239).

如果你需要加密多个文件，你需要创建一个敏感文件的压缩包，然后在构建时解密它并解压缩。

假设我们有敏感文件`foo`与`bar`。

{% highlight console %}
$ tar cvf secrets.tar foo bar
$ travis encrypt-file secrets.tar
$ vi .travis.yml
$ git add secrets.tar.enc .travis.yml
$ git commit -m 'use secret archive'
$ git push
{% endhighlight %}

{% highlight yaml %}
before_install:
  - openssl aes-256-cbc -K $encrypted_5880cf525281_key -iv $encrypted_5880cf525281_iv -in secrets.tar.enc -out secrets.tar -d
  - tar xvf secrets.tar
{% endhighlight %}

（按照你的需要调整`$*_key`与`$*_iv`）

### 警告

有报告指出这个功能在本地Windows机器上并不奏效，请在Linux或者OS X机器上使用。

## 手动加密

假设：

* 库在Travis CI设置过
* 你安装了最新版本的Travis CI命令行客户端并设置好了（你已经登陆）
* 你有一个库的本地拷贝，并且在这份拷贝的工作目录下打开一个终端
* 在库中有一个称作super_secret.txt文件，你在Travis CI中需要但并不希望发布其内容到GitHub。

文件可能太大而无法通过`travis encrypt`命令来直接加密它。胆识你可以使用一个密码来加密文件，然后加密密码。在Travis CI中，你可以使用密码来再次解密文件。

建立的进程像这样：

1. **提出一个密码。**首先你需要一个密码。我们推荐使用一个工具比如pwgen或者1password来生成一个随机密码。在我们的例子中，使用`ahduQu9ushou0Roh`。
2. **加密你的密码并添加到你的.travis.yml。** 这里我们可以使用`encrypt`命令：`travis encrypt super_secret_password=ahduQu9ushou0Roh --add`。注意如果你为多个文件这样设置多次，你将不得不使用不同的变量名，这样密码不会互相覆盖。
3. **在本地加密文件。** 使用你在本地安装并且Travis CI上也安装的一个工具（如下）。
4. **设置解密命令。**你应该在你的`.travis.yml`中的`before_install`部分添加命令来解密文件（如下）。

确保添加`super_secret.txt`到你的`.gitignore`列表，并同时提交加密的文件与你的`.travis.yml`的变更。

### 使用GPG

建立：

{% highlight console %}
$ travis encrypt super_secret_password=ahduQu9ushou0Roh --add
$ gpg -c super_secret.txt
(will prompt you for the password twice, use the same value as for super_secret_password above)
{% endhighlight %}

`.travis.yml`的内容（除了在那里你需要的其他东西）：

{% highlight yaml %}
env:
  global:
    secure: ... encoded secret ...
before_install:
  - echo $super_secret_password | gpg super_secret.txt.gpg
{% endhighlight %}

加密后的文件是`super_secret.txt.gpg`，需要提交到库中。

#### 使用OpenSSL


建立：

{% highlight console %}
$ travis encrypt super_secret_password=ahduQu9ushou0Roh --add
$ openssl aes-256-cbc -k "ahduQu9ushou0Roh" -in super_secret.txt -out super_secret.txt.enc
(keep in mind to replace the password with the proper value)
{% endhighlight %}

`.travis.yml`的内容（除了在那里你需要的其他东西）：

{% highlight yaml %}
env:
  global:
    secure: ... encoded secret ...
before_install:
  - openssl aes-256-cbc -k "$super_secret_password" -in super_secret.txt.enc -out super_secret.txt -d
{% endhighlight %}

加密后的文件时`super_secret.txt.enc`，需要提交到库中。
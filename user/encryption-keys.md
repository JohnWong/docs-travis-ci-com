---
title: 加密密钥
layout: en
permalink: /user/encryption-keys/
---

**我们有独立的[加密文件](/user/encrypting-files/)的文档。**

Travis CI生成一组公有和私有RSA密钥，可以用来加密你放在`.travis.yml`文件中的信息，并仍然保持私有。目前我们允许加密[环境变量](/user/environment-variables/)，通知设置和部署api密钥。

**请注意加密的环境变量在[来自fork的pull request](/user/pull-requests#Security-Restrictions-when-testing-Pull-Requests)中不可用**

## 使用

用公用密钥加密的最简单的方法是使用Travis CLI。这个工具使用Ruby编写并作为gem发布。首先你需要安装这个gem：

    gem install travis

然后你可以使用`encrypt`命令来加密数据（这个例子假设你在你的项目目录运行命令。如果不是，添加`-r owner/project`）。

    travis encrypt SOMEVAR=secretvalue

这将会输出一个字符串，看起来类似：

    secure: ".... encrypted data ...."

现在你可以将它放在`.travis.yml`文件中。

请注意环境变量的名称和值都编码为"travis encrypt."所产生的字符串。你必须添加这一条到"secure"键（在"env"键下面）中。这使得值为"secretvalue"的环境变量SOMEVAR在你的程序中可用。

你可能添加多条到你的.travis.yml的"secure."键中。它们在你的程序中都可用。

加密值可以在[构建矩阵的安全环境变量](/user/environment-variables#In-the-.travis.yml)与[通知](/user/notifications)中使用。

### 注意转义某些符号

当你使用`travis encrypt`来加密敏感数据，注意它将会作为一个`bash`语句处理是重要的。这意味着你在加密的秘密在`bash`解析它时不应该引起错误。

数据不完整会引起`bash`产生错误语句到日志中，包含你的敏感数据的一部分。

因此你需要转义符号，比如括号、反斜杠与竖杠符号。例如当你希望为`FOO`赋值字符串`6&a(5!1Ab\`，你需要执行：

    travis encrypt "FOO=6\\&a\\(5\\!1Ab\\\\"

`travis`加密字符串`FOO=6\&a\(5\!1Ab\\`然后`bash`在构建环境中使用来评估。

相当于你可以做

    travis encrypt 'FOO=6\&a\(5\!1AB\\'

### 通知示例

我们希望添加campfire通知到我们的.travis.yml文件，但是我们并不希望公开暴露我们的API token。

条目应该是这样的格式：

    notifications:
      campfire: [subdomain]:[api token]@[room id]

对于我们，那是somedomain:abcxyz@14。

我们加密这个字符串

    travis encrypt somedomain:abcxyz@14

就会产生一些像这样的东西

    Please add the following to your .travis.yml file:

      secure: "ABC5OwLpwB7L6Ca...."

我们添加到我们的.travis.yml文件中

    notifications:
      campfire:
        rooms:
          secure: "ABC5OwLpwB7L6Ca...."

And we're done.

### 详细讨论

安全变量系统从（解析的YAML）配置中获取```{ 'secure' => 'encrypted string' }```形式的值病将其替换为机密后的字符串。

因此

    notifications:
      campfire:
        rooms:
          secure: "encrypted string"

变成

    notifications:
      campfire:
        rooms: "decrypted string"

而

    notifications:
      campfire:
        rooms:
          - secure: "encrypted string"

变成

    notifications:
      campfire:
        rooms:
          - "decrypted string"

在安全环境变量的情况下

    env:
      - secure: "encrypted string"

变成

    env:
      - "decrypted string"

## 获取你的库的公共密钥

你可以使用Travis API获取公共密钥，使用`/repos/:owner/:name/key`或者`/repos/:id/key` endpoints，例如：

    https://api.travis-ci.org/repos/travis-ci/travis-ci/key

你也可以使用`travis`工具来获取上述密钥：

    travis pubkey

或者如果你不在你项目的目录下：

    travis pubkey -r owner/project

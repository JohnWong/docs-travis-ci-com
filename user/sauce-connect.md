---
title: 在Travis CI中使用Sauce Labs
layout: en
permalink: /user/sauce-connect/
---
Travis CI集成了[Sauce Labs](https://saucelabs.com)，一个浏览器与移动测试平台。例如它很好地集成了Selenium。

集成自动地设置了一个开始测试所需的通道。为了这个目的使用了Sauce Connect。

注意犹豫安全限制，Sauce Labs插件在pull request的构建中不可用。禁用的详细原因参见[pull request页面](http://docs.travis-ci.com/user/pull-requests/#Security-Restrictions-when-testing-Pull-Requests)

## 建立Sauce Connect

[Sauce Connect][sauce-connect]安全地代理Sauce
Labs的基于云的虚拟机与你的本地服务器之间的流量。Connect使用端口443与80与Sauce的云通信。如果你为你的Selenium测试而使用Sauce Labs，这使得连接你的服务器容易许多。

[sauce-connect]: https://saucelabs.com/connect

首先，如果没有注册过的话，去[注册][sauce-sign-up]（对开源工程[免费][open-sauce]），从你的[账户页][sauce-account]得到你的访问密钥。一旦你有了它，添加到你的.travis.yml文件：

    addons:
      sauce_connect:
        username: "Your Sauce Labs username"
        access_key: "Your Sauce Labs access key"

[sauce-sign-up]: https://saucelabs.com/signup/plan/free
[sauce-account]: https://saucelabs.com/account
[open-sauce]: https://saucelabs.com/signup/plan/OSS

如果你不希望你的访问密钥在你的库中公开可用，你可以通过`travis encrypt "your-access-key"`加密它（参见[加密密钥][encryption-keys]），并像这样添加安全字符串：

    addons:
      sauce_connect:
        username: "Your Sauce Labs username"
        access_key:
          secure: "The secure string output by `travis encrypt`"

如果你分别将它们命名为`SAUCE_USERNAME`与`SAUCE_ACCESS_KEY`，你也可以将`username`与`access_key`作为环境变量。此时你需要添加到你的.travis.yml中的所有内容是这样：

    addons:
      sauce_connect: true

[encryption-keys]: http://docs.travis-ci.com/user/encryption-keys/

要允许多个通道同时打开，Travis CI打开一个Sauce Connect的[认证通道][identified-tunnels]。确保在你打开到Sauce Labs的selenium grid的连接时发送`TRAVIS_JOB_NUMBER`环境变量，作为所需能力的`tunnel-identifier`，否则将无法连接到虚拟机上运行的服务器。

[identified-tunnels]: https://saucelabs.com/connect#tunnel-identifier

这看起来的样子取决于你在使用的客户端库，在Ruby的[selenium-webdriver][ruby-bindings]绑定：

    caps = Selenium::WebDriver::Remote::Capabilities.firefox({
      'tunnel-identifier' => ENV['TRAVIS_JOB_NUMBER']
    })
    driver = Selenium::WebDriver.for(:remote, {
      url: 'http://username:access_key@ondemand.saucelabs.com/wd/hub',
      desired_capabilities: caps
    })

[ruby-bindings]: https://code.google.com/p/selenium/wiki/RubyBindings

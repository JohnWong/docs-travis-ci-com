---
title: GUI与headless浏览器测试
layout: en
permalink: /user/gui-and-headless-browsers/
---

## 本指南覆盖什么

本指南覆盖使用Travis[命令行环境](/user/ci-environment/)所提供的工具进行headless（headless意思是无用户图形界面）用户图形界面与浏览器测试。大多数内容是技术中立的，并不覆盖特定测试工具（比如Poltergeist与Capybara）的所有细节。我们推荐你在阅读本章前先阅读[准备开始](/user/getting-started/)与[定制构建](/user/build-configuration/)指南。

## 使用Sauce Labs

[Sauce Labs](https://saucelabs.com)提供了一个Selenium云，可以访问超过170个不同设备/操作系统/浏览器的组合。如果你有使用Selenium的浏览器测试，使用Sauce Labs来运行测试非常简单。首先，你需要注册他们的服务（对开源项目免费）。

一旦你注册成功，你需要使用Sauce Connect来建立一个隧道，这样Sauce Labs可以连接到你的web服务器。我们的[Sauce Connect插件](/user/sauce-connect)使得这件事更简单，只需要添加下面内容到你的.travis.yml：

    addons:
      sauce_connect:
        username: "Your Sauce Labs username"
        access_key: "Your Sauce Labs access key"

如果希望的话，你可以[加密你的访问密钥](/user/encryption-keys/)。

现在Sauce Labs有了一个到达你服务器的方式，但是你仍然需要启动它。参见下面的[启动web服务器](#Starting-a-Web-Server)来获取更多怎样做的信息。

最后，你需要配置你的Selenium测试来运行在Sauce Labs上而不是本地。可以通过使用[Remote WebDriver](https://code.google.com/p/selenium/wiki/RemoteWebDriver)来完成。确切的代码依赖于你所使用的工具/平台，但是对于Python来说将会类似这样：

    username = os.environ["SAUCE_USERNAME"]
    access_key = os.environ["SAUCE_ACCESS_KEY"]
    capabilities["tunnel-identifier"] = os.environ["TRAVIS_JOB_NUMBER"]
    hub_url = "%s:%s@localhost:4445" % (username, access_key)
    driver = webdriver.Remote(desired_capabilities=capabilities, command_executor="http://%s/wd/hub" % hub_url)

Sauce Connect插件导出环境变量 `SAUCE_USERNAME` 与 `SAUCE_ACCESS_KEY`，并将hub URL传递回Sauce Labs。

这就是你将Selenium测试运行在Sauce Labs所需做的所有事情。但是你可能希望只将Sauce Labs用于Travis CI构建，而非本地构建。要这么做，你需要使用 `CI` 或者 `TRAVIS` 环境变量来一句条件改变你所使用的驱动（参见[我们的可用环境变量列表](/user/ci-environment/#Environment-variables)来找寻更多的探测是否运行在Travis CI上的方式）。

要让Sauce Labs上的测试结果更容易浏览，你可能希望随着构建提供更多元数据。你可以通过传入更多期望的能力来完成这件事：

    capabilities["build"] = os.environ["TRAVIS_BUILD_NUMBER"]
    capabilities["tags"] = [os.environ["TRAVIS_PYTHON_VERSION"], "CI"]

对于我们自己的travis-web，我们使用Sauce Labs在不同的浏览器里运行浏览器测试。我们使用我们[.travis.yml](https://github.com/travis-ci/travis-web/blob/15dc5ff92184db7044f0ce3aa451e57aea58ee19/.travis.yml#L14-15)中的环境变量来将构建拆分为不同的工作，然后使用[期望的能力](https://github.com/travis-ci/travis-web/blob/15dc5ff92184db7044f0ce3aa451e57aea58ee19/script/saucelabs.rb#L9-13)将期望的浏览器传入Sauce Labs。在Travis CI网站，最终像[这样](https://travis-ci.org/travis-ci/travis-web/builds/12857641)。

## 使用xvfb来运行需要用户图形界面的测试（例如一个web浏览器）

你可以在Travis CI中运行需要用户图形界面（例如一个web浏览器）的测试套件。环境安装了`xvfb` (X Virtual Framebuffer)于Firefox。大致就是`xvfb`模拟一个显示器，让你可以在一个headless机器上运行一个真实的用户图形界面程序或者web浏览器，就像连接了一个合适的显示器。

在`xvfb`可以使用之前，需要启动它。一个做这件事典型的最佳位置是`before_install`，比如：

    before_install:
      - "export DISPLAY=:99.0"
      - "sh -e /etc/init.d/xvfb start"

这将会在显示端口:99.0启动`xvfb`。显示端口在 `/etc/init.d` 脚本中直接设置。其次当你运行你的测试时，你需要告诉你的测试工具进程（比如Selenium）那个显示端口，这样它知道在哪里启动Firefox。这在不同的测试工具与编程语言间可能变化。

### 配置xvfb屏幕尺寸及更多

设置xvfb屏幕尺寸与像素深度是有可能的。由于xvfb是一个虚拟屏幕，它可以虚拟地模拟出任何分辨率。当这么做时，你需要直接启动xvfb或者通过`start-stop-daemon`工具启动，不要通过init脚本启动：

    before_install:
      - "/sbin/start-stop-daemon --start --quiet --pidfile /tmp/custom_xvfb_99.pid --make-pidfile --background --exec /usr/bin/Xvfb -- :99 -ac -screen 0 1280x1024x16"

在上面的例子中，我们设置分辨率为 `1280x1024x16`。

更多信息参见[xvfb手册](http://www.xfree86.org/4.0.1/Xvfb.1.html)。

## 启动一个web服务器

如果你的项目需要一个web应用运行着来进行测试，你需要在运行测试之前启动一个。通常使用Ruby，Node.js，基于JVM的web服务器来提供用来运行测试套件的HTML页面服务。由于每个travis-ci.org虚拟机提供至少一个版本的Ruby，Node.js和OpenJDK，你可以依赖三个选项中的一个。

添加一个 `before_script` 来启动一个服务器，例如：

    before_script:
      - "export DISPLAY=:99.0"
      - "sh -e /etc/init.d/xvfb start"
      - sleep 3 # give xvfb some time to start
      - rackup  # start a Web server
      - sleep 3 # give Web server some time to bind to sockets, etc

如果你需要web服务器在80端口监听，记得使用 `sudo` （Linux不允许未授权进程绑定到80端口）。对于超过1024的端口号，不必使用`sudo`（而且也不推荐）。


<div class="note-box">
注意<code>sudo</code>在运行在<a href="/user/workers/container-based-infrastructure">基于容器的worker</a>上的构建中不可用。
</div>


## 使用PhantomJS

[PhantomJS](http://phantomjs.org/)是一个提供JavaScript API的headless WebKit。这是一个快速headless测试，网站抓取，网页快照，SVG渲染与网络监测及其他许多其他用例的最佳解决方案。

[命令行环境](/user/ci-environment/)预装了PhantomJS（在`phantomjs`目录下可用；不要依赖于精确位置）。由于它是完全headless，并不需要运行`xvfb`。

一个非常简单的例子：

    script: phantomjs testrunner.js

如果你需要一个web服务器来为测试服务，参考前一节。

## 示例

### 真实世界的项目

 * [Ember.js](https://github.com/emberjs/ember.js/blob/master/.travis.yml) (starts web server programmatically)
 * [Sproutcore](https://github.com/sproutcore/sproutcore/blob/master/.travis.yml) (starts web server with *before_script*)

### Ruby

#### RSpec, Jasmine, Cucumber

这是一个运行Rspec、Jasmine与Cucumber测试的示例rake任务：

    task :travis do
      ["rspec spec", "rake jasmine:ci", "rake cucumber"].each do |cmd|
        puts "Starting to run #{cmd}..."
        system("export DISPLAY=:99.0 && bundle exec #{cmd}")
        raise "#{cmd} failed!" unless $?.exitstatus == 0
      end
    end

在这个例子中，Jasmine与Cucumber都需要显示端口，因为它们都适用真实浏览器。Rspec运行不需要它，但是设置了也没有坏处。

## 故障排除

### Selenium与Firefox弹窗

如果你的测试套件操作一个模态弹窗，例如[到另一个位置的重定向](https://support.mozilla.org/en-US/questions/792131)，那么你需要添加一个自定义配置，这样就禁止弹窗了。

这可以通过应用一个选项关闭的自定义的Firefox配置：（例如在Ruby下使用Capybara）

    Capybara.register_driver :selenium do |app|

      custom_profile = Selenium::WebDriver::Firefox::Profile.new

      # Turn off the super annoying popup!
      custom_profile["network.http.prompt-temp-redirect"] = false

      Capybara::Selenium::Driver.new(app, :browser => :firefox, :profile => custom_profile)
    end

### Karma与Firefox静止超时

当使用Karma与Firefox测试时，你可能遇到浏览器静止超时引起的构建错误。遇到这个问题时Karma输出一个错误类似：

    WARN [Firefox 31.0.0 (Linux)]: Disconnected (1 times), because no message in 10000 ms.

在这种情况下，你应该在 `karma.conf.js` 中增加浏览器静止超时到一个更高的值，例如：

    browserNoActivityTimeout: 30000,

更多信息参考Karma[配置文件](https://karma-runner.github.io/0.12/config/configuration-file.html)的文档。

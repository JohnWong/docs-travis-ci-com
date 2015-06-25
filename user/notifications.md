---
title: 配置构建提醒
layout: en
permalink: /user/notifications/
---

<div id="toc"></div>

## 提醒

Travis CI可以通过邮件、IRC或者webhook来通知你构建结果。

如果提交者或者提交的作者是库的成员（也就是他们有公共库的推送或者管理权限，或者有私有库的拉取、推送或者管理权限），那么默认会发送邮件通知给他们。

并且在给定的分支上在下面的情况下默认发送邮件：

* 一个构建刚刚中断或者仍然中断
* 一个之前中断的构建刚修复

你可以使用下面的选项来改变这一行为：

> 注意：括号中的条目是占位符。括号应该省略。

### 注意SSL/TLS Ciphers

当通过SSL/TLS发送通知时，注意接收服务器所接受的cipher。如果服务器没有一个cipher生效，那么通知将会失败。

目前下列cipher（在[jruby-openssl gem](https://rubygems.org/gems/jruby-openssl)中定义的）已知会奏效：

AES-128 AES-128-CBC AES-128-CFB AES-128-CFB1 AES-128-CFB8 AES-128-ECB AES-128-OFB AES-192 AES-192-CBC AES-192-CFB AES-192-CFB1 AES-192-CFB8 AES-192-ECB AES-192-OFB AES-256 AES-256-CBC AES-256-CFB AES-256-CFB1 AES-256-CFB8 AES-256-ECB AES-256-OFB BF BF-CBC BF-CFB BF-CFB1 BF-CFB8 BF-ECB BF-OFB BLOWFISH CAST CAST-CBC CAST5 CAST5-CBC CAST5-CFB CAST5-CFB1 CAST5-CFB8 CAST5-ECB CAST5-OFB DES DES-CBC DES-CFB DES-CFB1 DES-CFB8 DES-ECB DES-EDE DES-EDE-CBC DES-EDE-CFB DES-EDE-CFB1 DES-EDE-CFB8 DES-EDE-ECB DES-EDE-OFB DES-EDE3 DES-EDE3-CBC DES-EDE3-CFB DES-EDE3-CFB1 DES-EDE3-CFB8 DES-EDE3-ECB DES-EDE3-OFB DES-OFB RC2 RC2-40-CBC RC2-64-CBC RC2-CBC RC2-CFB RC2-CFB1 RC2-CFB8 RC2-ECB RC2-OFB RC4 RC4-40

也请查阅[cipher套件名称映射](https://www.openssl.org/docs/apps/ciphers.html)。

如果上面列出的cipher没有一个生效，请打开一个[GitHub issue](https://github.com/travis-ci/travis-ci/issues)。

## 电子邮件通知

你可以指定会收到构建结果通知的收件人，例如：

    notifications:
      email:
        - one@example.com
        - other@example.com

你可以完全关掉电子邮件通知：

    notifications:
      email: false

你也可以指定你什么时候希望收到通知：

    notifications:
      email:
        recipients:
          - one@example.com
          - other@example.com
        on_success: [always|never|change] # default: change
        on_failure: [always|never|change] # default: always

`always`与`never`意味着你希望电子邮件通知总是或者从不发送。`change`意味着在给定分支的构建状态变更时你将会知道。

### 如何确定构建电子邮件接收者？

构建电子邮件默认会发送给提交者或者作者，但是只有在他们可以访问提交推送到的库时才成立。这会阻止Travis CI上fork活动在他们推送任何upstream变更到他们的fork上时提醒upstream库的作者。这也阻止构建通知不会发到没有在Travis CI上注册的fork上。

电子邮件地址通过提交中的电子邮件地址来确定，但是只有它匹配我们数据库中的电子邮件之一。我们从GitHub同步你所有的电子邮件地址，只是为了构建通知。

默认值可以如上所示在`.travis.yml`中覆盖。如果指定了一个设置，那么Travis CI只给指定的地址发送电子邮件，而不会发给提交者或者作者。

### 我们如何为构建通知修改电子邮件地址？

电子邮件地址由GitHub拉取。你的用户账号的所有注册的电子邮件在Travis CI中也都可用。

你可以通过为指定的库设置一个不同的电子邮件地址来修改构建电子邮件地址。运行`git config user.email my@email.com`来为你的库设置一个不同的电子邮件地址而不是使用默认值。

注意我们目前并不遵守GitHub的[详细通知设置](https://github.com/settings/notifications)，因为目前它们并没有通过一个API暴露出来。

### 我并未收到任何构建通知

没有收到构建通知的最常见的原因，除了在Travis CI上有一个用户账户外，就是电子邮件地址并没有在GitHub上注册并验证。参考上面的如何修改电子邮件地址来修改为已注册的或者确保添加在这个库所使用的地址到GitHub上[你的已验证电子邮件地址](https://github.com/settings/emails)。

## IRC通知

你也可以指定发送通知到一个IRC通道：fy notifications sent to an IRC channel:

    notifications:
      irc: "chat.freenode.net#my-channel"

或者多个通道：

    notifications:
      irc:
        - "chat.freenode.net#my-channel"
        - "chat.freenode.net#some-other-channel"

你也可以通过其它通知类型来指定什么时候会发送IRC通知：

    notifications:
      irc:
        channels:
          - "chat.freenode.net#my-channel"
          - "chat.freenode.net#some-other-channel"
        on_success: [always|never|change] # default: always
        on_failure: [always|never|change] # default: always

你也可以通过一个模版来定制将会发送到通道的消息：

    notifications:
      irc:
        channels:
          - "chat.freenode.net#my-channel"
          - "chat.freenode.net#some-other-channel"
        template:
          - "%{repository} (%{commit}) : %{message} %{foo} "
          - "Build details: %{build_url}"

你也可以插入如下变量：

* *repository_slug*: your GitHub repo identifier (like ```svenfuchs/minimal```)
* *repository_name*: the slug without the username
* *repository*: same as repository_slug [Deprecated]
* *build_number*: build number
* *build_id*: build id
* *branch*: branch build name
* *commit*: shortened commit SHA
* *author*: commit author name
* *commit_message*: commit message of build
* *result*: result of build
* *message*: travis message to the build
* *duration*: duration of the build
* *compare_url*: commit change view URL
* *build_url*: URL of the build detail

默认模版是：

    notifications:
      irc:
        template:
          - "%{repository}#%{build_number} (%{branch} - %{commit} : %{author}): %{message}"
          - "Change view : %{compare_url}"
          - "Build details : %{build_url}"

如果你希望机器人使用提醒而非常规消息，可以使用`use_notice`标识：

    notifications:
      irc:
        channels:
          - "chat.freenode.net#my-channel"
          - "chat.freenode.net#some-other-channel"
        on_success: [always|never|change] # default: always
        on_failure: [always|never|change] # default: always
        use_notice: true

如果你希望机器人在发送消息前不加入，稍后才加入。使用`skip_join`标识：

    notifications:
      irc:
        channels:
          - "chat.freenode.net#my-channel"
          - "chat.freenode.net#some-other-channel"
        on_success: [always|never|change] # default: always
        on_failure: [always|never|change] # default: always
        use_notice: true
        skip_join: true

如果你启用`skip_join`，记得移除IRC通道的机器人提醒的`NO_EXTERNAL_MSGS`标识。

如果你希望机器人发送通道密钥保护的消息到通道（例如设置`/mode #channel +k password`），你可以使用`channel_key`变量：

    notifications:
      irc:
        channels:
          - "irc.freenode.org#my-channel"
        channel_key: 'password'

## Campfire通知

通知也可以通过Campfire的聊天室发送，使用如下格式：

    notifications:
      campfire: [subdomain]:[api token]@[room id]


* *子域*：是你的campfire子域（例如访问'https://your-subdomain.campfirenow.com'时是'your-subdomain'）
* *api token*：你希望用来发送通知的账号的token。
* *room id*：是房间id，而不是名称。

> 注意：如果你的.travis.yml存储在公共库中，我们强烈推荐你[加密](/user/encryption-keys/)这个值：

    travis encrypt subdomain:api_token@room_id --add notifications.campfire.rooms

你也可以定制通知，比如IRC通知：

    notifications:
      campfire:
        rooms:
          - [subdomain]:[api token]@[room id]
        template:
          - "%{repository} (%{commit}) : %{message} %{foo} "
          - "Build details: %{build_url}"

其它标识，比如`on_success`与`on_failure`像在IRC通知配置中一样有效。

## Flowdock通知

通知可以通过使用如下格式发送到你的Flowdock Team Inbox收件箱：

    notifications:
      flowdock: [api token]

* *api token*：是你希望通知的Team Inbox的API Token。你可以通过逗号分割的字符串或者数组来传递多个token。

> 注意：如果你的.travis.yml存储在公共库中，我们强烈推荐你[加密](/user/encryption-keys/)这个值：

    travis encrypt api_token --add notifications.flowdock

## HipChat通知

可以通过如下格式将通知发送到你的HipChat聊天室：

    notifications:
      hipchat: [api token]@[room id or name]

如果你在运行HipChat服务器，那么你可以像这样指定主机名：

    notifications:
      hipchat: [api token]@[hostname]/[room id or name]

* *api token*：你希望用来发送通知的账号的token。这个token可以使你的组管理员给你的API v1 token或者你管理的一个API v2 token。
* *主机名*：可选的，默认是api.hipchat.com，但是可以为HipChat服务器实例指定。
* *room id或者名称*：你希望提醒的房间的id或者名称。

如果你的房间名包含了空格，那么使用房间id。

> 注意：如果你的.travis.yml存储在公共库中，我们强烈推荐你[加密](/user/encryption-keys/)这个值：

    travis encrypt api_token@room_id_or_name --add notifications.hipchat.rooms

HipChat通知也支持模版，因此你可以定制通知的外观，例如减少到只有一行：

    notifications:
      hipchat:
        rooms:
          - [api token]@[room id or name]
        template:
          - '%{repository}#%{build_number} (%{branch} - %{commit} : %{author}): %{message}'

如果你希望发送HTML通知，你需要像这样添加`format: html`（注意这会关闭一些特性比如@提及与自动链接）：

    notifications:
      hipchat:
        rooms:
          - [api token]@[room id or name]
        template:
          - '%{repository}#%{build_number} (%{branch} - %{commit} : %{author}): %{message} (<a href="%{build_url}">Details</a>/<a href="%{compare_url}">Change view</a>)'
        format: html

使用V2 API，你可以通过设置`notify: true`来触发一个用户通知：

    notifications:
      hipchat:
        rooms:
          - [api token]@[room id or name]
        template:
          - '%{repository}#%{build_number} (%{branch} - %{commit} : %{author}): %{message}'
        notify: true

### 通知中的`From`值

当使用一个V1 token，通知由"Travis CI"发送。

使用一个V2 token，这个值由token的Label设置。创建一个特定目的带有期望的标签的房间的通知token（在房间的管理部分的"Tokens"），并且使用这个token。

<figure>
  <img src="{{ "/images/hipchat_token_screen.png" | prepend: site.baseurl }}" alt="HipChat Room Notification Tokens screenshot" width="550px" />
</figure>

## Sqwiggle通知

有了[Sqwiggle](https://www.sqwiggle.com)，你可以将Travis CI构建通知与当构建中断或者修复时看到你的队友的脸的乐趣结合起来。

要开始，你需要创建一个[Sqwiggle API的API token](https://www.sqwiggle.com/company/clients)。它足够只创建一个流式客户端，因为它有最少的权限。

接下来你需要发送通知的房间。

你可以在URL中使用房间的名称或者使用房间的id，这个当前可以通过[从API获取](https://www.sqwiggle.com/docs/endpoints/rooms#listallrooms)。

现在你可以添加详情到你的.travis.yml：

    notifications:
      sqwiggle: <api_token>@room

如果你希望通知多个房间，你可以给出一个token/房间组合的列表。

    notifications:
      sqwiggle:
        rooms:
          - <api_token>@mainhall
          - <api_token>@developers

Sqwiggle通知支持模版，因此你可以定制在你的流中消息 如何弹出。

默认值看起来是这样：

![](http://s3itch.paperplanes.de/sqwiggle_20140212_101412.jpg_20140213_103612.jpg)

要定制它，添加一个模版定义到你的.travis.yml。

    notifications:
      sqwiggle:
        rooms: <api_token>@mainhall
        template: '%{repository}#%{build_number} (%{branch} - %{commit} : %{author}): %{message}'

加密认证信息是推荐的。

## Slack通知

Travis CI支持将构建结果通知给任意的[Slack](http://slack.com)通道。

在Slack，建立一个[新的Travis CI集成](https://my.slack.com/services/new/travis)。选择一个通道，你将会找到要粘贴到你的.travis.yml中的详情。

<figure>
  <img src="http://s3itch.paperplanes.de/slackintegration_20140313_075147.jpg"/>
</figure>

在Slack设置中的通道名称可以被Travis CI的通知设置覆盖，因此你可以建立一个集成并在多个通道中使用，而不必理会初始设置。

只需要复制粘贴已经包含了合适token的设置到你的`.travis.yml`，那么你就可以开始了。

就像做派一样简单，但是如果你希望更多定制，那么继续阅读。

最简单的配置需要你的账号名称与你刚创建的token。

    notifications:
      slack: '<account>:<token>'

覆盖通道也是可能的，只需要用一个`#`隔开账户与token，添加到配置中。

    notifications:
      slack: '<account>:<token>#development'

你也可以指定多个通道。

    notifications:
      slack:
        rooms:
          - <account>:<token>#development
          - <account>:<token>#general

一如既往，我们推荐用我们的[travis](https://github.com/travis-ci/travis#readme)命令行客户端加密认证信息。

    travis encrypt "<account>:<token>" --add notifications.slack

一旦所有东西都设置好了，推送一个新的提交，你将会看到一些东西，就像下面的屏幕快照：

<figure>
  <img src="http://s3itch.paperplanes.de/slackmessage_20140313_180150.jpg">
</figure>

Slack将会提醒普通分支构建与pull request。

## Webhook通知

你可以用同样的方式定义将构建结果通知给webhook：

    notifications:
      webhooks: http://your-domain.com/notifications

或者多个URL：

    notifications:
      webhooks:
        - http://your-domain.com/notifications
        - http://another-domain.com/notifications

就像webhook负载发送时你可以指定的其他通知类型一样：

    notifications:
      webhooks:
        urls:
          - http://hooks.mydomain.com/travisci
          - http://hooks.mydomain.com/events
        on_success: [always|never|change] # default: always
        on_failure: [always|never|change] # default: always
        on_start: [always|never|change] # default: always

### Webhooks交付格式

使用HTTP POST来交付webhook， 内容类型是`application/x-www-form-urlencoded`，body包括了一个包含URL编码格式的JSON webhook负载的`payload`参数。

这里是一个你可以在`payload`中找到的例子：

<script src="https://gist.github.com/roidrage/9272064.js"></script>

你可以在`status`/`result`字段中看到下面代表构建状态的值中的一个。

* *0*：代表一个完全成功的构建
* *1*：代表一个构建并未完成或者完成并失败

此外一个消息将出现在`status_message`/`result_message`字段来进一步描述构建的状态。

* *Pending*：一个构建已经请求
* *Passed*：构建完全成功
* *Fixed*：在之前一次失败的构建后，构建完全成功
* *Broken*：在之前构建成功后，构建完成但是失败
* *Failed*: 构建是一个新分支的第一次构建，失败了
* *Still Failing：在之前一次失败的构建后，构建完成但是失败

对于pull requests， `type`字段值会是`pull_request`，`pull_request_number`字段也会包含，来指出pull request在GitHub上的issue号码。

这里有一个[Sinatra](http://sinatrarb.com)应用的例子，来解码请求与负载：

	require 'sinatra'
	require 'json'
	require 'digest/sha2'

	class TravisWebhook < Sinatra::Base
	  set :token, ENV['TRAVIS_USER_TOKEN']

	  post '/' do
	    if not valid_request?
	      puts "Invalid payload request for repository #{repo_slug}"
	    else
	      payload = JSON.parse(params[:payload])
	      puts "Received valid payload for repository #{repo_slug}"
	    end
	  end

	  def valid_request?
	    digest = Digest::SHA2.new.update("#{repo_slug}#{settings.token}")
	    digest.to_s == authorization
	  end

	  def authorization
	    env['HTTP_AUTHORIZATION']
	  end

	  def repo_slug
	    env['HTTP_TRAVIS_REPO_SLUG']
	  end
	end

要快速识别出涉及的库，我们包含了一个`Travis-Repo-Slug`header，格式是`account/repository`，例如`travis-ci/travis-ci`。

### Webhook授权

当Travis CI做POST请求时，包含了一个名为`Authorization`的header。其值是GitHub用户名的SHA2哈希值（如下），库名称与你的Travis CI token。

例如，在Python中，使用这个代码片段：

    from hashlib import sha256
    sha256('username/repository' + TRAVIS_TOKEN).hexdigest()

以此来确保发起到你的webhook请求的是Travis CI。

用来授权webhook的Travis CI token是用户的token，你可以在你的资料页找到。

![]({{ "/images/token.jpg" | prepend: site.baseurl }})

这是最初在Travis CI建立库的用户的token。如果你不确定是谁，那么你可以在GitHub上你的库的服务hook页找到。

这个进程将来会重写，因此用户token并不一直可靠，但是我们将会提前宣布任何改变。

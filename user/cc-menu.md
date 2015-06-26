---
title: 使用CCMenu与Travis CI
layout: en
permalink: /user/cc-menu/
---
<figure class="small right">
  <img src="http://s3itch.paperplanes.de/Backstop_Menubar_20140305_155352_20140305_155425.jpg"/>
</figure>

[CCMenu](http://ccmenu.org/)是一个OS X状态栏上方便地追踪你的库的最新构建状态的一个小工具。

[CCTray](http://sourceforge.net/projects/ccnet/files/CruiseControl.NET%20Releases/CruiseControl.NET%201.8.4/)是你的Windows环境下类似的工具，[BuildNotify](https://bitbucket.org/Anay/buildnotify/wiki/Home)是Linux系统下的。通用指南可以应用在所有这些上。

它们本来是为CruiseControl构建的，但是与Travis CI配合良好，你可以用来轮训你的Travis CI病将其状态展示在菜单栏或者托盘。

### 使用库的CC feed

开源库使用URL协议`https://api.travis-ci.org/repos/<owner>/<repository>/cc.xml`来获取CruiseControl feed。这个服务直接由我们的API提供。

<figure>
  <img src="http://s3itch.paperplanes.de/Projects_20140305_165324_20140305_165329.jpg"/>
</figure>

私有库使用不同的URL协议，由一个不同的[API endpoint](https://api.travis-ci.com)提供服务：

<figure>
  <img src="http://s3itch.paperplanes.de/Screenshot_20140305_165022_20140305_165032.jpg"/>
</figure>

私有库需要一个包含token的认证URL。你可以在你的资料中找到token：

![]({{ "/images/token.jpg" | prepend: site.baseurl }})

### 使用账号的CC feed

上面的技术只允许你一次添加一个库，对于着手于多个库的组织的成员来说是笨拙的。不必指定拥有者与库，你可以简单地指出拥有者并选择项目的一个子集。

* 对于开源项目使用`https://api.travis-ci.org/repos/<owner>.xml`
* 对于闭源项目使用`https://api.travis-ci.com/repos/<owner>.xml?token=<token>`

CCMenu将会展示所有你可以一个一个地快速添加的可用库的列表。

<figure>
  <img src="http://s3itch.paperplanes.de/Screenshot_20140305_164512_20140305_164517.jpg"/>
</figure>

非常容易！

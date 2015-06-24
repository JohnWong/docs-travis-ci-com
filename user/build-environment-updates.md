---
title: 构建环境更新历史
layout: en
permalink: /user/build-environment-updates/
---
### 2014年12月及以后

环境大概会在偶数月（2月，4月，6月，8月，10月，12月）的第一周更新。特定语言的更新将会按需发布。

<ul class="list--blank">
{% for page in site.pages %}
{% if page.category == "build_env_updates" %}
	<li><a href="{{ page.permalink | prepend: site.baseurl }}">{{ page.permalink | remove:'/user/build-environment-updates/' | remove: '/' }}</a></li>
{% endif %}
{% endfor %}
</ul>

### Atom feed
<a href="/feed.build-env-updates.xml">Atom feed</a>也可用。

### 邮件列表
你也可以注册<a href="http://eepurl.com/9OCsP">公告邮件列表</a>。

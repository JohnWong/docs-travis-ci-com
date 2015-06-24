---
title: 添加到SSH已知主机
layout: en
permalink: /user/ssh-known-hosts/
---
<div id="toc">
</div>

Travis CI可以在克隆你的git库之前添加条目到`~/.ssh/known_hosts`。如果有除了`github.com`、`gist.github.com`或者`ssh.github.com`之外的git submodules，那么这是必要的。

主机名与IP地址都支持，键是通过`ssh-keyscan`添加的。单个主机可以像这样指定：

    addons:
      ssh_known_hosts: git.example.com


多个主机或者IP可以作为列表添加：

    addons:
      ssh_known_hosts:
      - git.example.com
      - 111.22.33.44

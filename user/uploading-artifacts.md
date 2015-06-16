---
title: 上传工件到Travis CI
layout: en
permalink: /user/uploading-artifacts/
---
<div id="toc">
</div>

Travis CI可以自动上传你的构建工件（artifact）到S3。

最简单的配置，你需要做的只是添加如下内容到你的 `.travis.yml`：

    addons:
      artifacts: true

并添加下面的环境变量到库设置：

    ARTIFACTS_KEY=(AWS access key id)
    ARTIFACTS_SECRET=(AWS secret access key)
    ARTIFACTS_BUCKET=(S3 bucket name)

你可以[这里](https://console.aws.amazon.com/iam/home?#security_credential)找到你的AWS访问密钥。

### 部署到指定路径

上传到S3的默认路径可以通过`git ls-files -o`来找到git工作副本中所有未被追踪的文件。如果有任何需要上传的额外路径，可以像下面这样在 `addons.artifacts.paths` 中指定它们：

    addons:
      artifacts:
        # ...
        paths:
        - $(git ls-files -o | tr "\n" ":")
        - $(ls /var/log/*.log | tr "\n" ":")
        - $HOME/some/other/thing.log

或者在库设置中作为一个环境变量：

    # ':'-delimited paths, e.g.
    ARTIFACTS_PATHS="./logs:./build:/var/log"

### 调试

如果你希望了解工件插件所做事情的更多详情，设置 `addons.artifacts.debug` 为任何非空值可以打开调试日志。

    addons:
      artifacts:
        # ...
        debug: true

或者将其定义为库设置的环境变量，或者在 `env.global` 部分添加：

    ARTIFACTS_DEBUG=1

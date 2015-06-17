---
title: 建立服务和数据库 
layout: en
permalink: /user/database-setup/
---

这份指南涵盖了在Travis CI环境下设置大多数常用数据库和其他服务。希望寻找的信息是[配置多个数据库？](/user/database-setup/#Multiple-database-systems)

下面的服务都可用，它们都使用默认设置，除了一些额外的用户和松懈的安全设置：

{% include databases.html %}

## 启动服务

大多数服务并不在随系统启动，从而给测试套件留出更多RAM，所以你需要使用 `.travis.yml` 中的 `services` 条目来告诉Travis CI来启动他们：

    services: mongodb

或者启动几个服务：

    services:
      - riak      # start riak
      - rabbitmq  # start rabbitmq-server
      - memcached # start memcached

> 注意，这个特性只在我们提供的[CI环境](/user/ci-environment/)下可以提供服务。如果你下载Apache Jackrabbit，你仍然不得不在 `before_install` 步骤中启动它。

### MySQL

在Travis CI中，MySQL会在**随系统启动**，绑定到127.0.0.1并请求身份验证。你可以使用用户名 “travis”或“root”和空密码来连接。

> 注意，“travis”用户并没有“root”用户所拥有的所有特权。

#### 使用MySQL和ActiveRecord

使用ActiveRecord的Ruby项目的 `config/database.yml` 示例：

    test:
      adapter: mysql2
      database: myapp_test
      username: travis
      encoding: utf8

你可能需要先创建 `myapp_test` 数据库。将运行它作为你的构建脚本的一部分：

    # .travis.yml
    before_script:
      - mysql -e 'create database myapp_test;'

#### 注意 `test` 数据库

在一些老版本的MySQL中，Ubuntu包默认提供了 `test` 数据库。在5.5.37版本处于安全的考虑不再这样做（参见[变更记录](http://changelogs.ubuntu.com/changelogs/pool/main/m/mysql-5.5/mysql-5.5_5.5.37-0ubuntu0.12.04.1/changelog)）。

如果你需要它，使用下面的 `before_install` 来创建它：

```yaml
before_install:
  - mysql -e "create database IF NOT EXISTS test;" -uroot
```

### PostgreSQL


#### 选择一个PostgreSQL版本

默认的构建环境中9.1版本已经在运行。

你可以很容易地通过你的 .travis.yml 来选择一个不同的版本。

我们安装了PostgreSQL 9.1, 9.2 and 9.3， 通常安装了最新发布的布丁。我们从官方的[PostgreSQL APT库](http://apt.postgresql.org)安装它们。

目前你会发现分别**安装了如下版本：9.1.14, 9.2.9 and 9.3.5**。

要选择一个非默认的不同版本，在你的 .travis.yml 使用下面的设置：

    addons:
      postgresql: "9.3"

这将会选择PostgreSQL 9.3作为你的构建期望运行的版本。

**可用的选择有“9.1”，“9.2”和“9.3”**。确保只指定主版本号和副版本号，不包括补丁发布。

#### 在你的构建中使用PostgreSQL

本地访问PostgreSQL服务器的默认用户是 `postgres` ，没有设置密码。

为你的应用创建一个数据库只需要在你的 .travis.yml 额外添加一行：

    before_script:
      - psql -c 'create database travis_ci_test;' -U postgres

对于Rails应用，现在你可以使用下面的 database.yml 配置来本地访问数据库：

    test:
      adapter: postgresql
      database: travis_ci_test
      username: postgres

如果你的本地测试应该设置不同的认证或者访问本地测试数据的设置，我们推荐你将这些设置放到你的库的 database.yml.travis 中，并将它作为你的构建的一部分：

    before_script:
      - cp config/database.yml.travis config/database.yml

#### 使用PostGIS

所有可用的PostgreSQL版本都预装了PostGIS 2.1包。

如果你的构建需要启用扩展，默认并不会开启。下面是你的 .travis.yml 的例子：

    before_script:
      - psql -U postgres -c "create extension postgis"

#### PostgreSQL与语言环境

默认可用的语言环境有限，因此你可能需要依据你需要的语言来安装它们。

下面是安装西班牙语言包的步骤。注意，你需要从你的.travis.yml的 `addons` 部分删除PostgreSQL版本。

    before_install:
      - sudo apt-get update
      - sudo apt-get install language-pack-es
      - sudo /etc/init.d/postgresql stop
      - sudo /etc/init.d/postgresql start 9.3

<div class="note-box">
注意，在<a href="/user/workers/container-based-infrastructure">基于容器的worker</a>中运行构建时<code>sudo</code>不可用。
</div>

下面是当前默认安装在系统中的语言环境列表：

    C
    C.UTF-8
    en_AG
    en_AG.utf8
    en_AU.utf8
    en_BW.utf8
    en_CA.utf8
    en_DK.utf8
    en_GB.utf8
    en_HK.utf8
    en_IE.utf8
    en_IN
    en_IN.utf8
    en_NG
    en_NG.utf8
    en_NZ.utf8
    en_PH.utf8
    en_SG.utf8
    en_US.utf8
    en_ZA.utf8
    en_ZM
    en_ZM.utf8
    en_ZW.utf8
    POSIX

目前Ubuntu 12.04中可用的所有的语言包可以从[包网站](http://packages.ubuntu.com/search?keywords=language-pack&searchon=names&suite=precise&section=all)找到。

### SQLite3

可能是你的关系数据库需求的最容易最简单的解决方案。如果你并不明确地希望测试你的代码在其他数据库的行为，内存SQLite可能是最好的选择。

#### Ruby工程中的SQLite3

对于Ruby工程来说，确保你的bundle中Sqlite3的ruby绑定：

    # Gemfile
    # for CRuby, Rubinius, including Windows and RubyInstaller
    gem "sqlite3", :platform => [:ruby, :mswin, :mingw]

    # for JRuby
    gem "jdbc-sqlite3", :platform => :jruby

项目使用ActiveRecord的 `config/database.yml` 的示例：

    test:
      adapter: sqlite3
      database: ":memory:"
      timeout: 500

如果你不在使用一个 `config/database.yml` 文件来配置ActiveRecord，你需要在测试中手动连接数据库。例如可以通过如下方法链接ActiveRecord：

    ActiveRecord::Base.establish_connection :adapter => 'sqlite3',
                                            :database => ':memory:'

### MongoDB

MongoDB**不随系统启动**。添加如下内容到 `.travis.yml` 来启动它。

    services:
      - mongodb

MongoDB绑定到127.0.0.1，不需要身份验证或者预先创建数据库。如果你添加了一个admin用户，将会启用身份验证，因为 `mongod` 是随着参数 `--auth` 启动的。

> 注意：admin用户是在admin数据库创建的用户。

要在你的数据库创建用户，添加 `before_script` 到你的 `.travis.yml` 文件：

    # .travis.yml
    before_script:
      - mongo mydb_test --eval 'db.addUser("travis", "test");'

#### MongoDB可能不会立即接受连接

一些用户反馈在工作尝试执行命令时，MongoDB可能不会接受连接。这个问题是间歇性的，唯一可靠的避免方法是在第一次连接前插入人工等待：

    # .travis.yml
    before_script:
      - sleep 15
      - mongo mydb_test --eval 'db.addUser("travis", "test");'

### CouchDB

CouchDB **不随系统启动**。添加如下内容到 `.travis.yml` 来启动它：

    services:
      - couchdb

CouchDB绑定到127.0.0.1，使用stock配置，不需要身份验证（在CouchDB文档中，它运行在admin组）。

你不得不将创建数据库作为你构建进程的一部分：

    # .travis.yml
    before_script:
      - curl -X PUT localhost:5984/myapp_test

### RabbitMQ

RabbitMQ**不随系统启动**。添加如下内容到`.travis.yml`来启动它：

    services:
      - rabbitmq

RabbitMQ使用默认配置的vhost（`/`），用户名（`guest`）和密码（`guest`）。如果需要，你可以使用 `before_script` 设置更多的vhost和角色（例如测试身份验证）。

### Riak

Riak**不随系统启动**。添加如下内容到 `.travis.yml` 来启动它：

    services:
      - riak

Riak使用stock配置，除了一个例外：配置为使用[LevelDB存储后端](http://docs.basho.com/riak/latest/ops/advanced/backends/leveldb/)。Riak搜索时启用的。

### Memcached

Memcached**不随系统启动**。添加如下内容到`.travis.yml`来启动它：

    services:
      - memcached

Memcached使用stock配置，绑定到localhost。

### Redis

Redis**不随系统启动**。添加如下内容到 `.travis.yml` 来启动它：

    services:
      - redis-server

Redis使用stock配置，可以在localhost上使用。


### Cassandra

Cassandra**不随系统启动**。添加如下内容到`.travis.yml`来启动它：

    services:
      - cassandra

Cassandra由[Datastax社区版](http://www.datastax.com/products/community)提供，使用stock配置可以在127.0.0.1使用。

#### 老版本

如果你需要一个老版本的Cassandra，你可以添加类似如下内容的命令到你的 `.travis.yml`：

```yaml
before_install:
  - sudo rm -rf /var/lib/cassandra/*
  - wget http://www.us.apache.org/dist/cassandra/1.2.18/apache-cassandra-1.2.18-bin.tar.gz && tar -xvzf apache-cassandra-1.2.18-bin.tar.gz && sudo sh apache-cassandra-1.2.18/bin/cassandra
```

> `sudo` 在[基于容器的worker](/user/ci-environment/#Virtualization-environments)下不可用。

### Neo4J

Neo4J服务器（社区版）**不随系统启动**。添加如下内容到`.travis.yml`来启动它：

    services:
      - neo4j

Neo4J服务器使用默认的配置（localhost，7474端口）。

> Neo4j无法在基于容器的worker启动。参见[https://github.com/travis-ci/travis-ci/issues/3243](https://github.com/travis-ci/travis-ci/issues/3243)

### ElasticSearch

ElasticSearch**不随系统启动**。添加如下内容到`.travis.yml`来启动它：

    services:
      - elasticsearch

ElasticSearch可能需要花费几秒钟来启动，可能在脚本执行后不可用。 在这种情况下，我们可以使用 ``sleep`` 命令让Travis CI等待：

    before_script:
      - sleep 10

ElasticSearch由官方Debian包提供，使用stock配置（在127.0.0.1可用）。

#### 使用指定版本的ElasticSearch

你可以用如下方法用你需要的版本（比如1.2.4）覆盖已安装的ElasticSearch：

```yaml
before_install:
  - curl -O https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.2.4.deb && sudo dpkg -i --force-confnew elasticsearch-1.2.4.deb
```
> `sudo` 在[基于容器的worker](/user/ci-environment/#Virtualization-environments)下不可用。

#### 混乱的输出

当ElasticSearch启动时，你可能看到像下面这样损坏的消息：

```
$ sudo service elasticsearch start
 * Starting ElasticSearch Server       ission denied on key 'vm.max_map_count'
```

这是由于[ElasticSearch最近的变更](https://github.com/elasticsearch/elasticsearch/issues/4397)，报告在[这里](https://github.com/elasticsearch/elasticsearch/issues/4978)。信息是无害的，服务功能正常。

### 多数据库系统

如果你项目的测试需要使用不同的数据库运行多次，可以在Travis CI上使用一个env变量技术来配置。这个技术只是一个惯例，需要 `before_script` 或者 `before_install` 来生效。

#### 在before_script步骤使用ENV变量

使用`DB`环境变量来制定数据库配置的名称。在本地运行：

    $ DB=postgres [commands to run your tests]

在Travis CI你需要创建一个包含三个构建的[构建矩阵](/user/customizing-the-build/#Build-Matrix)，每个都有导出为不同值的`DB`变量，你可以在 `.travis.yml` 使用  `env` 选项：

    env:
      - DB=sqlite
      - DB=mysql
      - DB=postgres

然后你可以在`before_install`（或`before_script`）步骤使用这些值来设置每个数据库。例如：

    before_script:
      - sh -c "if [ '$DB' = 'pgsql' ]; then psql -c 'DROP DATABASE IF EXISTS doctrine_tests;' -U postgres; fi"
      - sh -c "if [ '$DB' = 'pgsql' ]; then psql -c 'DROP DATABASE IF EXISTS doctrine_tests_tmp;' -U postgres; fi"
      - sh -c "if [ '$DB' = 'pgsql' ]; then psql -c 'create database doctrine_tests;' -U postgres; fi"
      - sh -c "if [ '$DB' = 'pgsql' ]; then psql -c 'create database doctrine_tests_tmp;' -U postgres; fi"
      - sh -c "if [ '$DB' = 'mysql' ]; then mysql -e 'create database IF NOT EXISTS doctrine_tests_tmp;create database IF NOT EXISTS doctrine_tests;'; fi"


> Travis CI对这些变量并没有任何特殊的支持，它只是创建了有不同导出值的三个构建。这取决于你的测试套件或者`before_install`/`before_script`步骤来利用它们。

实际例子可以参考[doctrine/doctrine2 .travis.yml](https://github.com/doctrine/doctrine2/blob/master/.travis.yml)。

#### 一个Ruby特定的方法

另一个Ruby特定的方法是将你所有的数据库配置放到一个YAML文件（例如`test/database.yml`），比如ActiveRecord是：

    sqlite:
      adapter: sqlite3
      database: ":memory:"
      timeout: 500
    mysql:
      adapter: mysql2
      database: myapp_test
      username:
      encoding: utf8
    postgres:
      adapter: postgresql
      database: myapp_test
      username: postgres

接下来在你的测试套件中读取数据到一个配置hash:

    configs = YAML.load_file('test/database.yml')
    ActiveRecord::Base.configurations = configs

    db_name = ENV['DB'] || 'sqlite'
    ActiveRecord::Base.establish_connection(db_name)
    ActiveRecord::Base.default_timezone = :utc

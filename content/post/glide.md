---
author : "杨承龙"
date : "2017-07-23T10:08:28+08:00"
draft : false
title : "glide"
tags : ["go"]
comments : true     
share : true        
menu : "main" 
          
---
Golang依赖管理工具：glide

go 包管理

golang是一个十分有趣，简洁而有力的开发语言，用来开发并发/并行程序是一件很愉快的事情。但有一个问题的是它的包管理，并没有官方最佳管理方案。go语言的包是没有中央库统一管理的，通过使用go get命令从远程代码库(github.com,goolge code 等)拉取，直接跳过中央版本库的约束，让代码的拉取直接基于源代码版本控制库，开发者间的协同直接依赖于源代码的版本控制。直接去除了库版本的概念，没有明显的包版本标识，官方的建议是把外部依赖的代码全部复制到自己可控的源代码库中，进行统一管理，从而做到对依赖包的可控管理。但使用过程中，还是有很多缺陷，总结来说就是以下几点：

- 能拉取源码的平台很有限，绝大多数依赖的是 github.com
- 不能区分版本，以至于令开发者以最后一项包名作为版本划分
- 依赖列表/关系无法持久化到本地，需要找出所有依赖包然后一个个 go get
- 只能依赖本地全局仓库（GOPATH/GOROOT），无法将库放置于局部仓库（$PROJECT_HOME/vendor）

1.5版本的vendor目录特性后，官方wiki推荐了多种支持这种特性的包管理工具如：Godep、gv、gvt、glide、Govendor等。这里我们介绍下glide。

glide



glide是Go的包管理工具。支持语义化版本,支持Git、Svn等，支持Go工具链，支持vendor目录，支持从Godep、GB、GPM、Gom倒入，支持私有的Repos和Forks。

Glide 是 Golang 的 Vendor 包管理器，方便你管理 vendor 和 verdor 包。类似 Java 的 Maven，PHP 的 Composer。

- Github：https://github.com/Masterminds/glide
- 在线文档：http://glide.readthedocs.io/en/stable

使用glide管理的工程目录结构如下：

    - $GOPATH/src/myProject (Your project)
      |
      |-- glide.yaml
      |
      |-- glide.lock
      |
      |-- main.go (Your main go code can live here)
      |
      |-- mySubpackage (You can create your own subpackages, too)
      |    |
      |    |-- foo.go
      |
      |-- vendor
           |-- github.com
                |
                |-- Masterminds
                      |
                      |-- ... etc.

主要特性：

- 简单管理依赖
- 支持所有 go 工具
- 支持定制本地和全局插件 (see docs/plugins.md)
- 仓库缓存
- 持久化依赖列表至配置文件中，包括依赖版本（支持范围限定）以及私人仓库等
- 持久化关系树至 lock 文件中（类似于 yarn 和 cargo），以重复拉取相同版本依赖
- 兼容 go get 所支持的版本控制系统：Git, Bzr, HG, and SVN
- 支持 GO15VENDOREXPERIMENT 特性，使得不同项目可以依赖相同项目的不同版本
- 可以导入其他工具配置，例如： Godep, GPM, Gom, and GB

安装

Golang环境设置

采用vendor目录特性，Go 1.5 做为试验特性加入（需要指定 GO15VENDOREXPERIMENT=1 环境变量），并在 Go 1.6 正式引入的一个概念。多数 go 依赖解决方案都基于它的。GO15VENDOREXPERIMENT 是 Go 1.5 版本新增的一个环境变量，如果将值改为 1 则表示启用。它可以将项目根目录名为 vendor 的目录添加到 Go 的库搜寻路径中，实现一个局部依赖的效果。

特性在 1.5 版本作为实验特性被添加，1.6 中默认被启用，1.7 移除变量加入标准中。

Go 提供了原始的 go get ，让第三方包管理可以基于 go get 做扩展。GO15VENDOREXPERIMENT 特性让局部依赖成为现实。

    //设置环境变量 使用vendor目录
    GO15VENDOREXPERIMENT=1

安装glide

    获取
    $ go get github.com/Masterminds/glide
    进入目录
    $ cd github.com/Masterminds/glide
    编译
    $ make build
    $ go build -o glide -ldflags "-X main.version=v0.11.0" glide.go

或者

    curl https://glide.sh/get | sh

或者

    $ go get github.com/Masterminds/glide
    $ go install github.com/Masterminds/glide

验证是否安装成功

    $ glide
    NAME:
       glide - Vendor Package Management for your Go projects.
    
       Each project should have a 'glide.yaml' file in the project directory. Files
       look something like this:
    
           package: github.com/Masterminds/glide
           imports:
           - package: github.com/Masterminds/cookoo
        ......
       [$GLIDE_TMP]
       --no-color              Turn off colored output for log messages
       --help, -h              show help
       --version, -v           print the version

看到这样，已经安装成功了！

使用

篇幅有限，我只介绍经常使用到的。 先进入在GOPATH的一个项目中。

    cd $GOPATH/src/foor

初始化 (glide init)

Glide 能检测项目的依赖包并且创建一个名为 glide.yaml 的文件。能自动识别 Godep, GPM, Gom, and GB 等工具的配置文件。在 项目的根目录执行如下命令：

    $ glide init
    [INFO]  Generating a YAML configuration file and guessing the dependencies
    [INFO]  Attempting to import from other package managers (use --skip-import to skip)
    [INFO]  Scanning code to look for dependencies
    [INFO]  --> Found reference to github.com/urfave/cli
    [INFO]  Writing configuration file (glide.yaml)
    [INFO]  Would you like Glide to help you find ways to improve your glide.yaml configuration?
    [INFO]  If you want to revisit this step you can use the config-wizard command at any time.
    [INFO]  Yes (Y) or No (N)?
    Y
    [INFO]  Loading mirrors from mirrors.yaml file
    [INFO]  Looking for dependencies to make suggestions on
    [INFO]  --> Scanning for dependencies not using version ranges
    [INFO]  --> Scanning for dependencies using commit ids
    [INFO]  Gathering information on each dependency
    [INFO]  --> This may take a moment. Especially on a codebase with many dependencies
    [INFO]  --> Gathering release information for dependencies
    [INFO]  --> Looking for dependency imports where versions are commit ids
    [INFO]  Here are some suggestions...
    [INFO]  The package github.com/urfave/cli appears to have Semantic Version releases (http://semver.org).
    [INFO]  The latest release is v1.19.1. You are currently not using a release. Would you like
    [INFO]  to use this release? Yes (Y) or No (N)
    Y
    [INFO]  Would you like to remember the previous decision and apply it to future
    [INFO]  dependencies? Yes (Y) or No (N)
    Y
    [INFO]  Updating github.com/urfave/cli to use the release v1.19.1 instead of no release
    [INFO]  The package github.com/urfave/cli appears to use semantic versions (http://semver.org).
    [INFO]  Would you like to track the latest minor or patch releases (major.minor.patch)?
    [INFO]  Tracking minor version releases would use '>= 1.19.1, < 2.0.0' ('^1.19.1'). Tracking patch version
    [INFO]  releases would use '>= 1.19.1, < 1.20.0' ('~1.19.1'). For more information on Glide versions
    [INFO]  and ranges see https://glide.sh/docs/versions
    [INFO]  Minor (M), Patch (P), or Skip Ranges (S)?
    P
    [INFO]  Would you like to remember the previous decision and apply it to future
    [INFO]  dependencies? Yes (Y) or No (N)
    Y
    [INFO]  Updating github.com/urfave/cli to use the range ~1.19.1 instead of commit id v1.19.1
    [INFO]  Configuration changes have been made. Would you like to write these
    [INFO]  changes to your configuration file? Yes (Y) or No (N)
    Y
    [INFO]  Writing updates to configuration file (glide.yaml)
    [INFO]  You can now edit the glide.yaml file.:
    [INFO]  --> For more information on versions and ranges see https://glide.sh/docs/versions/
    [INFO]  --> For details on additional metadata see https://glide.sh/docs/glide.yaml/
    $ ll
    glide.yaml
    $ cat glide.yaml 
    package: foor
    import: []

在初始化过程中，glide扫描代码目录， 期间会询问一些问题，最后会创建一个glide.yaml文件。 glide.yaml记载了依赖包的列表及其更新规则，每次执行 glide up 时，都会按照指定的规则（如只下载补丁(patch)不下载升级(minor)）下载新版。

一个完整的gilde.yaml

    package: foor
    homepage: https://github.com/qiangmzsx
    license: MIT
    owners: 
    - name: qiangmzsx
      email: qiangmzsx@hotmail.com
      homepage: https://github.com/qiangmzsx
    # 去除包
    ignore:
    - appengine
    - golang.org/x/net
    # 排除目录
    excludeDirs:
    - node_modules
    # 导入包
    import:
    - package: github.com/astaxie/beego
      version: 1.8.0
    - package: github.com/coocood/freecache
    - package: github.com/garyburd/redigo/redis
    - package: github.com/go-sql-driver/mysql
    - package: github.com/bitly/go-simplejson
    - package: git.oschina.net/qiangmzsx/beegofreecache
    testImport:
    - package: github.com/smartystreets/goconvey
      subpackages:
      - convey

可以看到文件中有以下元素：

- package: 顶级包导入路径，即 相对 GOPATH/src 的包的导入路径。
- homepage: 项目的主页，如 http://k8s.io 。
- license: 项目的使用权协议。
- owners: 项目的作者信息，如 姓名、联系方式（email）、个人主页 等。
- ignore: 需要忽略导入的包名列表。注意，不是文件夹名。
- excludeDirs: 需要忽略扫描依赖包的文件夹列表。
- import: 需要导入的依赖包，每个依赖包 包含以下元素：
  - package: 包路径信息。
  - version: 版本号，如 语义版本、范围版本、分支版本、标签版本、提交号版本等，更多的信息查看文档：versioning documentation。
  - repo: 包的仓库地址信息。如果包名不是包仓库路径信息或者是 私有仓库，可以从线上的仓库 check out 文件到 包名中指定的（本地）路径，也可以使用 fork 指令。 
  - vcs: 使用到的版本控制系统，如 git、hg、bzr 或 svn。如果不能从上述的 package 中确定版本控制系统类型的话，需要在这里指定。如仓库路径带有 .git 后缀或 GitHub仓库路径等都可以自动识别类型为 git。
  - subpackages: 当前包所使用到的子包名列表。
  - os: 能适用于哪些操作系统。 与运行时(runtime) 当中的操作系统变量(GOOS) 匹配比较，如果是匹配上，就获取该包。
  - arch: 能使用于哪些结构。与 运行时(runtime) 当中的 runtime architecture 变量（GOARCH）匹配比较，如果匹配上，就获取该包。
- testImport: 用于测试的包。

版本号的指定（version字段）

Glide 支持 语义版本，范围版本，分支版本，标签版本，版本ID 最为版本号。

基本范围

举个简单例子，范围版本 > 1.2.3，表示 Glide 会用 大于 1.2.3 以后的最新版本。目前支持如下操作符：

- =: equal (aliased to no operator)
- !=: not equal
- >: greater than
- <: less than
- >=: greater than or equal to
- <=: less than or equal to

它们还可以组合用，“,”表示 “逻辑与”，“||”表示“逻辑或”，如 ">= 1.2, < 3.0.0 || >= 4.2.3"

连字符范围

- 1.2 - 1.4.5 which is equivalent to >= 1.2, <= 1.4.5
- 2.3.4 - 4.5 which is equivalent to >= 2.3.4, <= 4.5

通配符比较

字符 “x”、“X”、"*" 可以当做通配符，它可以应用于所有的比较操作中。当应用于“=”操作符中，它表示补丁最低级别的比较，如下所示：

- 1.2.x is equivalent to >= 1.2.0, < 1.3.0
- >= 1.2.x is equivalent to >= 1.2.0
- <= 2.x is equivalent to < 3
- - is equivalent to >= 0.0.0

波浪线 范围比较（补丁）

波浪线比较操作符主要用于补丁级别范围比较，当为次要版本数字没指定时，次要版本不变，主版本修改。如下所示：

- ~1.2.3 is equivalent to >= 1.2.3, < 1.3.0
- ~1 is equivalent to >= 1, < 2
- ~2.3 is equivalent to >= 2.3, < 2.4
- ~1.2.x is equivalent to >= 1.2.0, < 1.3.0
- ~1.x is equivalent to >= 1, < 2

脱字符 范围比较（主版本）

脱字符比较操作符是 为主版本号修改服务的。通常用于接口 主版本比较，如下所示：

- ^1.2.3 is equivalent to >= 1.2.3, < 2.0.0
- ^1.2.x is equivalent to >= 1.2.0, < 2.0.0
- ^2.3 is equivalent to >= 2.3, < 3
- ^2.x is equivalent to >= 2.0.0, < 3

glide 命令

安装依赖 (glide install)

如果想下载所需依赖包，执行如下命令：

    $ glide install
    [ERROR]    Failed to parse /home/users/xxxx/golang/src/foor/glide.yaml: yaml: invalid leading UTF-8 octet

报错了！别担心看看你的yaml文件是否为utf-8编码，不是就转换一下就好啦！

    $ glide install
    [INFO]    Lock file (glide.lock) does not exist. Performing update.
    [INFO]    Downloading dependencies. Please wait...
    [INFO]    --> Fetching updates for github.com/go-sql-driver/mysql
    [INFO]    --> Fetching updates for github.com/astaxie/beego
    [INFO]    --> Fetching updates for github.com/coocood/freecache
    [INFO]    --> Fetching updates for git.oschina.net/qiangmzsx/beegofreecache
    [INFO]    --> Fetching updates for github.com/bitly/go-simplejson
    [INFO]    --> Fetching updates for github.com/garyburd/redigo
    [INFO]    --> Fetching updates for github.com/smartystreets/goconvey
    [INFO]    --> Detected semantic version. Setting version for github.com/astaxie/beego to v1.8.0
    [INFO]    Resolving imports
    [INFO]    Downloading dependencies. Please wait...
    [INFO]    Setting references for remaining imports
    [INFO]    Exporting resolved dependencies...
    [INFO]    --> Exporting github.com/astaxie/beego
    [INFO]    --> Exporting github.com/coocood/freecache
    [INFO]    --> Exporting github.com/bitly/go-simplejson
    [INFO]    --> Exporting github.com/go-sql-driver/mysql
    [INFO]    --> Exporting github.com/garyburd/redigo
    [INFO]    --> Exporting github.com/smartystreets/goconvey
    [INFO]    --> Exporting git.oschina.net/qiangmzsx/beegofreecache
    [INFO]    Replacing existing vendor dependencies
    [INFO]    Project relies on 6 dependencies.
    $ ll
    total 12
    glide.lock
    glide.yaml
    vendor
    $ ll vendor/
    git.oschina.net
    github.com

这个命令会做以下 2 件事：

(1) 如果 glide.lock 文件已经存在，Glide 会把 glide.lock 中已经计算好的确定版本号的依赖包批量下载到 vendor/ 目录下；

(2) 如果 glide.local 文件不存在，则会先执行 update 命令；

glide同时也递归获取依赖包需要的任何依赖项包括配置文件中定义的依赖项目。glide递归获取依赖，可以识别Glide、Godep、gb、gom和GPM管理的项目。

当依赖被制定到特定的版本时，名为glide.lock的文件会被创建或者更新。例如，如果在glide.yaml中一个版本被指定在一个范围内(如：^1.2.3),那么glide将在glide.yaml中设定一个特定提交ID（commit id）。如此，将允许重复安装。当glide.lock文件和glide.yaml不同步时，如glide.yaml发生改变，glide将会提供一个警告。运行glide up命令更新依赖树时，将会重建glide.lock文件。

从获取的依赖包中移除嵌套的vendor/目录可以使用-v标记。

升级版本 (glide up)

glide up 会按照语义化版本规则更新依赖包代码，开发过程中如果需要使用新版代码，可以执行这个命令： 修改一下glide.yaml中的一个Package.

    - package: github.com/astaxie/beego
      version: 1.8.3

执行glide up。

    $ glide up
    [INFO]    Downloading dependencies. Please wait...
    [INFO]    --> Fetching updates for git.oschina.net/qiangmzsx/beegofreecache
    [INFO]    --> Fetching updates for github.com/garyburd/redigo
    [INFO]    --> Fetching updates for github.com/go-sql-driver/mysql
    [INFO]    --> Fetching updates for github.com/astaxie/beego
    [INFO]    --> Fetching updates for github.com/bitly/go-simplejson
    [INFO]    --> Fetching updates for github.com/coocood/freecache
    [INFO]    --> Fetching updates for github.com/smartystreets/goconvey
    [INFO]    --> Detected semantic version. Setting version for github.com/astaxie/beego to v1.8.3
    [INFO]    Resolving imports
    [INFO]    Downloading dependencies. Please wait...
    [INFO]    Setting references for remaining imports
    [INFO]    Exporting resolved dependencies...
    [INFO]    --> Exporting github.com/astaxie/beego
    [INFO]    --> Exporting github.com/bitly/go-simplejson
    [INFO]    --> Exporting github.com/garyburd/redigo
    [INFO]    --> Exporting github.com/go-sql-driver/mysql
    [INFO]    --> Exporting github.com/coocood/freecache
    [INFO]    --> Exporting github.com/smartystreets/goconvey
    [INFO]    --> Exporting git.oschina.net/qiangmzsx/beegofreecache
    [INFO]    Replacing existing vendor dependencies
    [INFO]    Project relies on 6 dependencies.

添加并下载依赖 (glide get)

除了自动从代码中解析 import 外，glide 还可以通过 glide get 直接下载代码中没有的依赖，与 go get 的用法基本一致：

    $ glide get github.com/orcaman/concurrent-map
    [INFO]    Preparing to install 1 package.
    [INFO]    Attempting to get package github.com/orcaman/concurrent-map
    [INFO]    --> Gathering release information for github.com/orcaman/concurrent-map
    [INFO]    --> Adding github.com/orcaman/concurrent-map to your configuration
    [INFO]    Downloading dependencies. Please wait...
    [INFO]    --> Fetching updates for github.com/garyburd/redigo
    [INFO]    --> Fetching updates for github.com/astaxie/beego
    [INFO]    --> Fetching updates for github.com/go-sql-driver/mysql
    [INFO]    --> Fetching updates for git.oschina.net/qiangmzsx/beegofreecache
    [INFO]    --> Fetching updates for github.com/bitly/go-simplejson
    [INFO]    --> Fetching github.com/orcaman/concurrent-map
    [INFO]    --> Fetching updates for github.com/coocood/freecache
    [INFO]    --> Fetching updates for github.com/smartystreets/goconvey
    [INFO]    Resolving imports
    [INFO]    Downloading dependencies. Please wait...
    [INFO]    --> Detected semantic version. Setting version for github.com/astaxie/beego to v1.8.3
    [INFO]    Exporting resolved dependencies...
    [INFO]    --> Exporting github.com/smartystreets/goconvey
    [INFO]    --> Exporting github.com/garyburd/redigo
    [INFO]    --> Exporting github.com/go-sql-driver/mysql
    [INFO]    --> Exporting github.com/orcaman/concurrent-map
    [INFO]    --> Exporting github.com/astaxie/beego
    [INFO]    --> Exporting github.com/bitly/go-simplejson
    [INFO]    --> Exporting github.com/coocood/freecache
    [INFO]    --> Exporting git.oschina.net/qiangmzsx/beegofreecache
    [INFO]    Replacing existing vendor dependencies

这个 get 命令 也可以带版本号，如下所示：

    glide get github.com/Masterminds/semver#~1.2.0

“#”是依赖包 和 版本号的分隔符，版本号可以是 语义版本，范围版本，分支版本，标签 或 提交的版本ID。

如果没有指定 具体版本号 或 范围版本号， Glide 会使用 语义版本号，并且提示要求你确认所需要的版本号

注意：

glide 会把下载下来的包缓存到  ~/.glide/cache 目录下 。

使用镜像 (glide mirror)

    [WARN]    Unable to checkout golang.org/x/crypto
    [ERROR]    Update failed for golang.org/x/crypto: Cannot detect VCS
    [ERROR]    Failed to do initial checkout of config: Cannot detect VCS

这几行信息估计很多人都是遇到过的。因为某些不可描述的原因，我们不能访问一些站点，导致很多Golang的依赖包不能通过go get下载。此时也就是glide大发神威的时候到了，可以通过配置将墙了的版本库 URL 映射到没被墙的 URL，甚至也可以映射到本地版本库。 将golang.org映射到github: 修改glide.yaml加入

    - package: golang.org/x/crypto

如果你的网络可以访问就不需要使用glide镜像功能，可以跳过。

    $ glide mirror set golang.org/x/crypto github.com/golang/crypto
    [INFO]    golang.org/x/crypto being set to github.com/golang/crypto
    [INFO]    mirrors.yaml written with changes
    $ glide up
    [INFO]    Loading mirrors from mirrors.yaml file
    [INFO]    Downloading dependencies. Please wait...
    [INFO]    --> Fetching updates for github.com/orcaman/concurrent-map
    [INFO]    --> Fetching golang.org/x/crypto
    [INFO]    --> Fetching updates for github.com/astaxie/beego
    [INFO]    --> Fetching updates for github.com/go-sql-driver/mysql
    [INFO]    --> Fetching updates for github.com/garyburd/redigo
    [INFO]    --> Fetching updates for github.com/coocood/freecache
    [INFO]    --> Fetching updates for github.com/bitly/go-simplejson
    [INFO]    --> Fetching updates for git.oschina.net/qiangmzsx/beegofreecache
    [INFO]    --> Fetching updates for github.com/smartystreets/goconvey
    [INFO]    --> Detected semantic version. Setting version for github.com/astaxie/beego to v1.8.3
    [INFO]    Resolving imports
    [INFO]    Downloading dependencies. Please wait...
    [INFO]    Setting references for remaining imports
    [INFO]    Exporting resolved dependencies...
    [INFO]    --> Exporting github.com/astaxie/beego
    [INFO]    --> Exporting github.com/coocood/freecache
    [INFO]    --> Exporting github.com/smartystreets/goconvey
    [INFO]    --> Exporting github.com/garyburd/redigo
    [INFO]    --> Exporting github.com/go-sql-driver/mysql
    [INFO]    --> Exporting github.com/bitly/go-simplejson
    [INFO]    --> Exporting github.com/orcaman/concurrent-map
    [INFO]    --> Exporting golang.org/x/crypto
    [INFO]    --> Exporting git.oschina.net/qiangmzsx/beegofreecache
    [INFO]    Replacing existing vendor dependencies
    [INFO]    Project relies on 8 dependencies.
    $ ll vendor/
    git.oschina.net
    github.com
    golang.org

终于看到golang.org啦！细心的你一定已经发现了，

    [INFO]    mirrors.yaml written with changes

说明执行glide mirror时候镜像配置写入到的是$HOME/.glide/mirrors.yaml中，打开看看。

    repos:
    - original: golang.org/x/crypto
      repo: github.com/golang/crypto

还可以映射到本地目录。 推荐大家可以去https://www.golangtc.com/download/package下载很多Golang类库。 现在我去下载了：https://www.golangtc.com/static/download/packages/golang.org.x.text.tar.gz，解压到本地目录/home/users/qiangmzsx/var/golang/golang.org/x/text。

    $ glide mirror set golang.org/x/text /home/users/qiangmzsx/var/golang/golang.org/x/text
    [INFO]    golang.org/x/text being set to /home/users/qiangmzsx/var/golang/golang.org/x/text
    [INFO]    mirrors.yaml written with changes
    $ glide up
    [INFO]    Loading mirrors from mirrors.yaml file
    [INFO]    Downloading dependencies. Please wait...
    [INFO]    --> Fetching golang.org/x/text
    [INFO]    --> Fetching updates for github.com/garyburd/redigo
    [INFO]    --> Fetching updates for git.oschina.net/qiangmzsx/beegofreecache
    [INFO]    --> Fetching updates for github.com/astaxie/beego
    [INFO]    --> Fetching updates for github.com/bitly/go-simplejson
    [INFO]    --> Fetching updates for github.com/go-sql-driver/mysql
    [INFO]    --> Fetching updates for github.com/coocood/freecache
    [INFO]    --> Fetching updates for github.com/orcaman/concurrent-map
    [INFO]    --> Fetching updates for golang.org/x/crypto
    [INFO]    --> Fetching updates for github.com/smartystreets/goconvey
    [INFO]    --> Detected semantic version. Setting version for github.com/astaxie/beego to v1.8.3
    [INFO]    Resolving imports
    [INFO]    Downloading dependencies. Please wait...
    [INFO]    Setting references for remaining imports
    [INFO]    Exporting resolved dependencies...
    [INFO]    --> Exporting github.com/astaxie/beego
    [INFO]    --> Exporting github.com/go-sql-driver/mysql
    [INFO]    --> Exporting github.com/bitly/go-simplejson
    [INFO]    --> Exporting github.com/coocood/freecache
    [INFO]    --> Exporting github.com/smartystreets/goconvey
    [INFO]    --> Exporting github.com/garyburd/redigo
    [INFO]    --> Exporting github.com/orcaman/concurrent-map
    [INFO]    --> Exporting golang.org/x/text
    [INFO]    --> Exporting golang.org/x/crypto
    [INFO]    --> Exporting git.oschina.net/qiangmzsx/beegofreecache
    [INFO]    Replacing existing vendor dependencies
    [INFO]    Project relies on 9 dependencies.

全局选项

运行glide，在最后就可以看到

    GLOBAL OPTIONS:
       --yaml value, -y value  Set a YAML configuration file. (default: "glide.yaml")
       --quiet, -q             Quiet (no info or debug messages)
       --debug                 Print debug verbose informational messages
       --home value            The location of Glide files (default: "/home/users/qiangmzsx/.glide") [$GLIDE_HOME]
       --tmp value             The temp directory to use. Defaults to systems temp [$GLIDE_TMP]
       --no-color              Turn off colored output for log messages
       --help, -h              show help
       --version, -v           print the version

如果大家想把glide的yaml文件换别的默认名称可以执行

     $ glide -y qiangmzsx.yaml

在官网中会看到一个GLIDE_HOME变量，该变量就是/home/users/qiangmzsx/.glide。 这个目录之前有提到过，除了包含有mirrors.yaml还有一个很重要的目录cache本地 cache,每次更新代码时， glide 都会在本地保存 cache，以备下次 glide install 使用 。

GLIDE_HOME可以通过如下命令修改。

     $ glide  --home /home/glide

查看glide.yaml中依赖名称

    $ glide name

查看依赖列表

    $ glide list

查看帮助

    $ glide help

参看glide版本信息

    $ glide --version

总结

总结一下，glide是一款功能丰富，完全满足需求的依赖管理工具，希望大家都能掌握。

---
title: "Golang 1.12 module的使用"
date: 2019-04-23T15:22:41+08:00
tags: [“go”,"golang"]
categories: [“go”,"golang"]
draft: false
---

## 起因
获取了一个新的golang项目，放入IDE后，代码使用go fmt的时候报了一堆的错误，并且代码内部都是不可用的。报错信息如下：
```
go: finding github.com/mattn/go-colorable v0.0.9
go: finding github.com/beorn7/perks v0.0.0-20180321164747-3a771d992973
go: finding github.com/pkg/term v0.0.0-20180730021639-bffc007b7fd5
go: finding github.com/gonum/internal v0.0.0-20180125090855-fda53f8d2571
go: finding github.com/sergi/go-diff v1.0.0
go: finding github.com/gonum/integrate v0.0.0-20180125090255-09c2f478329f
go: finding github.com/gonum/mathext v0.0.0-20180126232648-3ffefb3e36fc
go: finding github.com/pierrec/lz4 v2.0.5+incompatible
go: finding github.com/influxdata/line-protocol v0.0.0-20180522152040-32c6aa80de5e
go: finding github.com/kr/pretty v0.1.0
go: golang.org/x/sync@v0.0.0-20180314180146-1d60e4601c6f: unrecognized import path "golang.org/x/sync" (https fetch: Get https://golang.org/x/sync?go-get=1: dial tcp 216.239.37.1:443: i/o timeout)
go: golang.org/x/oauth2@v0.0.0-20181106182150-f42d05182288: unrecognized import path "golang.org/x/oauth2" (https fetch: Get https://golang.org/x/oauth2?go-get=1: dial tcp 216.239.37.1:443: i/o timeout)
```

## i/o timeout解决
所以Google搜索解决方案，由于Google被墙，所以我们需要设置GOPROXY来避免i/o timeout的错误  

### 关于 $GOPROXY

当我们使用go的时候，go默认会直接从代码库中去下载所需的相关依赖，GOPROXY 这个环境变量可以让我们控制自己从哪里去下载源代码，如果 GOPROXY 没有设置，go 会直接从代码库下载相关依赖代码。如果你像下面这样设置了这个环境变量，那么你就会通过 goproxy.io 下载所有的源代码。  

`export GOPROXY=https://goproxy.io`

你可以通过置空这个环境变量来关闭，export GOPROXY= 。  

以前大家执行 go get golang.org/x/net net代码库会下载到本地GOPATH中，以后有任何项目引用到了 golang.org/x/net 都不会再去下载这个代码库，因为本地GOPATH已经有了，哪怕版本不对，golang也会引用。但是随着 module 概念引入go语言，每个引入的 module 拥有了 version。随着代码库的不断更新迭代，大家即使是对同一个代码库的引用也可能用了不同的tag 或者 commit hash，基于这个现状，go1.12 的 module 会比以前更频繁的下载源代码。但是基于中国有中国特色的互联网，我们有时候很难get到我们需要的依赖源代码，进而导致项目编译失败，CI失败。于是，我们需要一个proxy。

![goproxy](http://ws1.sinaimg.cn/mw690/79b5b049gy1g2clp6kesqj20l40eq3yf.jpg)
goproxy.io 是一个开源项目，当用户请求一个依赖库时，如果它发现本地没有这份代码就会自动请求源，然后cache到本地，用户就可以从 goproxy.io 请求到数据。当然，这些都是在一个请求中完成的。goproxy.io 只支持 go module 模式。当用户执行 go get 命令时，会去检查$GOPROXY//@v/list这个文件中是否有用户想要获取的版本，如果有，就依次获取 $GOPROXY//@v/.info、$GOPROXY//@v/.mod、$GOPROXY//@v/.zip 等文件，如果没有就直接从源码库中去下载。

得益于 go module 在设计的时候非常重视安全这个领域，所以在启用了 go module 后，你会发现除了 go.mod 这个文件之外，还有一个 go.sum 文件，这个文件保存了每个依赖库的对应的hash值，来保证下载回来的代码库是正确的，不被人篡改的。同时， goproxy.io 也是个开源的项目。可以自行部署到自己的IDC中，因为公司内部自己的代码库 goproxy.io 是无法访问到的。开源地址：

https://github.com/goproxyio/goproxy

## go module介绍

go 1.11 有了对模块的实验性支持，大部分的子命令都知道如何处理一个模块，比如 run build install get list mod 子命令，第三方工具可能会支持的晚一些。到 go 1.12 会删除对 GOPATH 的支持，go get 命令也会变成只能获取模块，不能像现在这样直接获取一个裸包。

可以用环境变量 GO111MODULE 开启或关闭模块支持，它有三个可选值：off、on、auto，默认值是 auto。

GO111MODULE=off 无模块支持，go 会从 GOPATH 和 vendor 文件夹寻找包。
GO111MODULE=on 模块支持，go 会忽略 GOPATH 和 vendor 文件夹，只根据 go.mod 下载依赖。
GO111MODULE=auto 在 $GOPATH/src 外面且根目录有 go.mod 文件时，开启模块支持。
在使用模块的时候，GOPATH 是无意义的，不过它还是会把下载的依赖储存在 $GOPATH/src/mod 中，也会把 go install 的结果放在 $GOPATH/bin 中。

### 定义模块

模块根目录和其子目录的所有包构成模块，在根目录下存在 go.mod 文件，子目录会向着父目录、爷目录一直找到 go.mod 文件。

模块路径指模块根目录的导入路径，也是其他子目录导入路径的前缀。go.mod 文件第一行定义了模块路径，有了这一行才算作是一个模块。

go.mod 文件接下来的篇幅用来定义当前模块的依赖和依赖版本，也可以排除依赖和替换依赖。
```
cat ../github.com/mattn/go-colorable/go.mod
module github.com/mattn/go-colorable

require github.com/mattn/go-isatty v0.0.5
```

这个文件不用手写，可以用 go mod -init -module example.com/m 生成 go.mod 的第一行，文件的剩余部分也不用担心，在执行 go build、go test、go list 命令时会根据需要的依赖自动生成 require 语句。

官方建议经常维护这个文件，保持依赖项是干净的。对于国内用户来说，手动维护这个文件是必然的，因为你需要把 golang.org/x/text 替换成 github.com/golang/text 啊。不需要像以前那样以 hack 的方式替换 GOPATH 中的依赖，我一开始还保持着老思维，居然想要去替换模块的下载缓存，不过如果用 GOPROXY 功能也确实可以做到替换。

### go list 命令
go list -m 可以查看当前的依赖和版本

### go mod 命令
这个子命令用来处理 go.mod 文件，上一小节我们已经见过 go mod -init 了。

go mod edit -fmt  格式化 go.mod 文件。
go mod tidy       从 go.mod 删除不需要的依赖、新增需要的依赖，这个操作不会改变依赖版本。
go mod require=path@version 添加依赖或修改依赖版本，这里支持模糊匹配版本号，详情可以看下文 go get 的用法。
go mod vendor 生成 vendor 文件夹。
其他的自行 go help mod 查看。

## go get 命令
获取依赖的特定版本，用来升级和降级依赖。可以自动修改 go.mod 文件，而且依赖的依赖版本号也可能会变。在 go.mod 中使用 exclude 排除的包，不能 go get 下来。

与以前不同的是，新版 go get 可以在末尾加 @ 符号，用来指定版本。

它要求仓库必须用 v1.2.0 格式打 tag，像 v1.2 少个零都不行的，必须是语义化的、带 v 前缀的版本号。
```
go get github.com/gorilla/mux    # 匹配最新的一个 tag
go get github.com/gorilla/mux@latest    # 和上面一样
go get github.com/gorilla/mux@v1.6.2    # 匹配 v1.6.2
go get github.com/gorilla/mux@e3702bed2 # 匹配 v1.6.2
go get github.com/gorilla/mux@c856192   # 匹配 c85619274f5d
go get github.com/gorilla/mux@master    # 匹配 master 分支
```

latest 匹配最新的 tag。

v1.2.6 完整版本的写法。

v1、v1.2 匹配带这个前缀的最新版本，如果最新版是 1.2.7，它们会匹配 1.2.7。

c856192 版本 hash 前缀、分支名、无语义化的标签，在 go.mod 里都会会使用约定写法 v0.0.0-20180517173623-c85619274f5d，也被称作伪版本。

go get 可以模糊匹配版本号，但 go.mod 文件只体现完整的版本号，即 v1.2.0、v0.0.0-20180517173623-c85619274f5d，只不过不需要手写这么长的版本号，用 go get 或上文的 go mod -require 模糊匹配即可，它会把匹配到的完整版本号写进 go.mod 文件。

## go build 命令
go build -getmode=vendor 在开启模块支持的情况下，用这个可以退回到使用 vendor 的时代。


## 错误解决

### 格式化 go.mod 文件
`go mod edit -fmt`

### 从 go.mod 删除不需要的依赖、新增需要的依赖，
`go mod tidy`


## 参考链接

goproxy.io for Go modules <https://blog.csdn.net/ra681t58cjxsgckj31/article/details/82392870>  
用 golang 1.11 module 做项目版本管理 <https://www.jianshu.com/p/c5733da150c6?utm_campaign=studygolang.com&utm_medium=studygolang.com&utm_source=studygolang.com>


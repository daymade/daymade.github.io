
<!--more-->

# 我就是要用 go get!

众所周知, 使用 git 管理的 golang 代码可以直接使用 go get 下载到 $GOPATH 下面. 

例如:

```shell
$ go get github.com/Masterminds/glide
```

不过在实践中, 我们遇到了非 Github 的其他仓库, 如

- gitlab
- other private git servers

这时, 在使用 `go get` 下载时就会遇到各种各样的错误:

```shell
$ go get gitlab/bla/bla
error: 

$ go get gitlab/bla/bla
bla@gitlab.com's password: 
//blabla
error:
```

本文解决了非 Github 的其他仓库在使用 `go get` 时遇到的各种问题, 按照步骤操作后可以正常使用 `go get` 拉取各种 git 仓库的代码, 亲测有效提升各位 golang 开发者的幸福感 ^_^. 

## 解决方案

为了 `go get` 时使用 ssh 方式, 只要把 http/https 链接替换成 ssh 协议就好了.

手段是: 

```shell
git config .insteadOf
```

## Gitlab

在 shell 下执行:

```shell
$ git config --global url."git@gitlab.com:".insteadOf "http://gitlab.com/"
```

验证配置如下:

```shell
$ git config --global -l
url.git@gitlab.com:.insteadof=https://gitlab.com/
```

测试

```shell
$ go get gitlab/daymade/123
//it works!
```

## Private git servers

在一些实现的不太标准的私有 Git Server 中, 对 `go-import-tag`  的实现总是有些小问题, 这里不点名某些 `gogs` 啊啥的, 建议大家自己搭建的时候尽量绕道走. 

所以按照 gitlab 的方法去配置这些 git server 会发现行不通, 比如这样是无效的:

```shell
$ git config --global url."git@git.xxx.tm:".insteadOf "http://git.xxx.tm/"
```

测试一下:

```shell
$ go get git.xxx.tm/warrior/scavenger
// fxxk
```

这时需要我们在私有的 git 系统中, 比如 https://git.xxx.tm/warrior/scavenger 上看一下系统生成的 SSH Url:

```shell
ssh://git@git.xxx.tm:2233/warrior/scavenger.git
```

如法炮制:

```shell
$ git config --global url."ssh://git@git.xxx.tm:2233/".insteadOf "https://git.xxx.tm/"
```

验证配置如下:

```shell
$ git config --global -l
url.ssh://git@git.xxx.tm:2233/.insteadof=https://git.xxx.tm/
```

测试

```shell
$ go get git.xxx.tm/warrior/scavenger.git
//it works!
```

## Reference

https://help.github.com/articles/caching-your-github-password-in-git/

https://git-scm.com/docs/git-config#git-config-urlltbasegtinsteadOf
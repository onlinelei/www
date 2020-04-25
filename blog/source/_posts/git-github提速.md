---
title: github提速
date: 2019-12-09 23:31:06
tags: git
---
最近在github克隆代码的时候太慢，而且基本上是下载不下来。细心研究了下Git的代理设置。
<!--more-->
### 设置git代理
指定git代理地址为本地翻墙地址（我使用的是小飞机，windows默认是1080，mac默认是1086，具体可以在软件配置页面查到）注意这里默认你的机器已经科学上网。不同的翻墙软件端口可能不一样，这里我就没有继续研究了

```shell
git config --global http.proxy socks5://127.0.0.1:1086

```
下面看下效果，

```shell
# 配置代理前
git clone https://github.com/spring2go/staffjoy.git
Cloning into 'staffjoy'...
remote: Enumerating objects: 1785, done.
error: RPC failed; curl 18 transfer closed with outstanding read data remaining
fatal: the remote end hung up unexpectedly
fatal: early EOF
fatal: index-pack failed
--------------------------------------------------------------
# 配置代理后
git clone https://github.com/spring2go/staffjoy.git
Cloning into 'staffjoy'...
remote: Enumerating objects: 1785, done.
remote: Total 1785 (delta 0), reused 0 (delta 0), pack-reused 1785
Receiving objects: 100% (1785/1785), 50.74 MiB | 4.81 MiB/s, done.
Resolving deltas: 100% (515/515), done.

```

### 卸载代理
如果需要访问国内的git仓库，速度会变慢，可以把代理去掉。

```shell
git config --global --unset http.proxy
``` 


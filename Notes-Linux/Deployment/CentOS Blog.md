最近双 11 服务器挺便宜的，遂在腾讯云买了个 ECS 准备把博客搭起来玩玩，顺便复习一些 Linux 知识。

#### 在 CentOS 7.6 部署 Homebrew 

Homebrew 是一个在 IOS 上非常流行的开发环境部署工具，这里准备先在 Linux 操作一下，方便以后在 IOS 部署开发环境。

首先是官网 [Homebrew](https://brew.sh/index_zh-cn) ，根据官网的给出的指令：

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

很明显是使用 `curl` 工具下载 `shell` 进行安装，兴高采烈的准备 CV 之后等了 3 分钟左右，竟然发现 `curl` 报错了。

```sh
curl: (35) TCP connection reset by peer
```

大致可以看出来是因为 TCP 链接重置了，直接上 stackoverflow 进行一个查。

吐槽一下 sof 最近 Human verification 好频繁，不翻墙根本刷不出来 CAPTCHA 好痛苦。

但其实这个问题比较有地域特色，因为 homebrew 的资源地址在境外，所以导致 curl 的链接 reset。

在 csdn 找到了很好的解决方案：多装几遍，不错真的有用。

不过这里大概能猜到一些解决方法，大致和 git 下载速度慢等原因相似，更改 host 应该可以提速 90% 左右。

执行 homebrew 的安装指令后提示要安装 git，问题来了，这里用 yum install 自动只会安装 1.8.* 版本的 git，但是 homebrew 的官方文档指出需要最少 2.8.* 以上的支持，只能使用编译安装，但是 git 的依赖又非常多，使用编译安装总会缺东西。

这边在 csdn 看到一个比较好的解决方案：

```sh
# 惯例先卸载 
yum remove git 
# 安装 WANDisco re package 
yum install http://opensource.wandisco.com/centos/7/git/x86_64/wandisco-git-release-7-1.noarch.rpm
```

这个 WANDisco 是啥玩意儿，就不在这里展开说了，是一个开源的 repo 。


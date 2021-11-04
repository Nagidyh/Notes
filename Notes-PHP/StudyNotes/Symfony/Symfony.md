# Symfony

按照官方文档的进度记录笔记，尽量参考文档不搜百度。

[toc]

## 入门

### 安装 / 设置

主要记录一下两个环境的安装步骤，分别模拟生产环境和开发环境。

#### Linux

这里不得不要提一下 Linux 的版本问题，CentOS 虽然相对占用资源较少，但是 yum 是真的好难用啊，仓库版本都是较老的，安装 Homebrew 的时候一会儿 curl 版本太低一会儿 git 版本太低，折磨:cry:。

所以选择了支持非常丰富的 Ubuntu 20.2 ，不谈了 apt-get yyds !!!

Linux 安装大致分为以下几步，首先安装 [Homebrew](https://brew.sh/)，一个特别方便的包管理工具，适用 Linux 和 MacOS 。

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Ubuntu 20.2 更新后会自动给 git 启动 http.proxy，所以就会

```sh
fatal: unable to access 'https://github.com/homebrew/brew/'
```

这个时候只需要 unset 一下就行

```sh
git config --global --unset http.proxy
git config --global --unset https.proxy
```

Homebrew 下载大概率会失败，多试几次总会出来的。

大概率还会出现 update 更新失败的情况，也是网络问题。

```sh
Error: Fetching /home/linuxbrew/.linuxbrew/Homebrew failed!
 
Failed during: /home/linuxbrew/.linuxbrew/bin/brew update --force --quiet
```

需要先配置好 Homebrew 的国内镜像地址，记得先检查 `usr/local` 中 Homebrew 有没有安装成功。

```sh
# 使用中科院的镜像替代
git clone git://mirrors.ustc.edu.cn/homebrew-core.git/ /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core --depth=1

# 把homebrew-core的镜像地址也设为中科院的国内镜像
cd "$(brew --repo)"
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git

brew update
```

##### 使用国内镜像安装方法

```sh
rm Homebrew.sh ; wget https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh ; bash Homebrew.sh
```

国内服务器安装的一键式脚本。

#### Windows

两个方式，一个是在 [Symfony](https://symfony.com/) 官网找到安装程序，全局安装 symfony 就可以在命令行使用 symfony 命令配置了。这种方式大概率会因为网络问题导致根本安装不上 symfony ，所以还是建议选择第二种。

使用 composer 安装 symfony ， 用 composer 装就相对简单和灵活了。

Web 应用分支

```sh
# web application
composer create-project symfony/website-skeleton project_name

# micro service
composer create-project symfony/skeleton project_name
```



> 参考：[symfony 官方文档](https://symfony.com/doc)

### 第一个页面



### 路由 / URLs 生成器



### 控制器



### 模板



### 配置 / 环境变量

## 架构

### 请求 / 响应



### 内核



### 业务 / 依赖注入(DI)



### 事件



### 协议



### Bundles

## 基础

### 数据库 / Doctrine



### Forms



### Tests



### Session



### Cache



### Logger



### Errors / Debugging

## 高级

### Console



### Mailer / Emails



### Validation



### Messaging / Queues 



### Notifications



### Serialization



### Translation / i18n

## 安全

### Users



### Authentication / Firewalls



### Authorization / Voters 



### Passwords



### CSRF



### LDAP

## 前端

### Webpack Encore



### React.js



### Vue.js



### Bootstrap



### Web Assets



### WebLink

## 工具

### HTTP Client



### Files/ Filesystem



### Expression Language



### Locks



### Workflows



### String / Unicode



### UID / UUID



### YAML parser

## 生产

### Deployment



### Performance



### HTTP Cache



### SymfonyCloud
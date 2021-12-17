[TOC]

## 常用操作

### 常用命令

#### 版本回退

使用 `git commit` 将代码提交到版本库后，如果想要回退到上一次 commit 可以先使用 `git log` 查看提交记录，以获取版本号。

`git log --pretty=oneline` 可以简化打印的记录。

然后使用 `git reset --hard <commit_id>` 来返回对应的版本，其中 commit_id 可以简写，HEAD 代表当前版本，`HEAD~1` 或 `HEAD^` 代表上一个版本。

也可以使用 `git reflog` 来查看命令历史，以便回退版本。

#### 管理修改

![img](https://pic1.zhimg.com/80/v2-8e73605a803bd29ceacd86923e08f5f0_720w.jpg)

上图需要注意的几点是，add 只负责将工作区内容移入 暂存区，**commit 只负责将暂存区的内容提交到版本库。**

提交后，用 `git diff HEAD -- readme.txt` 命令可以查看工作区和版本库里面最新版本的区别。



![img](https://pic2.zhimg.com/80/v2-e95ac3baeb28734107f8bec468525661_720w.jpg)



#### 撤销修改

使用 `git checkout -- <file>` 命令可以撤销工作区的修改，有以下两种情况：

- `<file>` 未被放进暂存区，则把 `<file>` 还原为版本库 HEAD 的状态。
- 如果 `<file>` 进入了暂存区，且又在工作区对 `<file>` 进行了修改，则把 `<file>` 还原为暂存区的状态。

总之，就是让这个文件回到最近一次 `git commit` 或 `git add` 时的状态。



另外，可以使用 `git reset HEAD <file>` 指令来撤销已经 `add` 进入暂存区的修改。

#### 删除文件

如果将文件 add 到后删除，使用 status 可以对比工作区和版本库的状态。

此时，如果使用 `git rm <file>` 并 `git commit` ，那么此次删除就会被提交。

也可以使用，`git checkout -- <file>` 来恢复工作区的删除。

也就是说如果文件 commit 到了版本库，那么就是相对安全的。

### 远程仓库

#### 常用命令

一般操作是建立远程仓库，本地使用 git clone 拉取代码和仓库。

使用命令 `git remote -v` 查看 `origin` 地址。

需要更换远程仓库地址时，可以使用 `git remote add/rm origin` 命令来删除远程仓库地址。

使用 `git push origin master` 提交，使用 `git pull origin master` 拉取。  

#### SSH 与 HTTPS

一般的代码仓库在拉取代码时都需要鉴权，可以使用 HTTPS 与 SSH 两种方式，HTTPS 每次都需要使用账号和密码来认证，一般情况下建议使用 SSH 鉴权。

SSH 鉴权部署流程

使用命令 `ssh-keygen`



## Git 原理分析

### 三大分区

在本地使用 `git init` 命令初始化 git 仓库之后，就有了如下三个概念

- 工作区 Working Directory

  工作区就是 git 初始化后的根目录，也就是 .git 文件所在的目录，是用户可以直接修改的目录。

  所以在工作区中未被 add 到 stage 的文件，都不受 git 管理。

- 暂存区 Stage

  顾名思义，是数据暂存的区域。可以理解为数据在进入本地仓库之前，暂时存放的区域。

- 版本库 Commit History

  在使用了 commit 指令之后，就会将暂存区的代码放入版本库中，可以理解为一个本地的 Repository。
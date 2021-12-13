[TOC]

## 常用操作

![img](https://pic1.zhimg.com/80/v2-8e73605a803bd29ceacd86923e08f5f0_720w.jpg)

![img](https://pic2.zhimg.com/80/v2-e95ac3baeb28734107f8bec468525661_720w.jpg)

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
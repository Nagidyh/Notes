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

使用命令 `git remote -v` 查看 origin 地址。

需要更换远程仓库地址时，可以使用 `git remote add/rm origin` 命令来删除远程仓库地址。

使用 `git push origin master` 提交，使用 `git pull origin master` 拉取。  

#### SSH 与 HTTPS

一般的代码仓库在拉取代码时都需要鉴权，可以使用 HTTPS 与 SSH 两种方式，HTTPS 每次都需要使用账号和密码来认证，一般情况下建议使用 SSH 鉴权。

SSH 鉴权部署流程

使用命令 `ssh-keygen` 生成本机的公钥令牌，填写到对应的网络仓库平台，即可使用 SSH 协议通信。

### 分支管理

![git-br-initial](https://www.liaoxuefeng.com/files/attachments/919022325462368/0)

HEAD 是指向分支的，分支才指向提交，所以可以根据分支来管理不同工作进度下的提交。

![git-br-create](https://www.liaoxuefeng.com/files/attachments/919022363210080/l)

使用 `git checkout -b dev` 创建并将 HEAD 指向 dev 就完成了分支的切换和创建。

也可以使用 `git branch dev` 和 `git checkout dev / git switch dev` 来创建和切换分支。

![git-br-dev-fd](https://www.liaoxuefeng.com/files/attachments/919022387118368/l)

在 dev 分支上进行的提交不会影响 master 分支。

![git-br-ff-merge](https://www.liaoxuefeng.com/files/attachments/919022412005504/0)

在 dev 中完成开发后，可以使用 `git checkout master / git switch master` 和 `git merge dev` 来切换回 master 分支，并完成对 dev 分支的合并。

有一点值得注意，使用 `git merge dev` 操作合并分支，git 默认使用的是 fast-forward 方法，也就是直接将 master 指向 dev 的最新提交。

也可能使用 `git merge --no-ff -m "merge with no-ff" dev` 来禁用 `fast-forward`，Git就会在merge时生成一个新的 commit，这样，从分支历史上就可以看出分支信息。

当然如果在合并中有冲突的情况下，git 会提示需要本地手动处理冲突后再重新进行 `commit`，那么分支就会变为：

![git-br-conflict-merged](https://www.liaoxuefeng.com/files/attachments/919023031831104/0)

可以使用 `git log --graph --pretty=oneline --abbrev-commit` 更清楚的战展示分支结构。

删除分支使用 `git branch -d feature1` 指令，改为 `-D` 可以强制删除。



### 常用分支管理策略

#### 多人协作

一般建立公用仓库后，会建立一个新的开发分支 dev。默认创建的 master 用来发布，开发成员使用 `git remote add origin` 来建立和远程仓库的链接，可以使用 `git remote -v` 来查看当前链接的远程仓库，`git remote rm origin` 来删除远程链接。

如果有的新的成员需要加入到开发中，那么就可以使用如下指令：

`git remote add origin master git@address` 创建与远程仓库的链接

`git checkout -b dev origin/dev` 创建本地的 dev 分支并且和远程仓库的分支建立链接

`git checkout -b michael` 创建属于自己的开发分支，就可以基于自己的分支进行开发了。

`git checkout dev && git merge --no-ff -m 'something dev --michael'` 完成开发后合并到 Dev。

`git push origin dev` 来提交本地的开发。

![git-br-policy](https://www.liaoxuefeng.com/files/attachments/919023260793600/0) 

如果本地的分支，未和 origin 分支进行绑定，那么是无法使用以上命令提交的。

绑定有两种方式，一是在建立分支时，标注绑定的 origin 分支，如 `git checkout -b dev origin/dev`。

二是在建立分支 dev 后，需要将 dev 绑定到远程的 origin/dev 时，使用 `git branch --set-upstream-to=origin/dev dev` 来完成。

#### 本地开发

在本地创建了属于自己的开发分支后，可以使用一些本地分支的策略来提高开发的舒适度。

##### BUG

BUG 分支，如果项目正在开发中，而工作只进行到一半，还没法提交，预计完成还需1天时间。但是，必须在两个小时内修复该bug，怎么办？

现在就可以使用 git 提供的 `git stash` 指令来暂存工作区的内容。

现在就可以切换到需要修复的分支来修复 bug，如果 master 分支存在一个 bug。

现在可以使用如下指令来进行操作：

`git checkout master` 切换到 master 分支。

`git checkout -b issue-101` 创建一个修改 bug 的分支来修复 bug。

`git checkout master && git merge --no-ff issue-101` 合并到 master 分支。

`git checkout -d issue-101` 来删除 issue-101 分支。

这个时候就可以使用 `git stash list` 来查看暂存的工作区列表，选择刚刚我们暂存的工作区使用：

`git stash apply` 恢复，恢复后暂存的 stash 并不会被删除。

`git stash pop` 恢复，恢复后暂存的 stash 会被删除。

如果在 master 分支上修改完 bug 后，dev 分支还存在着 bug 要怎么办呢。

git 提供了一个方法 `git cherry-pick <commit_id>` 来复制刚刚在 master 的 commit 到当前的分支。

此时我们就可以使用 `git checkout dev && git cherry-pick commit_id` 来复制刚刚在 master 的 commit。

##### Feature

如果此时，有新的功能需要加入到项目中。创建一个新的分支来开发功能，并且在开发完成后将其删除。这样我们在本地测试时，就不会使主分支被一些实验性代码污染。

可以使用如下指令

`git checkout -b feature-vulcan` 创建并切换到 feature-vulcan 分支。

`git checkout dev && git merge --no-ff -m 'new feature' && git checkout -d feature-vulcan` 合并分支，并删除 feature 分支。

但是如果在开发中，这个 Feature 他变成了一个 BUG 或是需要被废弃，不合并到主分支即可。

即使已经合并到了 dev 分支，使用 --no-ff 后会建立新的分支，只需要将 HEAD 回退到上词正常的 commit 就好。 

### 命令集：

![img](https://notes-image-host.oss-cn-beijing.aliyuncs.com/img/git-cheat-sheet.png)

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
# Git 简单实践

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

## 实践案例

### 协作架构

构建公共仓库，分为 master 分支与 dev 分支，所有开发者的开发和合并都应该在 dev 分支进行，直到迭代完成再在 master 分支进行发布。master 是受保护的，也就是不能够直接对 master 分支进行推送。当迭代完成、并进行过完整的测试后，才由项目经理将 dev 分支合并到 master 进行版本的发布。



开发人员在本地拉取 dev 分支并构建自己的 feature 分支或 issue 分支进行开发工作，开发完成后再将开发工作的与 dev 分支合并，此时可以使用 coding 等仓库构建工具将此次合并提交给同事进行 Code Review，以确保代码的规范性。

### 回滚与找回的艺术

#### 常见回滚场景

##### 仅在工作区修改

文件在工作区被修改，还未提交到暂存区和本地版本库时，可以很轻松的使用 `git checkout -- <file>` 来回滚这部分修改。

需要注意的是：**因为这部分内容没有被 Git 托管过，所以 Git 无法追踪其历史，一旦回退直接就被丢弃了。**

用 `git status` 查看，还没有提交到暂存区的修改出现在 “Changes not staged for commit:” 部分。

##### 已 add 但未 commit

使用 `git add` 添加到暂存区的修改，还没有 commit 时，可以使用 `git reset HEAD <file>` 来回滚。

回滚后，**工作区会保留文件的改动**，可以重新编辑再提交，或使用 `git checkout -- <file>` 彻底丢弃修改。

##### 已 commit 但未 push

已经提交到了本地版本库，但未推送到远程库的修改，可以通过：

`git reset <commit_id>` 或 `git reset --hard <commit_id>` 来回滚。

需要注意的是，使用 `git reset` 指令回滚 commit，**那么在 `<commit_id>` 之后的 commit 都将会被废弃。**

`git reset` 默认会将被丢弃的记录所改动的文件保留在工作区中，以便重新编辑和再提交。加上 `--hard` 选项则不保留这部分内容，需谨慎使用。

##### 修改本地最近一次 commit 

有时 commit 之后发现刚才没改全，想再次修改后仍记录在一个 commit 里。利用 “git reset” 可达到这个目的，不过，Git 还提供了更简便的方法来修改最近一次 commit。

命令格式如下：

`git commit --amend [ -m <commit说明> ]`

如果命令中不加`-m <commit说明>`部分，则 Git 拉起编辑器来输入日志说明。

**请注意，”git commit –amend” 只可用于修改本地未 push 的 commit，不要改动已 push 的 commit！**

##### 已 push 到远端时

**注意！此时不能用 “git reset”，需要用 “git revert”！**
**注意！此时不能用 “git reset”，需要用 “git revert”！**
**注意！此时不能用 “git reset”，需要用 “git revert”！**

之所以不能在远程仓库中使用 `git reset` 是因为使用 reset 会抹掉历史，用在远程仓库中会带来各种各样的问题。而使用 `git revert` 用于回滚某次提交时，会生成的新的提交而不会抹掉历史。

##### git reset 和 git revert 的区别 ***

分支初始状态如下：

![img](https://notes-image-host.oss-cn-beijing.aliyuncs.com/img/20210408195032.png)

如果执行 `git reset B`，工作区会指向 `B`，其后的提交（C、D）被丢弃。

![img](https://notes-image-host.oss-cn-beijing.aliyuncs.com/img/20210408195045.png)

此时如果做一次新提交生成 `C1`，`C1`跟 C、D 没有关联，也就无法恢复C、D了。

![img](https://notes-image-host.oss-cn-beijing.aliyuncs.com/img/20210408195059.png)

如果执行 `git revert B`，回滚了 `B` 提交的内容后生成一个新 commit `E`，原有的历史不会被修改。

![img](https://notes-image-host.oss-cn-beijing.aliyuncs.com/img/20210408195114.png)

#### 找回已删除内容

虽说 Git 是一款强大的版本管理工具，一般来说，提交到代码库的内容不用担心丢失，然而某些特殊情况下仍免不了要做抢救找回，例如不恰当的 reset、错删分支等。这就是 `git reflog`派上用场的时候了。

“git reflog”是恢复本地历史的强力工具，几乎可以恢复所有本地记录，例如被 reset 丢弃掉的 commit、被删掉的分支等，称得上代码找回的“最后一根救命稻草”。

然而需要注意，**并非真正所有记录”git reflog”都能够恢复**，有些情况仍然无能为力：

1. **非本地操作的记录**
   “git reflog”能管理的是本地工作区操作记录，非本地（如其他人或在其他机器上）的记录它就无从知晓了。
2. **未 commit 的内容**
   例如只在工作区或暂存区被回滚的内容（git checkout – 文件 或 git reset HEAD 文件）。
3. **太久远的内容**
   “git reflog”保留的记录有一定时间限制（默认 90 天），超时的会被自动清理。另外如果主动执行清理命令也会提前清理掉。

##### 使用 Reflog 恢复到特定 commit

一个典型的场景是使用 reset 进行回滚，之后发现滚错了，要恢复到另一个 commit 的状态。

可以使用 `git reflog` 来查看 commit 操作历史，找到目标 commit，再通过 reset 恢复到目标 commit。

通过这个示例我们还可以看到**清晰、有意义的 commit log 非常有帮助**。假如 commit 日志都是”update”、”fix”这类无明确意义的说明，那么即使有”git reflog”这样的工具，想找回目标内容也是一件艰苦的事。

##### Reflog - 恢复特定 commit 中的某个文件

场景：执行 reset 进行回滚，之后发现丢弃的 commit 中部分文件是需要的。
解决方法：通过 reflog 找到目标 commit，再通过以下命令恢复目标 commit 中的特定文件。

`git checkout <目标 commit> -- <文件>`

##### Reflog - 找回本地误删的分支

场景：用”git branch -D”删除本地分支，后发现删错了，上面还有未合并内容！
解决方法：通过 reflog 找到分支被删前的 commit，基于目标 commit 重建分支。

`git branch <分支名> <目标commit>`

Reflog 记录中，”to <分支名>”（如 moving from master to dev/pilot-001） 到切换到其他分支（如 moving from dev/pilot-001 to master）之间的 commit 记录就是分支上的改动，从中选择需要的 commit 重建分支。

##### 找回合流后删除的分支

>作为 Git 优秀实践之一，开发分支合流之后即可删掉，以保持代码库整洁，只保留活跃的分支。
>一些同学合流后仍保留着分支，主要出于“分支以后可能还用得到”的想法。其实大可不必，已合入主干的内容不必担心丢失，随时可以找回，包括从特定 commit 重建开发分支。并且，实际需要用到旧开发分支的情况真的很少，一般来说，即使功能有 bug，也是基于主干拉出新分支来修复和验证。

假如要重建已合流分支，可通过主干历史找到分支合并记录，进而找到分支节点，基于该 commit 新建分支，例如：

`git branch dev/feature-abc 1f85427`

#### 关于代码回滚的一些建议

以下是关于特定命令的使用建议：

[![img](https://notes-image-host.oss-cn-beijing.aliyuncs.com/img/20210408200046.png)](https://help-assets.codehub.cn/enterprise/20210408200046.png)

此外，总体来讲，回滚要谨填，不要过于依赖回滚功能，避免使用”git push -f”。正如某哲人所说：**如果用到”git push -f”，你肯定哪里做错了！**

#### .gitignore 

在本地或测试服务器部署代码时经常遇到本地环境有一些特殊文件时，比如 vendor 等。

可以使用 .gitignore 忽略部分文件。

如果需要删除现有版本库中的文件，需要先将文件加入 .gitignore 文件中，并且使用命令：

`git rm <file> --cache`

`git rm -r <file> --cache` 删除版本库文件，并保留本地文件。

使用指令后，需要重新 commit 并 push 才能更新远程仓库。

#### 测试环境部署

通常测试环境都会有一两个环境变量文件，在建立本地库 remote origin pull 时，会报错：
`fatal: refusing to merge unrelated histories`

使用参数

`--allow-unrelated-histories` 可以忽略提交历史

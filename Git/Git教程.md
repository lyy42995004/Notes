# Git 常用命令

## 仓库

```bash
# 在当前目录新建一个Git代码库
$ git init

# 新建一个目录，将其初始化为Git代码库
$ git init [project-name]

# 下载一个项目和它的整个代码历史
$ git clone [url]
```

## 配置

```bash
# 显示当前的Git配置
$ git config --list

# 编辑Git配置文件
$ git config -e [--global]

# 设置提交代码时的用户信息
$ git config [--global] user.name "[name]"
$ git config [--global] user.email "[email address]"
```

## 增加/删除文件

```bash
# 添加指定文件到暂存区
$ git add [file1] [file2] ...

# 添加指定目录到暂存区，包括子目录
$ git add [dir]

# 添加当前目录的所有文件到暂存区
$ git add .

# 添加每个变化前，都会要求确认
# 对于同一个文件的多处变化，可以实现分次提交
$ git add -p

# 删除工作区文件，并且将这次删除放入暂存区
$ git rm [file1] [file2] ...

# 停止追踪指定文件，但该文件会保留在工作区
$ git rm --cached [file]

# 改名文件，并且将这个改名放入暂存区
$ git mv [file-original] [file-renamed]
```

## 代码提交

```bash
# 提交暂存区到仓库区
$ git commit -m [message]

# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]

# 提交工作区自上次commit之后的变化，直接到仓库区
$ git commit -a

# 提交时显示所有diff信息
$ git commit -v

# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
$ git commit --amend -m [message]

# 重做上一次commit，并包括指定文件的新变化
$ git commit --amend [file1] [file2] ...
```

**分支**

```bash
# 列出所有本地分支
$ git branch

# 列出所有远程分支
$ git branch -r

# 列出所有本地分支和远程分支
$ git branch -a

# 新建一个分支，但依然停留在当前分支
$ git branch [branch-name]

# 新建一个分支，并切换到该分支
$ git checkout -b [branch]

# 新建一个分支，指向指定commit
$ git branch [branch] [commit]

# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch] [remote-branch]

# 切换到指定分支，并更新工作区
$ git checkout [branch-name]

# 切换到上一个分支
$ git checkout -

# 建立追踪关系，在现有分支与指定的远程分支之间
$ git branch --set-upstream [branch] [remote-branch]

# 合并指定分支到当前分支
$ git merge [branch]

# 选择一个commit，合并进当前分支
$ git cherry-pick [commit]

# 删除分支
$ git branch -d [branch-name]

# 删除远程分支
$ git push origin --delete [branch-name]
$ git branch -dr [remote/branch]
```

**标签**

```bash
# 列出所有tag
$ git tag

# 新建一个tag在当前commit
$ git tag [tag]

# 新建一个tag在指定commit
$ git tag [tag] [commit]

# 删除本地tag
$ git tag -d [tag]

# 删除远程tag
$ git push origin :refs/tags/[tagName]

# 查看tag信息
$ git show [tag]

# 提交指定tag
$ git push [remote] [tag]

# 提交所有tag
$ git push [remote] --tags

# 新建一个分支，指向某个tag
$ git checkout -b [branch] [tag]
```

**查看信息**

```bash
# 显示有变更的文件
$ git status

# 显示当前分支的版本历史
$ git log

# 显示commit历史，以及每次commit发生变更的文件
$ git log --stat

# 搜索提交历史，根据关键词
$ git log -S [keyword]

# 显示某个commit之后的所有变动，每个commit占据一行
$ git log [tag] HEAD --pretty=format:%s

# 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
$ git log [tag] HEAD --grep feature

# 显示某个文件的版本历史，包括文件改名
$ git log --follow [file]
$ git whatchanged [file]

# 显示指定文件相关的每一次diff
$ git log -p [file]

# 显示过去5次提交
$ git log -5 --pretty --oneline

# 显示所有提交过的用户，按提交次数排序
$ git shortlog -sn

# 显示指定文件是什么人在什么时间修改过
$ git blame [file]

# 显示暂存区和工作区的差异
$ git diff

# 显示暂存区和上一个commit的差异
$ git diff --cached [file]

# 显示工作区与当前分支最新commit之间的差异
$ git diff HEAD

# 显示两次提交之间的差异
$ git diff [first-branch]...[second-branch]

# 显示今天你写了多少行代码
$ git diff --shortstat "@{0 day ago}"

# 显示某次提交的元数据和内容变化
$ git show [commit]

# 显示某次提交发生变化的文件
$ git show --name-only [commit]

# 显示某次提交时，某个文件的内容
$ git show [commit]:[filename]

# 显示当前分支的最近几次提交
$ git reflog
```

**远程同步**

```bash
# 下载远程仓库的所有变动
$ git fetch [remote]

# 显示所有远程仓库
$ git remote -v

# 显示某个远程仓库的信息
$ git remote show [remote]

# 增加一个新的远程仓库，并命名
$ git remote add [shortname] [url]

# 取回远程仓库的变化，并与本地分支合并
$ git pull [remote] [branch]

# 上传本地指定分支到远程仓库
$ git push [remote] [branch]

# 强行推送当前分支到远程仓库，即使有冲突
$ git push [remote] --force

# 推送所有分支到远程仓库
$ git push [remote] --all
```

**撤销**

```bash
# 恢复暂存区的指定文件到工作区
$ git checkout [file]

# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit] [file]

# 恢复暂存区的所有文件到工作区
$ git checkout .

# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
$ git reset [file]

# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard

# 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
$ git reset [commit]

# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
$ git reset --hard [commit]

# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
$ git reset --keep [commit]

# 新建一个commit，用来撤销指定commit
# 后者的所有变化都将被前者抵消，并且应用到当前分支
$ git revert [commit]

# 暂时将未提交的变化移除，稍后再移入
$ git stash
$ git stash pop
```

**其他**

```bash
# 生成一个可供发布的压缩包
$ git archive
```

# 核心原理

![](https://www.baeldung-cn.com/wp-content/uploads/2021/11/git_workflow.png)

## Git 的核心原理：三个工作区域与数据流转

Git 的核心可以概括为：**它管理的是文件的变更（changes），而非文件本身。** 为了实现这种管理，Git 设立了三个核心工作区域：

1. **工作目录 (Working Directory)**
   - **是什么**：就是你电脑上能看到和编辑的普通文件和文件夹。你在这里进行所有的代码修改。
   - **特点**：这里的文件状态可能是`已修改(modified)`或`未跟踪(untracked)`。
2. **暂存区 (Index / Staging Area)**
   - **是什么**：一个独特的中间区域，像一个“购物车”或“准备台”。它连接着工作目录和本地仓库。
   - **核心作用**：**让你可以精确地控制哪些变更（可以是某个文件的部分改动）将被包含到下一次提交中。** 这是 Git 相比其他版本控制系统非常强大的一个特性。
3. **本地仓库 (Local Repository)**
   - **是什么**：位于你的电脑上的完整项目历史数据库（隐藏的`.git`目录）。它存储了一系列的**提交(commit)**。
   - **特点**：一旦代码提交到这里，就意味着你正式创建了一个新版本的历史快照。这个快照是**永久性**的，几乎不会被丢失。

**第四个区域：远程仓库 (Remote Repository)**

- 它是位于远端服务器（如 GitHub, GitLab）上的项目仓库，用于团队协作和备份。在 Git 的设计中，它本质上是另一个仓库。

------

## 核心操作流程：连接三个区域

图片中的箭头和命令，生动地描绘了数据在这三个区域之间流转的过程，这就是 Git 的工作流。

1. 从工作目录到暂存区 (`git add`)

- **操作**：当你修改了工作目录的文件后，使用 `git add <file>` 命令。
- **原理**：这个命令将文件的**当前快照**（而不仅仅是差异）从工作目录添加到暂存区。此时，变更被“暂存”了起来，准备进入下一次提交。
- **对应图中**：`Changes` 部分，从 `Working Directory` 指向 `Index` 的黄色箭头。

2. 从暂存区到本地仓库 (`git commit`)

- **操作**：使用 `git commit -m "message"` 命令。
- **原理**：这个命令将暂存区里的所有内容**永久性地**存储到本地仓库，生成一个新的提交对象（Commit Object）。这个提交对象包含指向当前暂存内容快照的指针、提交信息、作者以及指向上一个提交的指针，从而形成一条历史链。
- **对应图中**：`Changes` 部分，从 `Index` 指向 `Local Repository` 的黄色箭头。

**小结：`Changes` 流程 (`git add` -> `git commit`)**
 这就是你本地创建版本历史的核心循环。它让你可以小步、频繁地提交，而不是一次性提交所有改动。

3. 与远程仓库的同步 (`git push`, `git fetch`, `git pull`)

- **`git push`**
  - **操作/原理**：将你**本地仓库**中的新提交上传到**远程仓库**，让其他人能看到你的更改。
  - **对应图中**：`Sync` 部分，从 `Local Repository` 指向 `Remote Repository` 的箭头。
- **`git fetch`**
  - **操作/原理**：去**远程仓库**查询有哪些你本地还没有的新提交，并将这些新提交下载到你的本地仓库。**注意**：它只会更新你的本地仓库，但不会动你的工作目录。你需要再手动合并。
- **`git pull`**
  - **操作/原理**：`git pull` = `git fetch` + `git merge`。它一次性完成两个动作：先抓取(`fetch`)远程的新提交，然后立即尝试将其**合并(merge)** 到你当前的工作目录中。
  - **对应图中**：`Sync` 部分，从 `Remote Repository` 指向 `Local Repository` 以及后续的箭头。

4. 本地分支与合并 (`git checkout`, `git merge`)

- **`git checkout`**
  - **操作/原理**：用于切换分支或恢复工作目录的文件。当你切换分支时，Git 实际上是用该分支所指向的提交快照，来**更新你的工作目录和暂存区**。这解释了为什么你切换分支后，文件内容会立刻改变。
- **`git merge`**
  - **操作/原理**：将两个分支的历史合并在一起。它会在当前分支上创建一个新的“合并提交”，这个提交有两个父提交，将两条开发线汇合到一起。图片中它可能被展示为一种将变更应用到工作目录的方式。

**总结**

Git 的核心原理就是围绕 **三个核心区域（工作目录、暂存区、本地仓库）**，通过一系列命令（`add`, `commit`, `push`, `pull`等）在这些区域之间**移动和存储文件的变更快照**。

- **`git add`** 是“我要开始跟踪这些变更了”。
- **`git commit`** 是“我确认这个变更，并将其永久记录到历史中”。
- **`git push`/`pull`** 是“让我和远端的队友同步一下历史记录”。

---

**教程推荐**：

[Git教程](https://git.mosong.cc/)：

[Git-Book](https://git-scm.com/book/zh/v2)：官方电子书，中文翻译版，内容特别多，深入学习可以看

B站视频推荐：[Git工作流和核心原理 | GitHub基本操作 | VS Code里使用Git和关联GitHub](https://www.bilibili.com/video/BV1r3411F7kn/)


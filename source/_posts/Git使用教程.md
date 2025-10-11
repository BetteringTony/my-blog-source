---
title: Git使用教程
date: 2025-10-11 20:17:40
categories: [通用教程]
tags: [Git]
---

> 本教程源于https://liaoxuefeng.com/books/git/introduction/index.html
>
> 做了进一步精简，保留命令操作以及使用场景，可将其视为字典，方便查阅

## 1、Git 结构总览

Git结构图：

![git-repo](https://liaoxuefeng.com/books/git/time-travel/working-stage/repo.png)

主要概念：工作区、暂存区、本地仓库、分支、HEAD。

- **工作区**：存放实际文件的地方，用于开发和修改。
- **暂存区**：临时存储即将提交的变更。
- **本地仓库**：存储项目的完整历史记录。
- **分支**：并行开发的工具，用于隔离不同功能或任务的开发。
- **HEAD**：指向当前分支或提交的指针，用于跟踪当前状态。

## 2、常用命令

### 2.1、Git配置

- `git config --global user.name "Your Name"`
  `git config --global user.email "email@example.com"`

  全局配置，对该操作系统用户账户下的所有 Git 仓库生效，可以设置常用的用户名和邮箱，这样新建仓库时自动使用，不必每次都进行本地配置。优先级低于本地配置。

- `git config user.name "Your Name"`
  `git config user.email "email@example.com"`

  本地配置，当前这一个 Git 仓库生效，在不同项目中用不同身份提交代码，比如一个公司项目用公司邮箱，个人项目用私人邮箱。优先级高于全局配置。

### 2.2、创建仓库

- `git init`

  将某个目录变成Git可以管理的仓库，在仓库目录下，有一个隐藏的`.git`文件夹，这个目录是Git来跟踪管理版本库的，一般不需要去改动其中内容。

- `git status`

  显示当前工作区和暂存区的状态，告诉你当前所在的分支名称。显示工作区中哪些文件被修改过，哪些文件是未被跟踪的新文件。显示哪些文件已经被添加到暂存区，等待提交。如果当前分支有远程分支关联，还会显示当前分支与远程分支的差异情况。

- `git add`

  将文件从工作区（Working Directory）添加到暂存区（Staging Area），为后续提交做准备。可以一次添加单个文件，也可以添加单个文件，还可以一次提交所有文件。

- `git commit -m "有意义的提交信息"`

  将暂存区的内容提交到本地仓库。只会提交暂存区的内容，如果工作区中存在未添加到暂存区的更改，这些更改不会被提交。

- `git diff`

  主要是显示文件之间的差异。可以查看工作区与暂存区之间的差异、暂存区与上一次提交之间的差异、任意两个提交之间的差异、分支之间的差异。

### 2.3、时光机穿梭

- `git log`

  显示项目的提交历史。会列出从最近到最远的提交记录，包括提交的哈希值、提交者、提交时间、提交信息等，可以追踪代码的变更历史，了解项目的演变过程。

- `git reset`

  将当前分支的 HEAD 重置到某个指定的提交版本，同时可以选择性地调整暂存区和工作区的状态。HEAD 指向当前分支，可以简单地理解为当前分支的当前提交版本。HEAD~1表示前一个版本，其余版本类似表示。参数`--soft`：将 HEAD 重置到指定的提交，暂存区和工作区的内容都不会改变。参数`--mixed`：将 HEAD 重置到指定的提交，暂存区的内容会被重置，但工作区的内容不会改变。参数`--hard`：将 HEAD 重置到指定的提交，同时清空暂存区和工作区的内容，主要适用于想完全撤销所有更改，回到某个特定的提交时。

- `git reflog`

  记录对仓库的引用（如分支、标签等）的所有更新操作。每次对分支进行操作（如提交、重置、合并等），`reflog` 都会记录下当前引用的哈希值和操作描述。这使得你可以随时查看引用的历史记录，并恢复到之前的任何状态。

- `git restore <file>`

  工作区的某个文件进行了修改，但是修改还没有添加到暂存区，那么可以通过该命令将此次修改从工作区撤除，回到初始内容。

- `git restore --staged <file>`

  工作区的某个文件进行了修改，同时该修改已经添加到了暂存区，那么可以通过该命令将此次修改从暂存区撤除，然后可以选择进一步通过`git restore <file>`将修改从工作区撤除。

- `git rm`

  从工作区（文件系统）中删除文件，同时将删除操作记录到暂存区，以便后续提交。可以确保文件的删除操作被版本控制跟踪。如果想从版本控制中移除某些文件（不再被 Git 跟踪），但仍然保留文件在工作区中，则可可以使用`git rm --cached <file>`，如果不小心误删文件，可以通过 `git reflog` 找到之前的提交，利用`git checkout <commit-hash> -- <file>`恢复文件。

### 2.4、远程仓库

- `git remote`

  用于管理远程仓库的命令。允许查看、添加、修改和删除与本地仓库关联的远程仓库地址。`git remote -v`，查看远程仓库的详细信息（包括地址）。`git remote add <name> <url>`，添加一个新的远程仓库。`<name>` 是远程仓库的名称（通常为 `origin`），`<url>` 是远程仓库的地址。

- `git push`

  将本地分支的提交历史推送到远程仓库。它会将本地分支的最新提交同步到远程分支，更新远程仓库的状态。`git push <remote> <branch>`将本地当前分支的提交同步到指定远程仓库的某个分支。使用 `-u` 参数可以设置上游分支（即本地分支与远程分支的关联关系）：`git push -u <remote> <branch>`，之后可以直接使用 `git push` 和 `git pull`，而无需指定远程仓库和分支。

- `git pull`

  从远程仓库获取最新的提交，并将这些更改合并到当前本地分支。结合了 `git fetch`（获取远程仓库的最新更改）和 `git merge`（将远程更改合并到本地分支）的操作。`git pull <remote> <branch>`可以将指定远程仓库的某个分支拉取更改并合并到本地当前分支。如果存在关联，可以简写为`git pull`。

### 2.5、分支管理

- `git branch`

  列出所有分支，当前分支前面会标一个`*`号。

- `git branch <name>`

  创建一个新分支，但不会自动切换。

- `git switch <name>`

  切换到指定分支，该分支必须是已经存在的。如果想要不存在分支时自动创建且自动切换到该分支，可以使用`git switch -c <name>`。

- `git branch -d <name>`

  删除指定的本地分支。如果该分支更改尚未合并到其他分支，可能会丢失数据，Git 会阻止删除。如果你想强制删除未合并的分支，需要使用`-D`参数。

- `git merge <name>`

  将指定分支 `<branch>` 的更改合并到当前分支。如果合并发生冲突，需要打开冲突文件，手动解决冲突之后，再进行add、commit操作，最终成功合并。合并分支时，加上`--no-ff`参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而`fast forward`合并就看不出来曾经做过合并。

- `git stash`

  用于暂存（stash）当前工作区和暂存区的更改的命令。允许将未完成的更改临时保存起来，以便切换到其他分支或进行其他操作，而不用担心丢失当前的更改。完成其他操作后，可以重新应用这些暂存的更改。

### 2.6、标签管理

- `git tag <name>`

  创建一个轻量级标签，标记当前提交。通过标签，可以快速定位到某个特定版本的代码，如果想创建一个标签标记指定提交，可以使用：`git tag <tag-name> <commit-hash>`。

- `git tag -d <tagname>`

  删除一个本地标签。

- `git push origin <tagname>`

  推送一个本地标签到远程仓库。

## 3、常见使用场景及流程

在实际开发中，应该按照几个基本原则进行分支管理：

首先，`master`分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；干活都在`dev`分支上，也就是说，`dev`分支是不稳定的，到某个时候，比如1.0版本发布时，再把`dev`分支合并到`master`上，在`master`分支发布1.0版本；每个开发者都在`dev`分支上干活，每个人都有自己的分支，时不时地往`dev`分支上合并就可以了。所以，团队合作的分支看起来就像这样：

![git-br-policy](https://liaoxuefeng.com/books/git/branch/policy/branches.png)

相应的命令操作：

### 1. 创建 dev 分支

假设项目初始时只有一个`master`分支，并且`master`分支是稳定的。需要从`master`分支创建一个`dev`分支，作为开发的主分支。

```bash
# 切换到master分支
git checkout master

# 确保master分支是最新的
git pull origin master

# 从master分支创建dev分支
git checkout -b dev

# 将dev分支推送到远程仓库
git push -u origin dev
```

### 2. 开发者创建个人分支

每个开发者在开始开发新功能或修复问题时，都应该从`dev`分支创建自己的个人分支。这样可以避免直接在`dev`分支上进行开发，减少冲突。

**开发者A的操作：**

```bash
# 切换到dev分支
git checkout dev

# 确保dev分支是最新的
git pull origin dev

# 从dev分支创建个人分支，例如feature-A
git checkout -b feature-A

# 开始开发工作
# （进行代码修改、添加功能等）

# 完成开发后，将个人分支推送到远程仓库
git push -u origin feature-A
```

**开发者B的操作：**

```bash
# 切换到dev分支
git checkout dev

# 确保dev分支是最新的
git pull origin dev

# 从dev分支创建个人分支，例如feature-B
git checkout -b feature-B

# 开始开发工作
# （进行代码修改、添加功能等）

# 完成开发后，将个人分支推送到远程仓库
git push -u origin feature-B
```

### 3. 将个人分支合并到 dev 分支

当开发者的个人分支开发完成并通过测试后，需要将个人分支合并回`dev`分支。

**开发者A合并`feature-A`到`dev`：**

```bash
# 切换到本地dev分支
git checkout dev

# 确保dev分支是最新的
git pull origin dev

# 合并feature-A分支到dev分支
git merge feature-A

# 如果有冲突，解决冲突后继续合并
git add .
git commit

# 将合并后的dev分支推送到远程仓库
git push origin dev
```

**开发者B合并`feature-B`到`dev`：**

```bash
# 切换到本地dev分支
git checkout dev

# 确保dev分支是最新的
git pull origin dev

# 合并feature-B分支到dev分支
git merge feature-B

# 如果有冲突，解决冲突后继续合并
git add .
git commit

# 将合并后的dev分支推送到远程仓库
git push origin dev
```

在多个开发者向同一个远程分支push时，可能会遇到一些冲突，常见流程如下：

1. 首先，可以尝试用`git push origin <branch-name>`推送自己的修改；
2. 如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；
3. 如果合并有冲突，则解决冲突，并先在本地提交；
4. 没有冲突或者解决掉冲突后，再用`git push origin <branch-name>`推送就能成功！

如果`git pull`提示`no tracking information`，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream-to <branch-name> origin/<branch-name>`。

### 4. 发布版本时将 dev 分支合并到 master 分支

当`dev`分支上的开发工作完成，且经过测试确认稳定后，可以将`dev`分支合并到`master`分支，发布新版本。

```bash
# 切换到master分支
git checkout master

# 确保master分支是最新的
git pull origin master

# 合并dev分支到master分支
git merge dev

# 如果有冲突，解决冲突后继续合并
git add .
git commit

# 将合并后的master分支推送到远程仓库
git push origin master

# 可以选择打标签，标记版本
git tag -a v1.0 -m "Release version 1.0"
git push origin v1.0
```

### 5. 清理分支

在完成合并后，可以删除不再需要的个人分支，以保持仓库的整洁。

```bash
# 删除本地个人分支
git branch -d feature-A
git branch -d feature-B

# 删除远程个人分支
git push origin --delete feature-A
git push origin --delete feature-B
```


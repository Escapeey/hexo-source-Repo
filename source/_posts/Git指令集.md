---
title: Git指令集
tags: 工具
category: 工具
abbrlink: f38ad9ef
date: 2024-05-22 22:24:07
---
<img src="..\img\git\远程仓库操作.png" width="100%" height="100%" align="middle">

# 初始化设置

配置**用户名**

```bash
git config --global user.name "Your Name"
```

配置邮箱

```bash
git config --global user.email "mail@example.com"
```

**存储**配置

```bash
git config --global credential.helper store
```

# 创建仓库

**创建**一个新的本地仓库 （ 省略 `project-name ` 则在当前目录创建）

```bash
git init <project-name>
```

**克隆**一个远程仓库

```bash
git clone <url>
```

# 四个区域

### 工作区(Working Directory)

就是你在电脑里能**实际看到的目录**。

### 暂存区（**Stage/Index**）

暂存区也叫索引， 用来临时存放**未提交**的内容， 一般在 `.git` 目录下的 `index` 中。

### **本地仓库（**Repository）

Git在**本地的版本库**， 仓库信息存储在  `.git` 这个隐藏目录中。

### **远程仓库（**Remote Repository）

托管在**远程服务器**上的仓库。 常用的有GitHub、 GitLab、 Gitee。



## 对于区域和分支的问题

- 当对文件做出改动时，**工作区和暂存区**对所有分支都是可见的；

- 但当在某个分支提交后，则该改动**只能在该提交的分支查看**到，并且 `ls` 和 `ls-files` 都只能在**当前分支**中看到**该改动**，别的分支看不到

**查看**当前分支当前**工作区**的文件列表

```bash
ls
```

**查看**当前分支当前**暂存区**文件列表

```bash
git ls-files
```

# 文件状态

### 已修改（**Modified**）

**修改了**但是**没有保存**到暂存区的文件。

### 已暂存（**Staged**）

**修改后**已经**保存到暂存区**的文件。

### 已提交（Committed）

把暂存区的文件**提交到本地仓库**后的状态。

# 基本概念

- **`main / master`** 		默认主分支
- **`origin`** 				       默认远程仓库
- **`HEAD`** 			  		     指向**当前分支**的指针
- **`HEAD^`** 					     上一个版本
- **`HEAD~4`**			            上4个版本

 # 特殊文件

- **`.git`** 							Git仓库的**元数据**和对象数据库
- **`.gitignore`** 			   **忽略文件**，不需要提交到仓库的文件
- **`.gitattributes`** 	  指定文件的属性，比如换行符
- **`.gitkeep`** 				   使空目录被提交到仓库
- **`.gitmodules`**		      记录子模块的信息
- **`.gitconfig`**			    记录仓库的配置信息

# 添加和提交

**添加**一个文件到暂存区

```bash
git add <file>
```

添加所有文件到暂存区

```bash
git add .
```

**提交**所有暂存区的文件到仓库

```bash
git commit -m "message"
```

提交所有已修改的文件到仓库

```bash
git commit -am "message"
```

# 分支

查看所有**本地分支**，当前分支钱前面会有一个 `*` , `-r` 查看远程分支， `-a` 查看所有分支

```bash
git branch
```

**创建**一个新分支

```bash
git branch <branch-name>
```

**删除**一个已经合并的分支

```bash
git branch -d <branch-name>
```

**删除**一个分支，不管是否合并

```bash
git branch -D <branch-name>
```

**切换**到指定分支，**并更新**工作区

```bash
git checkout <branch-name>
```

**创建**一个新分支，**并切换**到该分支

```bash
git checkout -b <branch-name>
```

给当前的提交打上标签，通常用于版本发布

```bash
git tag <tag-name>
```

# 合并分支

合并分支a到分支b

`-no-ff` 参数表示禁用 `Fast forward` 模式，合并后的历史有分支，能看出曾经做过合并

`-ff` 参数表示使用 `Fast forward` 模式，合并后的历史会变成一条直线

```bash
git merge --no-ff -m "message" <branch-name>
```
<img src="..\img\git\合并分支1.png" width="100%" height="100%" align="middle">

```bash
git merge --ff -m "message" <branch-name>
```
<img src="..\img\git\合并分支2.png" width="100%" height="100%" align="middle">

合并&`squash`所有提交到一个提交
```bash
git merge --squash <branch-name>
```

`rebase` 不会产生新的提交，而是把当前分支的每一个提交都"复制"到目标分支上，然后再把当前分支指向目标分支

`merge` 会产生一个新的提交，这个提交有两个分支的所有修改

**Rebase**

`Rebase` 操作可以把本地未push的分支提交历史整理成直线，看起来更直观。但是，如果多人协作时，不要对已经推送到远程的分支执行 `Rebase` 操作

```bash
git checkout <dev>
git rebase <main>
```
<img src="..\img\git\合并分支3.png" width="100%" height="100%" align="middle">

# 撤销

移动一个文件到新的位置

```bash
git mv <file> <new-file>
```

从工作区和暂存区中删除一个文件，然后暂存删除操作

```bash
git rm <file>
```

只从暂存区中删除一个文件，工作区中的文件没有变化

```bash
git rm --cached <file>
```

恢复一个文件到之前的版本

```bash
git checkout <file> <commit-id>
```

提交一个新的提交，用来撤销指定的提交，后者的所有变化都将被前者抵消，并且应用到当前分支

```bash
git revert <commit-id>
```

重置当前分支的 `HEAD` 为之前的某个提交，并且删除所有之后的提交。 

- `--hard` 参数表示重置工作区和暂存区

- `--soft` 参数表示重置暂存区

- `--mixed` 参数表示重置工作区 

```bash
git reset --mixed <commit-id>
```

撤销暂存区的文件，重新放回工作区 （`git add` 的反向操作）

```bash
git restore --staged <file>
```

# 查看

列出还未提交的新的或修改的文件

```bash
git status
```

查看提交历史, `--oneline`可简略查看 

```bash
git log --oneline
```

查看未暂存的文件更新了哪些部分

```bash
git diff
```

查看两个提交之间的差异

```bash
git diff <commit-id> <commit-id>
```

# Stash

`Stash` 操作可以把当前工作现场"储藏"起来，等以后**恢复现场**后继续工作。

`-u` 参数表示把当前所有 **未跟踪** 的文件也一并存储

`-a` 参数表示把所有 **未跟踪** 的文件和 **忽略** 的文件也一并存储

`save` 参数表示存储的信息，可以不写

```bash
git stash save "message"
```

查看所有 `stash`

```bash
git stash list
```

恢复最近一次 `stash`

```bash
git stash pop
```

恢复指定的 `stash`, `stash@{2}`  表示第三个 `stash` , `stash@{0}` 表示最近的 `stash`

```bash
git stash pop stash@{2}
```

重新接受最近一次 `stash`

```bash
git stash apply
```

`pop` 和 `apply` 的区别时，`pop` 会把 `stash` 内容删除，而 `apply` 不会。

可以用 `git stash drop` 来删除 `stash`

```bash
git stash drop stash@{2}
```

删除所有 `stash`

```bash
git stash clear
```

# 远程仓库

添加远程仓库

```bash
git remote add <remote-name> <remote-url>
```

查看远程仓库

```bash
git remote -v
```

删除远程仓库

```bash
git remote rm <remote-name>
```

重命名远程仓库

```bash
git remote rename <old-name> <new-name>
```

查看远程分支

```bash
git branch -r
```

## 拉取

- `git fetch`是将远程主机的最新内容拉到本地，用户在检查了以后决定是否合并到工作本机分支中。

- `git pull` 则是将远程主机的最新内容拉下来后直接合并，即：`git pull = git fetch + git merge`，这样可能会产生冲突，需要手动解决。

从远程仓库**拉取**代码

```bash
git pull <remote-name> <branch-name>
```

`fetch` 默认远程仓库 `(origin)` 当前分支的代码，然后合并到本地分支

```bash
git pull
```

将本地改动的代码 `rebase` 到远程仓库的最新代码上（ 为了有一个干净、 线性的提交历史） 

```bash
git pull --rebase
```

获取所有远程分支

```bash
git fetch <remote-name>
```

`fetch` 某一个特定的远程分支 

```bash
git fetch <remote-name> <branch-name>
```

## 推送

**推送**代码到远程仓库 (然后再发起 `pull request` )

```bash
git push <remote-name> <branch-name>
```

# GitFlow

**GitFlow** 是一种流程模型，用于在Git上管理软件开发项目

### 主分支 (master/main)

代表了项目的稳定版本，每个提交到主分支的代码都应该是经过**测试和审核**的。

### 开发分支 (develop)

用于**日常开发**。所有的功能分支、发布分支和修补分支都应该从开发分支派生出来。

### 功能分支 (feature)

用于开发**单独的功能或者特性**。每个功能分支都应该从开发分支派生，并在开发完成后合并回开发分支。

### 发布分支 (release)

用于**准备项目发布**。发布分支应从开发分支派生，并在准备好发布版本后合并回主分支和开发分支。

### 热修复分支 (hotfix)

用于**修复**主分支上的紧急问题。热修复分支应该从主分支派生，并在修复完成后，合并回主分支和开发分支。

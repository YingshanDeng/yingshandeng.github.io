title: Git 的常用命令和技能
date: 2017-02-25 11:27:43
tags: Git
categories: Git
---


在开发过程中，常用的 git 命令和技巧整理如下。

<!-- more -->

## Git基础

![](http://7vikhl.com1.z0.glb.clouddn.com/56A5C90C-5C23-4915-96EC-4B4A4F60E5E1.png)

## stash 储藏
当你在某个分支上工作到一半时，需要切换到其他分支进行一些工作，但是你此时并不想提交当前工作了一半的内容时，你可以使用 `git stash` 这个命令解决你的问题。[参考文档](https://git-scm.com/book/zh/v1/Git-%E5%B7%A5%E5%85%B7-%E5%82%A8%E8%97%8F%EF%BC%88Stashing%EF%BC%89)

### `git stash`
`git stash` 可以获取你工作目录的中间状态——也就是你**修改过的被追踪的文件**和**暂存的变更**，并将它保存到一个未完结变更的堆栈中，随时可以重新应用。
注意：未被跟踪的文件在执行 `git stash` 时是不会被保存的。例如：

```
On branch master
Your branch is ahead of 'origin/master' by 5 commits.
  (use "git push" to publish your local commits)
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   file.js

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   1.html

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	stash.txt
```

当前状态中
	- 工作区：新增文件 stash.txt (**Untracked**)，修改文件 1.html
	- 暂存区：修改文件 file.js
此时执行 `git stash`，其中未被跟踪（Untracked）的 stash.txt 是不会被保存的 （此时你需要先 `git add stash.txt`）

### stash 的一些操作
1、查看当前 stash 栈中的情况 : `git stash list`
```
$ git stash list
stash@{0}: WIP on master: 049d078 added the index file
stash@{1}: WIP on master: c264051 Revert "added file_size"
stash@{2}: WIP on master: 21d80a5 added number to log
```

2、应用/移除 stash 栈中的储藏的内容
你可以在不同的分支应用 stash 储藏的内容；也可以在当前工作目录中包含了已修改和未提交的内容时，应用 stash 储藏的内容。

`git stash apply`: 应用最新 stash 储藏的内容
`git stash apply stash@{2}`: 应用指定的 stash 储藏内容
`git stash drop`: 移除最新 stash 储藏的内容
`git stash drop stash@{2}`: 移除指定的 stash 储藏内容
`git stash clear`: 清除 stash 栈中所有的内容

3、取消储藏(Un-applying a Stash)
在应用了 stash 储藏后，进行了一些修改，此时又要取消之前应用的储藏的修改，可以通过以下命令实现：
`git stash show -p stash@{0} | git apply -R`
同样的，如果你沒有指定具体的某个储藏，Git 会选择最近的储藏:
`git stash show -p | git apply -R`

4、从储藏中创建分支
恢复储藏的工作然后在新的分支上继续当时的工作
`git stash branch testchanges`

## tag 标签
Git 可以对某一时间点上的版本打上标签。人们在发布某个软件版本（比如 v1.0 等等）的时候，经常这么做。
> 关于如何软件开发过程中版本号如何定义，参考[论版本号的正确打开方式](http://taobaofed.org/blog/2016/08/04/instructions-of-semver/)

1、**添加标签**
```
// 添加了 v1.4 标签，同时指定了标签说明：my version 1.4
git tag -a v1.4 -m 'my version 1.4'
```
在开发过程中，可能有时候会忘记打标签了，那么如果出现这种情况如何补救呢？
可以通过 `git log --pretty=oneline` 等方式找到需要打标签的那个 commit，然后执行：
```
// 9fceb02: commit 对应的 hash 值
git tag -a v1.2 -m '补充tag' 9fceb02
```

2、**查看标签**
```
// 列出当前已有的标签
git tag
// 用特定的搜索模式列出符合条件的标签
git tag -l 'v1.4.2.*’
// 查看标签内容
git show v1.4
```

3、**推送标签**
```
// 推送指定标签到远程
git push origin v1.5
// 一次性推送所有本地标签到远程
git push origin --tags
```

4、**删除标签**
```
// 删除本地标签
git tag -d 标签名
// 删除远程标签
git push origin :refs/tags/标签名
```

## `git revert` 和 `git reset`
1、**git revert**
撤销某次操作(commit)，并把这次撤销当做一次新的提交，版本号递增，就是用一次新的提交(commit)来回滚之前的commit。
```
// 撤销前一次 commit
git revert HEAD
// 撤销某一指定commit（xxx 指某一次 commit 的 hash 值）
git revert xxx
```
当然，如果执行了这个命令后出现冲突等问题，需要解决比较麻烦，后悔了，不想继续这个撤销过程了，那么可以通过 `git revert —abort` 就可以取消。

2、**git reset**
`git reset` 是直接删除指定的 commit, 但是文件和修改会移动到 workspace 工作区，如果还想清除：
```
git clean -df
git checkout -- .
```
`git reset` 常用于代码回滚，譬如 `git reset xxx` 是将当期 HEAD 直接指向 hash 值为 xxx 的那个commit，xxx commit 之后的所有变更都移动到 workspace 工作区中，这样有点不好的是，会将 xxx commit 之后的所有 commit 都丢失掉。

执行 `git reset` 后，如果后悔了，怎么办？
通过 `git reflog` 查看操作历史，找到之前 HEAD 的 hash 值，然后 `git reset --hard xxx` 到那个 hash 即可。
那么如果在代码回滚的时候，想要保留回滚的那些变更，方便以后我们撤销这次回滚，怎么处理呢？就得用 `git revert` 了。
```
git revert —-no-commit xxx..yyy
git revert —-no-commit xxx^..yyy
```
**注意：**
* xxx 是在 yyy 之前的提交
* `xxx..yyy` 和 `xxx^..yyy` 的区别是，回滚操作中后者包含 xxx commit，而前者不包括 xxx commit.

这个命令会将 xxx 到 yyy commit 之间变更的内容的 revert 放到暂存区，然后这时，需要手动对暂存区的这些 revert 内容进行一次提交，那么有了这个提交，就方便我们以后撤销这次回滚了。

3、二者的区别：
git reset 是把HEAD向后移动了一下
git revert 是HEAD继续前进，只是新的 commit 的内容和要 revert 的内容正好相反，能够抵消要被revert的内容


## git-flow
> 关于 git 工作流可以参考:[A successful Git branching model](http://nvie.com/posts/a-successful-git-branching-model/)

![](http://7vikhl.com1.z0.glb.clouddn.com/git-model@2x.png)

* 项目仓库中一般包含四大类分支：master, hotfix, dev, feature。
	* master 主分支，用于项目发布，一般不直接在此分支上进行开发，仅在发布时，将开发分支 dev 代码合并进行发布
	* dev 开发分支，每一个版本开发代码都在要合并到此分支
	* feature 特性分支，不同的开发人员负责不同的特性开发，每一个特性可以单独一个分支，在各自分支上进行开发，减少干扰；特性分支开发完成后，需要合并到开发分支 dev 中
	* hotfix 热更新分支，用于线上 bug 的紧急修复或者某一个紧急功能的上线，从 master 上创建的分支
* 各个分支之间的 merge 和 rebase
    * 从 dev 开发分支更新 feature 特性分支: rebase
    * feature 特性分支合并到 dev 开发分支: merge
    * 从 dev / hotfix 合并到 master: merge
    * 远程分支到本地分支(例如: remote dev --> local dev): git pull -r
* merge 和 rebase 的区别是会产生一个 merge commit，而这也是区分何时使用 merge 和 rebase 的关键：
    * 从特性分支到开发分支，需要用 merge，产生一个额外的 merge commit 方便回滚
    * 远程分支到本地分支，就必须要用 rebase，否则会产生多余的 merge commit，显然是不需要的，想想同一个分支怎么需要合并操作呢？
    * 我们常用的 git pull 其实是包含着两个操作 git fetch + git merge

## 其他
1、**丢弃工作区 workspace 的修改**
`git clean` removes all untracked files and `git checkout` clears all unstaged changes.
```
git clean -df
git checkout -- .
```
2、**撤销最后一个提交(last commit)**
```
// 直接干掉最后一个commit不留痕迹
git reset --hard HEAD~1
// 撤销最后一个commit，并将该commit中的变化保留在 workspace 工作区
git reset HEAD~1
// 撤销最后一个commit，并将该commit中的变化保留在 index 暂存区
git reset --soft HEAD~1
```
[参考链接](http://stackoverflow.com/questions/927358/how-to-undo-last-commits-in-git)
3、**取消暂存区 index 中的变更**
```
// 取消暂存区的某个文件（xxx：文件名）
git reset HEAD xxx
// 取消暂存区中的所有变更
git reset HEAD .
```
4、**取消工作区 workspace 中的变更**
```
// 取消工作区中的某个文件（xxx：文件名）
git checkout -- xxx
// 取消工作区中的所有变更
git checkout -- .
```
5、**删除远程分支**
```
git push origin :branch-name
```
冒号前面的空格不能少，原理是把一个空分支push到server上，相当于删除该分支。
6、**git cherry-pick**
使用 `git cherry-pick` 命令可以把某个分支上的一个或者几个 commit 合并到另一个分支上
```
// 提取一个 commit
git cherry-pick [commitID]

// 提取一个 commit 到另一个 commit 之间的所有的 commit，不包括start-commitID，包括end-commitID
git cherry-pick [start-commitID]..[end-commitID]

// 提取一个 commit 到另一个 commit 之间的所有的 commit，既包括start-commitID，也包括end-commitID
git cherry-pick start-commitID]^..[end-commitID]
```
7、**学习链接**
[Git的奇技淫巧](https://github.com/521xueweihan/git-tips#%E8%81%94%E7%B3%BB%E6%88%91)



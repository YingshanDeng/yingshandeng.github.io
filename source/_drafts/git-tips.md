title: Git 的奇技淫巧
tags:
---

参考链接： https://github.com/521xueweihan/git-tips#%E8%81%94%E7%B3%BB%E6%88%91


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


## 丢弃工作区 workspace 的修改
`git clean` removes all untracked files and `git checkout` clears all unstaged changes.

```
git clean -df
git checkout -- .
```

## tag 标签


## 撤销: git revert / git reset

## 代码回滚


## git-flow




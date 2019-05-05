---
title: Git常用命令
date: 2019/04/27 09:58
categories:
- Git
tags:
- Git
---

## 远程仓库

### 解除与远程仓库的关联

首先获取远程的仓库名, 然后删除

```bash
# 获取仓库名
git remote
# 解除关联
git remote remove <name>
```

### 关联远程仓库

```bash
git remote add <name> <url>
```

## 撤销修改

### 撤销修改

#### 工作区

使用 HEAD 中的最新内容替换工作区中的文件

```bash
# 单个文件/文件夹
git checkout --<filename>
# 所有文件/文件夹
git checkout .
```

#### 暂存区

使用 `<commit-id>` 中的内容更新暂存区中的文件

```bash
# 单个文件/文件夹
git reset <commit-id> <filename>
# 所有文件/文件夹
git reset <commit-id> .
# git reset --mixed HEAD <filename> 的简写
git reset filename
git reset .
```

### 回到某个 commit

会抹去某个 commit-id 后面的所有 commit.

#### 工作区

```bash
# 回到某个 commit-id 版本, 不保留工作区修改.
git reset --hard <commit-id>
```

#### 暂存区

```bash
# 更新 HEAD 和 暂存区, 会保留工作区.
git reset <commit-id>
```

## 知识点

#### ^ VS ~

`^`: Git 将 `^` 解析为该引用的父提交. 

但加上数字时, `HEAD^n` 代表 `HEAD` 的第 n 个父提交. 这个语法只适用于合并(merge)的提交, 因为提交合并会有多个父提交, 所以第一个父提交是你合并时所在的分支, 而第二个父提交是你说合并的分支.

多个 `^` 连用, 如 `HEAD^^^` 表示 `HEAD` 的父提交的父提交的父提交.

`~`: Git 将 `~` 解析为该引用的第一个父提交. 第一个父提交 = 合并时所在分支的提交.

但加上数字时, 意义与 `^` 不同. `HEAD~2` 代表 `HEAD` 的第一个父提交的第一个父提交, 即"祖父提交".

多个 `~` 连, 如 `HEAD~~~` 表示 `HEAD` 的第一个父提交的第一个父提交的第一个父提交, 与加数字的作用相同.

##### 总结

`^` 与 `~` 在不加数字时, 含义相同, 均指当前引用的 `^`或 `~`个数 的第一个父提交.

`^n` 表示当前引用的第 n 个父提交, 即 parent.

`~n` 表示当前引用的向前第 n 辈父提交, 即 ancestor.

#### reset 流程

初始状态:

![初始状态](https://git-scm.com/book/en/v2/images/reset-start.png)

1. 移动 HEAD

   ![移动HEAd](https://git-scm.com/book/en/v2/images/reset-soft.png)

   `reset` 做的第一件事是移动 HEAD 的指向.  这与改变 HEAD 自身不同(`checkout` 所做的); `reset` 移动 HEAD 指向的分支. 这意味着如果 HEAD 设置为 `master` 分支(例如, 你正在 `master` 分支上), 运行 `git reset 9e5e64a` 将会使 `master` 指向 `9e5e64a`.

   它本质上是撤销了上一次 `git commit` 命令. 当你在运行 `git commit` 时，Git 会创建一个新的提交, 并移动 HEAD 所指向的分支来使其指向该提交. 当你将它 `reset`回 `HEAD~`(HEAD 的父结点)时, 其实就是把该分支移动回原来的位置, 而不会改变索引和工作目录. 现在你可以更新索引并再次运行 `git commit` 来完成 `git commit --amend` 所要做的事情了(见 [修改最后一次提交](https://git-scm.com/book/zh/v2/ch00/r_git_amend)).

2. 更新索引(--mixed)

   ![更新索引](https://git-scm.com/book/en/v2/images/reset-mixed.png)

   `reset` 这时会用 HEAD 指向的当前快照的内容来更新索引(暂存区).

   如果指定 `--mixed` 选项, `reset` 将会在这时停止. 这也是默认行为, 所以如果没有指定任何选项(在本例中只是 `git reset HEAD~`), 这就是命令将会停止的地方.

   它依然会撤销一上次 `commit`，但还会 *取消暂存* 的所有东西. 于是, 我们回滚到了所有 `git add` 和 `git commit` 的命令执行之前.

3. 更新工作目录(--hard)

   ![更新工作区](https://git-scm.com/book/en/v2/images/reset-hard.png)

   `reset` 这时会让工作目录看起来像索引. 如果使用 `--hard` 选项, 它将会执行这一步.

   须注意, `--hard` 标记是 `reset` 命令唯一的危险用法, 它也是 Git 会真正地销毁数据的仅有的几个操作之一. 其他任何形式的 `reset` 调用都可以轻松撤消, 但是 `--hard` 选项不能，因为它强制覆盖了工作目录中的文件. 在这种特殊情况下，我们的 Git 数据库中的一个提交内还留有该文件的 **v3** 版本，我们可以通过 `reflog` 来找回它. 但是若该文件还未提交, Git 仍会覆盖它从而导致无法恢复.

##### 总结

- `reset --soft`: 执行步骤 1. 只回退 `commit` 的信息, 暂存区和工作区没有发生变化, 与回退之前保持一致. 如果要继续提交, 直接 `git commit` 即可.

- `reset [--mixed]`: 执行步骤 1,2, 默认. 将 HEAD 重置到另外一个 commit, 且更新索引(暂存区)和 HEAD 相同. 工作区不会被更改.

- `reset --hard`: 执行步骤 1,2,3. 彻底回到指定的 commit-id, 暂存区和工作区都会变为指定 commit-id 版本的内容.
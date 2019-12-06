---
title: Git常用命令
date: 2019/04/27 09:58
categories:
- Git
tags:
- Git
---

## 环境相关

### 设置用户

```bash
git config --global user.name "Your name here"
git config --global user.email "email@example.com"
```

### 连接 Github

1. 生成 SSH 公钥。

   ```bash
   ssh-keygen -t rsa -C "email@example.com"
   ```

2. 获取刚才生成的公钥。

   ```bash
   cat ~/.ssh/id_rsa.pub
   ```

3. 在 Github 上 `Settings > SSH and GPG keys` 中添加生成的公钥。

4. 验证。

   ```bash
   ssh -T git@github.com
   ```

### 设置代理

```bash
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5//127.0.0.1:1080'
```



## 初始化

### 初始化一个已经有内容了的目录

如果本地已经有了内容, 但是还没有使用 Git 管理, 现在先在 Github 上创建了 Repository. 把项目同步到仓库的连招如下:

```bash
# 1 初始化
git init
# 2 暂存并提交
git add .
git commit -m "init"
# 3 设置远程仓库
git remote add <name> <url>
# 4 推送到远端
git push
# 此时发现 ! [rejected]        master -> master (fetch first) 
# 下面的黄色提示我们远程仓库有我们本地没有的工作(默认创建了README.txt), 推荐我们使用 git pull
# 5 听话用 git pull
git pull
# 此时没有高亮的提示了, 但还是有白字提醒我们当前分支没有跟任何远程分支建立关联
# 6 听话建立关联 
git branch --set-upstream-to=origin/<branch> master
# 7 总该 push 了吧
git push
# 此时提示  ! [rejected]        master -> master (non-fast-forward)
# 下面黄色提示我们: 本地分支落后于远程分支, 尝试先 git pull
# 由于远程分支创建了 README.txt, 所以如果直接 git pull 会被拒绝, 提示 refusing to merge unrelated histories
# 8 上干货
git pull --allow-unrelated-histories
# 9 接下来就是正常操作了
git add .
git commit -m "init remote"
git push
```

## 仓库

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

### 移除文件以及取消对文件的跟踪

```bash
# 首先列出操作会取消跟踪的文件列表
git rm -r -n --cached <filename>
# 真正地取消跟踪
git rm -r --cached <filename>
# 提交
git commit -m <msg>
```

## 分支管理

### 合并分支并保留目标分支的提交记录

如果合并没有冲突的分支, 想要保存下目标分支的提交记录, 可以禁止使用 *Fast-Forward* 模式.

```bash
git merge --no-ff <commit-id>
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

## 查看

### 查看某次 commit 的信息

```bash
git show <commmit-id>
```

## 修改

### 修改某次 commit 的注释

执行该命令后, 会出现交互界面让你修改最近一次尚未 push 的 commit 的注释.

```bash
git commit --amend
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

### Fast-Forward 模式

简单的说就是, 当你视图合并两个分支时, 如果顺着一个分支走下去能够到达另一个分支, 那么 Git 在合并两者的时候只会简单地将指针向前推进, 因为这种情况下的合并操作没有需要解决的冲突 -- 这就叫做*快进(Fast-Forward)*.

#### 示例

提前做的工作为: 

```
master > git commit -m "init"
master > git commit -m "a"
master > git checkout -b second
second > git commit -m "b1"
second > git commit -m "b2"
```

![初始化现场](https://i.loli.net/2019/05/08/5cd25ab10fe58.png)

现在处于 master 分支的 a commit, 想要合并 second 分支上的 b2 commit.

直接使用 `merge second` 后, 结果如下:

![1557290130149.png](https://i.loli.net/2019/05/08/5cd26337006e3.png)

因为 b2 之于 a 只是领先了 2 个 commit, 即: master 是 second 的直接上游, 所以这种情况下 merge 会默认使用 *Fast-Forward* 模式, 只是将指针向前移动. 

这种情况下, 想要看到 second 在被合并到 master 之前所做的提交就比较困难. 如果想要保留 second 的提交记录, 可以使用 `merge --no-ff` 参数来禁止 *Fast-Forward* 模式.

初始状态:

![初始化现场](https://i.loli.net/2019/05/08/5cd25ab10fe58.png)

使用 `merge --no-ff second` 结果如下:

![1557291421924.png](https://i.loli.net/2019/05/08/5cd2631c0292a.png)

master 分支和 second 分支成功地合并了, 并且 second 分支的提交记录也被保留了下来.


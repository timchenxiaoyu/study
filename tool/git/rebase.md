## rebase
我们执行以下命令获得四个 Commit：

```go

mkdir test
cd test

git init

echo "0" >> a
git add a
git commit -m "Commit-0"

echo "1" >> a
git add a
git commit -m "Commit-1"

echo "2" >> a
git add a
git commit -m "Commit-2"

echo "3" >> a
git add a
git commit -m "Commit-3"

```
我们可以看到 Git 的历史长成这样：

* b1b8189 - (HEAD -> master) Commit-3
* 5756e15 - Commit-2
* e7ba81d - Commit-1
* 5d39ff2 - Commit-0
那么问题来了，如何把 e7ba81d(Commit-1)、5756e15(Commit-2)、b1b8189(Commit-3) 合并到一起，并且只保留 e7ba81d(Commit-1) 的 Git message Commit-1 呢？

这个时候我们就要祭出我们这篇文章的主角—— git rebase -i 了！
这里我不想直接搬出写文档的那套，把所有的选项都介绍完，我们就把这次要用到的讲一下。

-i 实际上就是 --interactive 的简写，在使用 git rebase -i 时，我们要在后面再添加一个参数，这个参数应该是 最新的一个想保留的 Commit。这句话读起来有点坳口，所以这个情况下通常需要举个例子。就我们前面提到的那个例子中，这个「最新的一个想保留的 Commit」就是 5d39ff2(Commit-0)，于是我们的命令看起来就长这样：

git rebase -i 5d39ff2
当然，我们也可以通过 HEAD~3 来指定该 Commit：

git rebase -i HEAD~3
按下回车后，我们会进入到这么一个界面：
```go

pick e7ba81d Commit-1
pick 5756e15 Commit-2
pick b1b8189 Commit-3

# Rebase 5d39ff2..b1b8189 onto 5d39ff2 (3 command(s))
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out

```
前面三行是我们需要操作的三个 Commit，每行最前面的是对该 Commit 操作的 Command。关于每个 Command 具体做什么，下面的注释写得非常清楚。为了完成我们的需求，我们可以关注到这两个命令：

s, squash = use commit, but meld into previous commit
f, fixup = like "squash", but discard this commit's log message
为了让大家看得更明白，我不厌其烦地翻译一下：

squash：使用该 Commit，但会被合并到前一个 Commit 当中
fixup：就像 squash 那样，但会抛弃这个 Commit 的 Commit message
看样子两个命令都可以完成我们的需求，那么让我们先试一下 squash！由于我们是想把三个 Commit 都合并在一起，并且使 Commit Message 写成 Commit-1，所以我们需要把 5756e15(Commit-2) 和 b1b8189(Commit-3) 前面的 pick 都改为squash，于是它看起来像这样：

pick e7ba81d Commit-1
squash 5756e15 Commit-2
squash b1b8189 Commit-3
当然，因为我很懒，所以通常我会使用它的缩写：

pick e7ba81d Commit-1
s 5756e15 Commit-2
s b1b8189 Commit-3
完成后，使用 :wq 保存并退出。这个时候，我们进入到了下一个界面：
```go

# This is a combination of 3 commits.
# The first commit's message is:
Commit-1

# This is the 2nd commit message:

Commit-2

# This is the 3rd commit message:

Commit-3

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Tue Jan 5 23:27:22 2016 +0800
#
# rebase in progress; onto 5d39ff2
# You are currently editing a commit while rebasing branch 'master' on '5d39ff2'.
#
# Changes to be committed:
#   modified:   a

```
通过下面的注释，我们可以知道，这里其实就是一个编写 Commit Message 的界面，带 # 的行会被忽略掉，其余的行就会作为我们的新 Commit Message。为了完成我们的需求，我们修改成这样：
```go


Commit-1

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Tue Jan 5 23:27:22 2016 +0800
#
# rebase in progress; onto 5d39ff2
# You are currently editing a commit while rebasing branch 'master' on '5d39ff2'.
#
# Changes to be committed:
#   modified:   a
```
使用 :wq 后，再看一下我们的 log：

* 2d7b687 - (HEAD -> master) Commit-1
* 5d39ff2 - Commit-0
任务完成！
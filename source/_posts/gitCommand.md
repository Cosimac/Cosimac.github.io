---
title: Git 常见用法(包含🌰)
date: 2022-05-19 23:40:55
tags:
---
## Git 概念
Workspace：工作区
Index / Stage：暂存区
Repository：仓库区（或本地仓库）
Remote：远程仓库
![Git](https://www.ruanyifeng.com/blogimg/asset/2015/bg2015120901.png)
### git HEAD
HEAD 是一个对当前检出记录的符号引用, 也就是指向你正在其基础上进行工作的提交记录。
HEAD 总是指向当前分支上最近一次提交记录。大多数修改提交树的 Git 命令都是从改变 HEAD 的指向开始的。
HEAD 通常情况下是指向分支名的（如 bugFix）。在你提交时，改变了 bugFix 的状态，这一变化通过 HEAD 变得可见。

```
# 分离的 HEAD 就是让其指向了某个具体的提交记录而不是分支名。在命令执行之前的状态如下所示：
# HEAD -> main -> C1
# HEAD 指向 main， main 指向 C1
# C1 表示具体的某条提交记录 
$ git checkout C1
```

```
# 如果 HEAD 指向的是一个引用，还可以用 git symbolic-ref HEAD 查看它的指向
$ git symbolic-ref HEAD
```
### git 相对引用
背景：通过指定提交记录哈希值的方式在 Git 中移动不太方便。在实际应用时，并没有像本程序中这么漂亮的可视化提交树供你参考，所以你就不得不用 git log 来查查看提交记录的哈希值，通过哈希值指定提交记录很不方便，所以 Git 引入了相对引用。这个就很厉害了!
使用相对引用的话，你就可以从一个易于记忆的地方（比如 bugFix 分支或 HEAD）开始计算。
相对引用非常给力，这里我介绍两个简单的用法：
```
# 使用 ^ 向上移动 1 个提交记录
# 所以 main^ 相当于“main 的父节点”。
# main^^ 是 main 的第二个父节点
$ git checkout main^

# 使用 ~<num> 向上移动多个提交记录，如 ~3
$ git checkout main~3

# 你也可以将 HEAD 作为相对引用的参照。下面咱们就用 HEAD 在提交树中向上移动几次。
$ git checkout HEAD^

# 操作符 ^ 与 ~ 符一样，后面也可以跟一个数字, 但是该操作符后面的数字与 ~ 后面的不同，并不是用来指定向上返回几代，而是指定合并提交记录的某个父提交。还记得前面提到过的一个合并提交有两个父提交吧，所以遇到这样的节点时该选择哪条路径就不是很清晰了。
# Git 默认选择合并提交的“第一个”父提交，在操作符 ^ 后跟一个数字可以改变这一默认行为。
# C0 -> C1 -> C3(main) ｜ C0 -> C2 -> C3(main)
$ git checkout main^ (默认指向的是C1而不是C2)
$ git checkout main^2 (默认指向的是C2而不是C1)

# 这些操作符还支持链式操作！
$ git checkout HEAD~^2~2
```
## Git 常用语法
### git config
配置 Git 的相关参数。
Git 一共有3个配置文件：
1. 仓库级的配置文件：在仓库的 .git/.gitconfig，该配置文件只对所在的仓库有效。
2. 全局配置文件：Mac 系统在 ~/.gitconfig，Windows 系统在 C:\Users\<用户名>\.gitconfig。
3. 系统级的配置文件：在 Git 的安装目录下（Mac 系统下安装目录在 /usr/local/git）的 etc 文件夹中的 gitconfig。

```
# 查看配置信息
# --local：仓库级，--global：全局级，--system：系统级
$ git config <--local | --global | --system> -l

# 查看当前生效的配置信息
$ git config -l

# 编辑配置文件
# --local：仓库级，--global：全局级，--system：系统级
$ git config <--local | --global | --system> -e

# 添加配置项
# --local：仓库级，--global：全局级，--system：系统级
$ git config <--local | --global | --system> --add <name> <value>

# 获取配置项
$ git config <--local | --global | --system> --get <name>

# 删除配置项
$ git config <--local | --global | --system> --unset <name>

# 配置提交记录中的用户信息
$ git config --global user.name <用户名>
$ git config --global user.email <邮箱地址>

# 更改Git缓存区的大小
# 如果提交的内容较大，默认缓存较小，提交会失败
# 缓存大小单位：B，例如：524288000（500MB）
$ git config --global http.postBuffer <缓存大小>

# 调用 git status/git diff 命令时以高亮或彩色方式显示改动状态
$ git config --global color.ui true

# 配置可以缓存密码，默认缓存时间15分钟
$ git config --global credential.helper cache

# 配置密码的缓存时间
# 缓存时间单位：秒
$ git config --global credential.helper 'cache --timeout=<缓存时间>'

# 配置长期存储密码
$ git config --global credential.helper store
```

### git clone
从远程仓库克隆一个版本库到本地。

```
# 默认在当前目录下创建和版本库名相同的文件夹并下载版本到该文件夹下
$ git clone <远程仓库的网址>

# 指定本地仓库的目录
$ git clone <远程仓库的网址> <本地目录>

# -b 指定要克隆的分支，默认是master分支
$ git clone <远程仓库的网址> -b <分支名称> <本地目录>
```

### git init
初始化项目所在目录，初始化后会在当前目录下出现一个名为 .git 的目录。

```
# 初始化本地仓库，在当前目录下生成 .git 文件夹
$ git init
```

### git status
查看本地仓库的状态。

```
# 查看本地仓库的状态
$ git status

# 以简短模式查看本地仓库的状态
# 会显示两列，第一列是文件的状态，第二列是对应的文件
# 文件状态：A 新增，M 修改，D 删除，?? 未添加到Git中
$ git status -s
```

### git remote
操作远程库。

```
# 列出已经存在的远程仓库
$ git remote

# 列出远程仓库的详细信息，在别名后面列出URL地址
$ git remote -v
$ git remote --verbose

# 添加远程仓库
$ git remote add <远程仓库的别名> <远程仓库的URL地址>

# 修改远程仓库的别名
$ git remote rename <原远程仓库的别名> <新的别名>

# 删除指定名称的远程仓库
$ git remote remove <远程仓库的别名>

# 修改远程仓库的 URL 地址
$ git remote set-url <远程仓库的别名> <新的远程仓库URL地址>

```

### git branch
操作 Git 的分支命令。

```
# 列出本地的所有分支，当前所在分支以 "*" 标出
$ git branch

# 列出本地的所有分支并显示最后一次提交，当前所在分支以 "*" 标出
$ git branch -v

# 创建新分支，新的分支基于上一次提交建立
$ git branch <分支名>

# 修改分支名称
# 如果不指定原分支名称则为当前所在分支
$ git branch -m [<原分支名称>] <新的分支名称>
# 强制修改分支名称
$ git branch -M [<原分支名称>] <新的分支名称>

# 删除指定的本地分支
$ git branch -d <分支名称>

# 强制删除指定的本地分支
$ git branch -D <分支名称>

# 强制修改分支位置
# 我使用相对引用最多的就是移动分支。可以直接使用 -f 选项让分支指向另一个提交。将 main 分支强制指向 HEAD 的第 3 级父提交。
$ git branch -f main HEAD~3

```

### git checkout
检出命令，用于创建、切换分支等。

```
# 切换到已存在的指定分支
$ git checkout <分支名称>

# 创建并切换到指定的分支，保留所有的提交记录
# 等同于 "git branch" 和 "git checkout" 两个命令合并
$ git checkout -b <分支名称>

# 创建并切换到指定的分支，删除所有的提交记录
$ git checkout --orphan <分支名称>

# 替换掉本地的改动，新增的文件和已经添加到暂存区的内容不受影响
$ git checkout <文件路径>
```

### git cherry-pick
把已经提交（多个/单个）的记录有序的合并到当前分支。关键词：多个/有序

```
# 把已经提交的记录合并到当前分支
$ git cherry-pick <commit ID>

# 可进行多个提交的抓取 C1 C2 或者 C2 C1 产生的效果完全不一样是有孙旭的提交记录
$ git cherry-pick C1 C2
$ git cherry-pick C2 C1
```

### git add
把要提交的文件的信息添加到暂存区中。当使用 git commit 时，将依据暂存区中的内容来进行文件的提交

```
# 把指定的文件添加到暂存区中
$ git add <文件路径>

# 添加所有修改、已删除的文件到暂存区中
$ git add -u [<文件路径>]
$ git add --update [<文件路径>]

# 添加所有修改、已删除、新增的文件到暂存区中，省略 <文件路径> 即为当前目录
$ git add -A [<文件路径>]
$ git add --all [<文件路径>]

# 查看所有修改、已删除但没有提交的文件，进入一个子命令系统
$ git add -i [<文件路径>]
$ git add --interactive [<文件路径>]
```

### git commit
将暂存区中的文件提交到本地仓库中。

```
# 把暂存区中的文件提交到本地仓库，调用文本编辑器输入该次提交的描述信息
$ git commit

# 把暂存区中的文件提交到本地仓库中并添加描述信息
$ git commit -m "<提交的描述信息>"

# 把所有修改、已删除的文件提交到本地仓库中
# 不包括未被版本库跟踪的文件，等同于先调用了 "git add -u"
$ git commit -a -m "<提交的描述信息>"

# 修改上次提交的描述信息
$ git commit --amend
```
### git fetch
从远程仓库获取最新的版本到本地的 tmp 分支上。

```
# 将远程仓库所有分支的最新版本全部取回到本地
$ git fetch <远程仓库的别名>

# 将远程仓库指定分支的最新版本取回到本地
$ git fetch <远程主机名> <分支名>
```
### git merge
合并分支

```
# 把指定的分支合并到当前所在的分支下
$ git merge <分支名称>
```
### git diff
比较版本之间的差异。

```
# 比较当前文件和暂存区中文件的差异，显示没有暂存起来的更改
$ git diff

# 比较暂存区中的文件和上次提交时的差异
$ git diff --cached
$ git diff --staged

# 比较当前文件和上次提交时的差异
$ git diff HEAD

# 查看从指定的版本之后改动的内容
$ git diff <commit ID>

# 比较两个分支之间的差异
$ git diff <分支名称> <分支名称>

# 查看两个分支分开后各自的改动内容
$ git diff <分支名称>...<分支名称>
```
### git pull
从远程仓库获取最新版本并合并到本地。
首先会执行 git fetch，然后执行 git merge，把获取的分支的 HEAD 合并到当前分支。

```
# 从远程仓库获取最新版本。
$ git pull
```
### git push
把本地仓库的提交推送到远程仓库。

```
# 把本地仓库的分支推送到远程仓库的指定分支
$ git push <远程仓库的别名> <本地分支名>:<远程分支名>

# 删除指定的远程仓库的分支
$ git push <远程仓库的别名> :<远程分支名>
$ git push <远程仓库的别名> --delete <远程分支名>
# 为推送当前分支并建立与远程上游的跟踪，使用
$ git push --set-upstream <远程仓库别名> <远程分支名>
$ git push --set-upstream origin master
# 强行推送本地记录
$ git push -u origin +master
```
### git log
显示提交的记录。

```
# 打印所有的提交记录
$ git log

# 打印从第一次提交到指定的提交的记录
$ git log <commit ID>

# 打印指定数量的最新提交的记录
$ git log -<指定的数量>
```
### git reset
还原提交记录。

```
# 重置暂存区，但文件不受影响
# 相当于将用 "git add" 命令更新到暂存区的内容撤出暂存区，可以指定文件
# 没有指定 commit ID 则默认为当前 HEAD
$ git reset [<文件路径>]
$ git reset --mixed [<文件路径>]

# 将 HEAD 的指向改变，撤销到指定的提交记录，文件未修改
$ git reset <commit ID>
$ git reset --mixed <commit ID>

# 将 HEAD 的指向改变，撤销到指定的提交记录，文件未修改
# 相当于调用 "git reset --mixed" 命令后又做了一次 "git add"
$ git reset --soft <commit ID>

# 将 HEAD 的指向改变，撤销到指定的提交记录，文件也修改了
$ git reset --hard <commit ID>
```
### git revert
生成一个新的提交来撤销某次提交，此次提交之前的所有提交都会被保留。

```
# 生成一个新的提交来撤销某次提交
$ git revert <commit ID>
```
### git tag
分支很容易被人为移动，并且当有新的提交时，它也会移动。分支很容易被改变，大部分分支还只是临时的，并且还一直在变。
你可能会问了：有没有什么可以永远指向某个提交记录的标识呢，比如软件发布新的大版本，或者是修正一些重要的 Bug 或是增加了某些新特性，有没有比分支更好的可以永远指向这些提交的方法呢？
当然有了！Git 的 tag 就是干这个用的啊，它们可以（在某种程度上 —— 因为标签可以被删除后重新在另外一个位置创建同名的标签）永久地将某个特定的提交命名为里程碑，然后就可以像分支一样引用了。
更难得的是，它们并不会随着新的提交而移动。你也不能切换到某个标签上面进行修改提交，它就像是提交树上的一个锚点，标识了某个特定的位置。

```
# 我们将这个标签命名为 v1，并且明确地让它指向提交记录 C1，如果你不指定提交记录，Git 会用 HEAD 所指向的位置。
$ git tag v1 C1

# 打印所有的标签
$ git tag

# 添加轻量标签，指向提交对象的引用，可以指定之前的提交记录
$ git tag <标签名称> [<commit ID>]

# 添加带有描述信息的附注标签，可以指定之前的提交记录
$ git tag -a <标签名称> -m <标签描述信息> [<commit ID>]

# 切换到指定的标签
$ git checkout <标签名称>

# 查看标签的信息
$ git show <标签名称>

# 删除指定的标签
$ git tag -d <标签名称>

# 将指定的标签提交到远程仓库
$ git push <远程仓库的别名> <标签名称>

# 将本地所有的标签全部提交到远程仓库
$ git push <远程仓库的别名> –tags
# 强push
$ git push origin <远程分支名> --force
```
### git mv
重命名文件或者文件夹。

```
# 重命名指定的文件或者文件夹
$ git mv <源文件/文件夹> <目标文件/文件夹>
```
### git rm
删除文件或者文件夹。

```
# 移除跟踪指定的文件，并从本地仓库的文件夹中删除
$ git rm <文件路径>

# 移除跟踪指定的文件夹，并从本地仓库的文件夹中删除
$ git rm -r <文件夹路径>

# 移除跟踪指定的文件，在本地仓库的文件夹中保留该文件
$ git rm --cached
```
### git rebase
Rebase 实际上就是取出一系列的提交记录，“复制”它们，然后在另外一个地方逐个的放下去。
Rebase 的优势就是可以创造更线性的提交历史，这听上去有些难以理解。如果只允许使用 Rebase 的话，代码库的提交历史将会变得异常清晰。

```
# 将当前分支变基到目标分支
$ git rebase <远程分支名>
# 表示继续下一个冲突(git rebase --continue 就相当于 git commit)
$ git rebase --continue
# 表示跳过当前冲突
$ git rebase --skip
# 表示退出rebase模式, 回到运行git rebase master命令之前的状态
$ git rebase --abort
```

### git describe
由于标签在代码库中起着“锚点”的作用，Git 还为此专门设计了一个命令用来描述离你最近的锚点（也就是标签）

```
# <ref> 可以是任何能被 Git 识别成提交记录的引用，如果你没有指定的话，Git 会以你目前所检出的位置（HEAD）。
git describe <ref>

# 它输出的结果是这样的：<tag>_<numCommits>_g<hash>
# tag 表示的是离 ref 最近的标签， numCommits 是表示这个 ref 与 tag 相差有多少个提交记录， hash 表示的是你所给定的 ref 所表示的提交记录哈希值的前几位。
# C0(v1) -> C1 -> C2(main)
git describe main 会输出：v1_2_gC2 
```
## Git操作场景(🌰)
### 删除掉本地不存在的远程分支
多人合作开发时，如果远程的分支被其他开发删除掉，在本地执行 git branch --all 依然会显示该远程分支，可使用下列的命令进行删除

```
# 使用 pull 命令，添加 -p 参数
$ git pull -p

# 等同于下面的命令
$ git fetch -p
$ git fetch --prune origin
```
### 只提交某次记录
发中经常会遇到的情况：我正在解决某个特别棘手的 Bug，为了便于调试而在代码中添加了一些调试命令并向控制台打印了一些信息。
这些调试和打印语句都在它们各自的提交记录里。最后我终于找到了造成这个 Bug 的根本原因，解决掉以后觉得沾沾自喜！
最后就差把 bugFix 分支里的工作合并回 main 分支了。你可以选择通过 fast-forward 快速合并到 main 分支上，但这样的话 main 分支就会包含我这些调试语句了。你肯定不想这样，应该还有更好的方式

```
# 实际我们只要让 Git 复制解决问题的那一个提交记录就可以了。跟之前我们在“整理提交记录”中学到的一样，我们可以使用
$ git rebase -i
$ git cherry-pick 
```


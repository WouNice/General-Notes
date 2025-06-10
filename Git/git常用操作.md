# git常用操作

## 分支管理

### 查看分支信息

```
git branch：列出本地已经存在的分支，当前分支会用*标记
git branch -r：查看远程版本库的分支列表
git branch -a：查看所有分支列表（包括本地和远程，remotes/开头的表示远程分支）
git branch -v：查看一个分支的最后一次提交
git branch --merged：查看哪些分支已经合并到当前分支
git branch --no-merged：查看所有未合并工作的分支
```

### 创建和切换分支

1、创建新分支

```bash
git branch 新分支名称
```

2、切换分支

```bash
git checkout 分支名称
```

 3、创建分支的同时，切换到该分支上

```bash
git checkout -b 新分支名称
```

### 从远程仓库pull（**拉取**）代码到本地分支

1、指定远程分支到本地分支

```
git pull origin 远程分支名称:本地分支名称
```

特别注意的一点：origin是远程仓库默认的连接对象名称，有时我们可能自定义了过远程仓库连接的名称，可以在Gui软件上查看远程仓库连接的名称，也可以通过如下命令进行查询：

```bash
$ git remote -v
github  git@github.com:xxx/xxx.git (fetch)
github  git@github.com:xxx/xxx.git (push)
```

2、如果不写本地分支名称，则默认和远程分支同名，命令如下：

```
$ git pull origin 远程分支名称
```

### 将新分支推送到远程仓库

方法1：使用git命令

```
git push origin 分支名称
```

假设我本地创建了一个名为dev的分支，远程仓库还没有这个分支，推送的命令是：

```
git push --set-upstream origin dev
```

分析：

> git分支与远程主机存在对应分支，可能是单个可能是多个。
>
> -   simple方式：如果当前分支只有一个追踪分支，那么git push origin到主机时，可以省略主机名。
>
> -   matching方式：如果当前分支与多个主机存在追踪关系，那么git push --set-upstream origin master（省略形式为：git push -u origin master）将本地的master分支推送到origin主机（--set-upstream选项会指定一个默认主机），同时指定该主机为默认主机，后面使用可以不加任何参数使用git push。
>
>
> 注意：
>
> Git 2.0版本之前，默认采用matching方法，现在改为默认采用simple方式。

方法2：直接在GUI面板上使用Push功能

### 删除分支

1、删除本地分支（不能删除当前所在的分支，如果要删除，必须先切换到其他分支上）

```
git branch -d 分支名称
```

如果删除时报错：error: The branch '分支名称' is not fully merged. （意思是：分支未完全合并）。解决方法是使用 -D 强制删除，代码如下：

```
git branch -D 分支名称
```

2、删除远程分支

```
git push origin :分支名称
```

> 注意：分支名称前有个冒号，分支名前的冒号代表删除

### 合并分支

假如我们现在位于分支dev上，需要将dev合并到master中：

1、首先切换到master分支上

```
git checkout master
```

如果是多人开发的话，需要把远程master分支上的代码pull下来

```
git pull origin master
```

2、然后把dev分支的代码合并到master上

```
git merge 分支名称
```

3、如果git merge的时候出现冲突，可以执行下面的命令取消merge：

```
git merge --abort
```

4、然后查看状态

```
git status
```

5、最后一步，Push推送到远程仓库

```
git push origin master
```

## 文件清理

1、完全清除本地改动：

```
git reset --hard origin/master
```

这条命令会撤销本地、暂存区、版本库(用远程服务器的origin/master替换本地、暂存区、版本库)。

2、git clean删除所有未跟踪的文件/目录，不能撤消

有的时候只有`clean -f`无效。如果你没有跟踪`DIRECTORIES`，`-d`选项也需要：

```
git reset --hard HEAD
git clean -f -d
```

快捷命令：

```
git clean -xdf && git checkout .
```

## 冲突解决

### 解决Jetbrains显示git正在rebase的问题

在Jetbrains软件中进行rebase操作时，如果是变基失败或取消操作后，Jetbrains会一直显示git正在rebase操作，此时对分支进行一些操作可能会受到限制，此时可以通过如下方式解决：

- 可以运行 `git rebase --abort` 以完全撤消变基。Git 将返回到分支的状态，即调用 `git rebase` 之前的状态。
- 可以运行 `git rebase --skip` 以完全跳过提交。这意味着将不包括由有问题的提交引入的任何更改。

### git如何强制拉取远程项目覆盖本地项目？

应用场景：

- 远程分支与本地分支出现分支冲突，需要将远程分支拉到本地，即远程被`git push -f`强制推送
- 对本地进行提交时未及时同步远程分支
- 本地进行了修改，但未提交，这时候如果使用`git pull`命令，会提示本地有未缓存的修改。这时候就需要强制覆盖本地的改变。

【备份】在重置或操作之前可以通过从master创建一个分支来备份当前的本地提交：

```
git checkout master
git branch backup
git fetch --all
git reset --hard origin/master
```

在此之后，所有旧的提交都将保存在backup中。然而，没有提交的更改(即使staged)将会丢失。确保存储和提交任何你需要的东西。

1、如果无需保留本地更改，通过git命令强制覆盖本地：

```
git fetch --all && git reset --hard origin/master
```

> 如果您有任何本地更改，将会丢失。无论是否有--hard选项，任何未被推送的本地提交都将丢失。
>
> 如果您有任何未被Git跟踪的文件，这些文件将不会受到影响。

2、如果需要保留本地更改，如果可以通过分支代码回退进行同步远程分支（不对本地代码进行更改），此时有两种选择：

- 通过rebase命令：分支代码回退，当本地分支与远程分支不冲突时，将本地分支rebase到远程分支，即可更新，但这种方式进行同步有可能出现版本冲突，出现问题时需要解决，比较麻烦
- 通过merge命令：直接将远程分支合入本地分支，不产生冲突和额外的commit信息，推荐

3、可行的方式是通过使用`fetch`和`merge`定义的策略。这应该能使你的本地修改保留下来，只要它们不是你试图强制覆盖的文件之一。

首先做一个你的改变

```csharp
git add *
git commit -a -m "test commit message"
```

然后获取更改并覆盖，如果有冲突：

```
git fetch origin master
git merge -s recursive -X theirs origin/master
```

"-X"是选项名称，"theirs"是该选项的值。如果存在冲突，则选择使用"their"更改，而不是"your"更改。

4、所有这些解决方案的问题是，它们都是太复杂，或者更大的问题是，他们从本地仓库中中删除所有未跟踪的文件，这是我们不想要的，可以参考脚本：

```shell
# Fetch the newest code
git fetch

# Delete all files which are being added, so there
# are no conflicts with untracked files
for file in `git diff HEAD..origin/master --name-status | awk '/^A/ {print $2}'`
do
    rm -f -- "$file"
done

# Checkout all files which were locally modified
for file in `git diff --name-status | awk '/^[CDMRTUX]/ {print $2}'`
do
    git checkout -- "$file"
done

# Finally pull all the changes
# (you could merge as well e.g. 'merge origin/master')
git pull
```

第一个命令获取最新的数据。

第二个命令检查是否有任何正在添加到存储库的文件，并从本地存储库中删除那些会导致冲突的未跟踪文件。

第三个命令`checks-out`所有在本地修改的文件。

最后，我们将更新到最新版本，但是这次没有任何冲突，因为repo中的未跟踪文件不再存在，所有本地修改的文件已经与存储库中的相同。

### 将远程仓库更新取回本地

获取所有分支更新：

```
 git fetch --all
```

获取指定分支更新：

```
git fetch origin master
```

### 更新本地存放的远程仓库数据

将远程库与本地同步合并：

```
git pull --rebase origin master
```

### git远程分支已经删除，本地如何更新

当我们删除远程分支后执行

```
git branch -a
```

本地却依然能看到远程分支

这个时候我们只需要执行

```
git remote prune origin
```

清理远程分支信息


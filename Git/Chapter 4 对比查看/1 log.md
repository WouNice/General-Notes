# log

显示提交的记录。

## 查看提交日志

查看提交日志可以使用git log指令，语法格式如下：

```sh
git log [<options>] [<revision range>] [[\--] <path>…]
```

常用方式：

```bash
# 打印所有的提交记录
$ git log

# 打印从第一次提交到指定的提交的记录
$ git log <commit ID>

# 打印指定数量的最新提交的记录
$ git log -<指定的数量>
```

可以用 --oneline 选项来查看历史记录的简洁的版本。

```sh
$ git log --oneline
```

我们还可以用 --graph 选项，查看历史中什么时候出现了分支、合并。"git log --graph"以图形化的方式显示提交历史的关系，这就可以方便地查看提交历史的分支信息，我们可以清楚明了地看到何时工作分叉、又何时归并。

```sh
$ git log --oneline --graph
```

你也可以用`--reverse`参数来逆向显示所有日志。

```sh
git log --reverse --oneline
```

如果只想查找指定用户的提交日志可以使用命令：git log --author , 例如，比方说我们要找 Git 源码中 Linus 提交的部分：

```sh
git log --author=Linus  --oneline -5
```

如果你要指定日期，可以执行几个选项：--since 和 --before，但是你也可以用 --until 和 --after。例如，如果要查看 Git 项目中三周前且在四月十八日之后的所有提交（ --no-merges 选项以隐藏合并提交）：

```sh
git log --oneline --before={3.weeks.ago} --after={2010-04-18} --no-merges
```

## 查看所有分支日志

`git reflog`会记录这个仓库中所有的分支的所有更新记录，包括已经撤销的更新。

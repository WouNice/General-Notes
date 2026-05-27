# git reset

`git reset`：将当前 HEAD 重置为指定状态。

| 选项       | 说明                                                         |
| :--------- | :----------------------------------------------------------- |
| `--hard`   | 撤销并删除相应的更新                                         |
| `--soft`   | 撤销相应的更新，把这些更新的内容放到 Stage 中                |
| `--mixed`  | 重置暂存区，但文件不受影响（默认）                           |

## 撤销暂存区更新

如果已经用 `add` 命令把文件加入 stage 了，需要从 stage 中撤销：

```bash
# 重置暂存区，但文件不受影响
# 相当于将用 "git add" 命令更新到暂存区的内容撤出暂存区，可以指定文件
# 没有指定 commit ID 则默认为当前 HEAD
$ git reset [<文件路径>]
$ git reset --mixed [<文件路径>]
```

## 撤销本地仓库更新

撤销提交有两种方式：使用 HEAD 指针和使用 commit ID。

### 使用 HEAD 指针

在 Git 中，有一个 HEAD 指针指向当前分支中最新的提交。当前版本使用 `HEAD^`（再前一个版本可以使用 `HEAD^^`），如果想回退到更早的提交，可以使用 `HEAD~n`（`HEAD^` = `HEAD~1`，`HEAD^^` = `HEAD~2`）。

```bash
$ git reset --hard HEAD^
$ git reset --hard HEAD~1
```

示例：

```bash
$ git reset --hard HEAD@{7}
$ git reset --hard e0e79d7
```

### 使用 commit ID

通过 `git log` 查看提交日志，获取 commit ID：

```bash
# 将 HEAD 的指向改变，撤销到指定的提交记录，文件未修改
$ git reset <commit ID>
$ git reset --mixed <commit ID>

# 将 HEAD 的指向改变，撤销到指定的提交记录，文件未修改
# 相当于调用 "git reset --mixed" 命令后又做了一次 "git add"
$ git reset --soft <commit ID>

# 将 HEAD 的指向改变，撤销到指定的提交记录，文件也修改了
$ git reset --hard <commit ID>
```

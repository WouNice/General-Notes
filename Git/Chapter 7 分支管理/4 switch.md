# git switch

`git switch` 是 Git 2.23+ 引入的专门用于切换分支的命令，比 `git checkout` 更清晰、更安全。

```bash
# 切换到已存在的分支
$ git switch <分支名称>

# 创建并切换到新分支
$ git switch -c <新分支名称>

# 切换到上一个分支
$ git switch -
```

> `git switch` 只能切换分支，不能恢复文件，避免了 `git checkout` 的歧义。

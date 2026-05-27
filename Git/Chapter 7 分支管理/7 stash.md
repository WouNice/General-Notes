# git stash

`git stash` 用于临时保存工作目录的修改，让你能够切换分支或执行其他操作，稍后再恢复这些修改。

## 常用命令

```bash
# 保存当前修改到 stash 栈
$ git stash

# 保存修改并添加描述信息
$ git stash push -m "<描述信息>"

# 列出所有 stash
$ git stash list

# 恢复最近的 stash（默认 stash@{0}）
$ git stash pop

# 恢复指定的 stash
$ git stash pop stash@{n}

# 应用 stash 但不从栈中移除
$ git stash apply

# 删除最近的 stash
$ git stash drop

# 清空所有 stash
$ git stash clear

# 查看 stash 的详细内容
$ git stash show -p
```

## 示例

```bash
# 临时保存当前修改
git stash

# 切换到其他分支处理紧急问题
git checkout hotfix

# 处理完成后回到原分支
git checkout feature

# 恢复之前的修改
git stash pop
```

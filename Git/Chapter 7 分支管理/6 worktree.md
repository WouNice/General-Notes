# git worktree

`git worktree` 允许你在同一个仓库中同时检出多个分支到不同的工作目录。

这在需要同时处理多个分支时非常有用，避免了频繁切换分支或克隆多个仓库的麻烦。

## 常用命令

```bash
# 创建一个新的工作树，检出指定分支
$ git worktree add <路径> <分支名>

# 创建新分支并检出到新工作树
$ git worktree add -b <新分支名> <路径>

# 列出所有工作树
$ git worktree list

# 移除工作树
$ git worktree remove <路径>

# 清理已删除工作树的记录
$ git worktree prune
```

## 示例

```bash
# 在 ../feature-branch 目录检出 feature 分支
$ git worktree add ../feature-branch feature

# 进入该目录工作
cd ../feature-branch
```

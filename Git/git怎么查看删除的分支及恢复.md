# Git 查看已删除分支及恢复方法

## 查看已删除的分支

Git 提供了两种主要方式来查看已删除的分支记录，推荐优先使用 `git reflog`。

### 方法一：git reflog（推荐）

`git reflog` 记录了本地仓库中所有分支、提交等引用变更的完整历史，包括分支删除、切换等操作。

| 命令                            | 说明                                     |
| :------------------------------ | :--------------------------------------- |
| `git reflog`                    | 显示当前仓库的所有引用变更记录（最常用） |
| `git reflog show --all`         | 显示所有分支（包括远程）的完整记录       |
| `git reflog show <branch_name>` | 仅显示指定分支的操作历史                 |
| `git log -g`                    | 等效于 `git reflog`，以提交日志形式展示  |

**输出示例：**

```
abc1234 HEAD@{0}: commit: Fix bug in login
def5678 HEAD@{1}: checkout: moving from feature-branch to main
ghi9012 HEAD@{2}: branch: Delete feature-branch
```

其中 `branch: Delete` 的记录表明分支被删除，而它的前一条记录（`HEAD@{1}`）通常就是删除前最后一次提交。

### 方法二：git fsck（reflog 失效时）

当 reflog 记录因超过 90 天或被清理而失效时，`git fsck` 可扫描仓库中所有未被引用的"悬空"提交对象。

```bash
git fsck --lost-found
```

该命令会输出所有悬空对象的哈希值，可逐一检查提交内容来确认是否为目标分支。

## 恢复已删除的分支

找到目标提交的哈希值后，即可基于该哈希值重新创建分支。

### 从 git reflog 恢复（推荐）

**前提**：reflog 中存在分支删除前的记录。

```bash
# 创建并切换到新分支
git checkout -b <branch-name> <commit-hash>

# 或仅创建分支（不切换）
git branch <branch-name> <commit-hash>
```

### 从 git fsck 恢复

**前提**：reflog 中已无记录，但提交对象未被 Git 垃圾回收（GC）清理。

```bash
git branch <branch-name> <commit-hash>
git checkout <branch-name>
```

### 从远程仓库恢复

若删除的分支曾被推送到远程仓库，可通过远程引用来恢复。

```bash
# 拉取所有远程分支信息
git fetch --prune

# 基于远程分支创建本地分支
git checkout -b <branch-name> origin/<branch-name>
```

## 关键限制与注意事项

1. **垃圾回收（GC）时间窗口**：Git 默认保留 reflog 90 天，超过此期限后会被自动清理。主动运行 `git gc --prune=now` 则会立即清理，可能导致数据永久丢失。
2. **删除远程分支的风险**：若分支已被推送到远程并被删除，恢复依赖于本地是否还留有记录或他人是否持有副本。
3. **恢复前确认**：建议先用 `git show <commit-hash>` 或 `git log <commit-hash>` 检查提交内容，确保是正确目标。
4. **数据安全**：恢复操作本身是安全的，但在恢复前建议先备份当前工作区（`git stash`）以避免意外冲突。

## 进阶技巧

### 恢复被 git reset --hard 覆盖的提交

`git reset --hard` 会移动当前分支指针并丢弃工作区更改，但历史提交仍保留在 reflog 中。

```bash
# 找到 reset 前的提交哈希
git reflog

# 恢复到指定提交
git reset --hard <commit-hash>
```

### 恢复未提交但被丢弃的更改

```bash
git fsck --lost-found
```

然后在 `.git/lost-found/other/` 目录下查找悬空 blob 对象，这些可能是未跟踪或暂存区的文件内容。

### 防御性配置与习惯

- 对重要分支使用 `git branch -d`（安全删除）而非 `-D`（强制删除），避免误删未合并分支。
- 设置定期备份脚本，每日备份所有分支。
- 在 GitHub/GitLab 等平台启用分支保护（Protected Branches），禁止直接删除。
- 熟悉 `git reflog` 的输出格式，快速识别关键操作记录。

## 总结

| 方法         | 适用场景                   | 命令示例                                                     |
| :----------- | :------------------------- | :----------------------------------------------------------- |
| `git reflog` | 最近 90 天内删除的分支     | `git reflog` → `git checkout -b <branch> <commit>`           |
| `git fsck`   | reflog 已失效但提交未被 GC | `git fsck --lost-found` → `git branch <branch> <commit>`     |
| 远程仓库     | 分支曾被推送至远程         | `git fetch --prune` → `git checkout -b <branch> origin/<branch>` |

通过上述方法，绝大多数误删分支都可以安全恢复。养成定期备份、审慎使用强制命令的习惯，能有效降低数据丢失风险。

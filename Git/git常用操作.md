# Git 常用操作

本文档汇总 Git 日常开发中的高频操作命令，涵盖分支管理、文件清理、冲突解决等场景。

## 分支管理

### 查看分支信息

```bash
# 列出本地分支，当前分支以 * 标记
git branch

# 查看远程分支列表
git branch -r

# 查看所有分支（本地 + 远程，remotes/ 开头表示远程分支）
git branch -a

# 查看分支最后一次提交
git branch -v

# 查看已合并到当前分支的分支
git branch --merged

# 查看未合并的分支
git branch --no-merged
```

### 创建和切换分支

```bash
# 创建新分支
git branch <新分支名称>

# 切换分支
git checkout <分支名称>

# 创建并切换到新分支
git checkout -b <新分支名称>
```

### 从远程仓库拉取代码

```bash
# 指定远程分支拉取到本地分支
git pull origin <远程分支名称>:<本地分支名称>

# 省略本地分支名，默认与远程分支同名
git pull origin <远程分支名称>
```

> 可通过 `git remote -v` 查看远程仓库连接名称（默认为 origin）。

### 推送分支到远程仓库

```bash
# 推送本地分支到远程
git push origin <分支名称>

# 首次推送并建立追踪关系
git push --set-upstream origin <分支名称>
# 简写
git push -u origin <分支名称>
```

**推送模式说明：**

| 模式      | 说明                                                         |
| :-------- | :----------------------------------------------------------- |
| `simple`  | 默认模式，仅推送当前分支到同名的远程追踪分支（Git 2.0+）       |
| `matching`| 推送所有与远程分支同名的本地分支（Git 2.0 之前默认）           |

### 删除分支

```bash
# 删除本地分支（不能删除当前所在分支）
git branch -d <分支名称>

# 强制删除未完全合并的分支
git branch -D <分支名称>

# 删除远程分支
git push origin :<分支名称>
```

### 合并分支

以将 `dev` 分支合并到 `master` 为例：

```bash
# 1. 切换到目标分支
git checkout master

# 2. 拉取最新远程代码（多人开发时建议执行）
git pull origin master

# 3. 合并 dev 分支
git merge dev

# 4. 若出现冲突，可取消合并
git merge --abort

# 5. 查看状态
git status

# 6. 推送合并结果到远程
git push origin master
```

## 文件清理

### 完全清除本地改动

```bash
# 用远程 origin/master 强制替换本地、暂存区、版本库
git reset --hard origin/master
```

### 删除未跟踪文件

```bash
# 重置到 HEAD
git reset --hard HEAD

# 删除未跟踪的文件和目录
git clean -f -d

# 快捷命令：删除所有未跟踪文件并还原工作区
git clean -xdf && git checkout .
```

> `git clean` 操作不可撤销，请谨慎使用。

## 冲突解决

### 解决 JetBrains IDE 显示 "正在 rebase" 的问题

变基失败或取消后，JetBrains 可能持续显示 rebase 状态：

```bash
# 完全撤销变基，恢复到 rebase 之前的状态
git rebase --abort

# 跳过有问题的提交
git rebase --skip
```

### 强制拉取远程项目覆盖本地

**适用场景：**

- 远程分支被 `git push -f` 强制覆盖
- 本地未及时同步远程分支
- 本地有未提交的修改导致无法 `git pull`

**备份当前工作：**

```bash
git checkout master
git branch backup
git fetch --all
git reset --hard origin/master
```

> 未提交的更改（包括已暂存）将会丢失。未被 Git 跟踪的文件不受影响。

**方案 1：不保留本地更改（强制覆盖）**

```bash
git fetch --all && git reset --hard origin/master
```

**方案 2：保留本地更改（推荐 merge）**

```bash
git add .
git commit -a -m "临时提交本地更改"
git fetch origin master
git merge -s recursive -X theirs origin/master
```

> `-X theirs` 表示冲突时优先采用远程更改。

**方案 3：保留本地更改并避免删除未跟踪文件**

```bash
# 获取最新数据
git fetch

# 删除远程新增但会导致冲突的文件
for file in $(git diff HEAD..origin/master --name-status | awk '/^A/ {print $2}')
do
    rm -f -- "$file"
done

# 还原本地修改的文件
for file in $(git diff --name-status | awk '/^[CDMRTUX]/ {print $2}')
do
    git checkout -- "$file"
done

# 拉取更新
git pull
```

### 获取远程更新

```bash
# 获取所有分支更新
git fetch --all

# 获取指定分支更新
git fetch origin master

# 使用 rebase 方式同步
git pull --rebase origin master
```

### 清理已删除的远程分支

```bash
# 远程分支删除后，本地仍能看到残留引用
git remote prune origin
```

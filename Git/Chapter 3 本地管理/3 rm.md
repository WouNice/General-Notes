# git rm

`rm` 命令用于从工作树和索引中删除文件。

如果只是简单地从工作目录中手工删除文件，运行 `git status` 时就会在 `Changes not staged for commit` 的提示。

要从 Git 中移除某个文件，就必须要从已跟踪文件清单中移除，可以用以下命令完成此项工作：

```bash
# 删除文件
git rm <file>

# 强制删除（删除之前修改过并且已经放到暂存区域）
git rm -f <file>

# 递归删除目录
git rm -r <文件夹路径>
```

## 其他相关命令

```bash
# 从暂存区删除文件，工作区不做出改变
git rm --cached <file>

# 移除工作区的所有未跟踪文件
git clean [options]

# 常用：删除未跟踪的文件和目录
git clean -df
```

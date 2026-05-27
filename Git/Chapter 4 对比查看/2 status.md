# git status

`git status` 命令用于显示工作目录和暂存区的状态。使用此命令能看到哪些修改被暂存了，哪些没有，哪些文件没有被 Git tracked 到。

显示索引文件和当前 HEAD 提交之间存在差异的路径，工作树和索引文件之间存在差异的路径，以及工作树中未被 Git 跟踪（并且未被 `.gitignore` 忽略的路径）。第一个是您将通过运行提交的内容；第二个和第三个是在运行 `git commit` 之前运行 `git add` 可以提交的内容。

## 常用命令

```bash
# 查看本地仓库的状态
$ git status

# 查看指定文件状态
$ git status <filename>

# 简短的结果输出
$ git status -s
```

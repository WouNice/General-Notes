# git mergetool

`git mergetool` 用于启动可视化合并工具来解决合并冲突。

当 `git merge` 遇到冲突时，可以使用 mergetool 来图形化地解决冲突：

```bash
# 启动默认的合并工具
$ git mergetool

# 使用指定的工具
$ git mergetool --tool=<tool-name>
```

常用合并工具包括：vimdiff、meld、Beyond Compare、KDiff3 等。

配置默认合并工具：

```bash
$ git config --global merge.tool <tool-name>
```

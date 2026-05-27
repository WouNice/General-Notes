# git revert

可以通过 `git revert <commit-id>` 撤销某次提交，这条命令会把指定的提交的所有修改回滚，并同时生成一个新的提交。

```bash
# 生成一个新的提交来撤销某次提交
$ git revert <commit ID>
```

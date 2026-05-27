# git push

将本地分支的更新，推送到远程主机。

## 语法格式

```bash
git push <远程主机名> <本地分支名>:<远程分支名>
```

## 常用命令

```bash
# 将本地的 master 分支推送到 origin 主机的 master 分支
# 如果 master 不存在，则会被新建
$ git push origin master

# 如果省略远程分支名，则表示将本地分支推送与之存在"追踪关系"的远程分支
# 如果该远程分支不存在，则会被新建
$ git push origin master

# 如果省略本地分支名，则表示删除指定的远程分支
# 因为这等同于推送一个空的本地分支到远程分支
$ git push origin :master
# 等同于
$ git push origin --delete master

# 建立当前分支与远程分支的追踪关系
$ git push -u origin master

# 当前分支与远程分支之间存在追踪关系，则可以省略分支名和远程主机名
$ git push

# 不管是否存在对应的远程分支，将本地的所有分支都推送到远程主机
$ git push --all origin

# 强制推送
$ git push --force origin

# 推送标签
$ git push origin --tags
```

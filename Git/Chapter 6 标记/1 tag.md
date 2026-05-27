# git tag

操作标签的命令。

## 添加标签

如果希望标记一个特别的快照，可以使用 `git tag` 命令给它打上标签。

比如说，想为项目发布 "1.0" 版本。可以用 `git tag -a v1.0` 命令给最新一次提交打上（HEAD）"v1.0" 的标签。`-a` 选项意为"创建一个带注解的标签"。

```bash
$ git tag -a v1.0
```

当你执行 `git tag -a` 命令时，Git 会打开你的编辑器，让你写一句标签注解，就像你给提交写注解一样。

也可以直接指定标签信息：

```bash
$ git tag -a <tagname> -m "<标签信息>"
```

## 查看标签

使用 `git tag` 查看所有标签：

```bash
$ git tag
```

查看指定标签的信息：

```bash
$ git show <标签名称>
```

使用 `git log --decorate` 查看标签详情：

```bash
$ git log --decorate
```

输出示例：

```
commit d25e7e272bffbd997cad49274ca7a3b64c07040e (HEAD -> master, tag: v1.0)
Author: YaoLiang <vtearth@qq.com>
Date:   Thu Jul 2 11:38:53 2020 +0800
    '第一次版本提交'
```

## 切换到指定的标签

```bash
$ git checkout <标签名称>
```

## 追加标签

例如，假设发布了提交 `<commit ID>`，但是忘了给它打标签。可以通过如下命令追加标签：

```bash
# 添加轻量标签，指向提交对象的引用，可以指定之前的提交记录
$ git tag <标签名称> [<commit ID>]

# 添加带有描述信息的附注标签，可以指定之前的提交记录
$ git tag -a <标签名称> -m <标签描述信息> [<commit ID>]
```

## 删除标签

删除指定的标签：

```bash
$ git tag -d <标签名>
```

## 推送标签

```bash
# 将指定的标签提交到远程仓库
$ git push <远程仓库的别名> <标签名称>

# 将本地所有的标签全部提交到远程仓库
$ git push <远程仓库的别名> --tags
```

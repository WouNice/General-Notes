# git remote

为了便于管理，Git 要求每个远程主机都必须指定一个主机名。`git remote` 命令就用于管理主机名。

## 列出远程主机

使用 `git remote` 命令列出所有远程主机：

```bash
$ git remote
origin
```

使用 `-v` 或者 `--verbose` 选项，可以查看远程主机的网址：

```bash
$ git remote -v
origin  git@github.com:xxx/example.git (fetch)
origin  git@github.com:xxx/example.git (push)
```

## 添加远程主机

克隆版本库的时候，所使用的远程主机自动被 Git 命名为 `origin`。如果想用其他的主机名，需要用 `git clone` 命令的 `-o` 选项指定：

```bash
$ git clone -o github https://github.com/xxx/xxx.git
```

添加远程仓库：

```bash
$ git remote add <远程仓库的别名> <远程仓库的URL地址>
```

## 查询主机

`git remote show` 命令加上主机名，可以查看该主机的详细信息：

```bash
$ git remote show <主机名>
```

## 重命名远程主机

```bash
$ git remote rename <原主机名> <新主机名>
```

## 删除远程主机

```bash
$ git remote rm <主机名>
```

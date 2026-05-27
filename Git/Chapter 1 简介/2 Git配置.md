# Git 配置

配置 Git 的相关参数，其主要操作有：

```bash
# 查看配置信息
# --local：仓库级，--global：全局级，--system：系统级
$ git config <--local | --global | --system> -l

# 查看当前生效的配置信息
$ git config -l

# 编辑配置文件
$ git config <--local | --global | --system> -e

# 添加配置项
$ git config <--local | --global | --system> --add <name> <value>

# 获取配置项
$ git config <--local | --global | --system> --get <name>

# 删除配置项
$ git config <--local | --global | --system> --unset <name>

# 更改 Git 缓存区的大小
# 如果提交的内容较大，默认缓存较小，提交会失败
# 缓存大小单位：B，例如：524288000（500MB）
$ git config --global http.postBuffer <缓存大小>

# 调用 git status/git diff 命令时以高亮或彩色方式显示改动状态
$ git config --global color.ui true

# 配置可以缓存密码，默认缓存时间 15 分钟
$ git config --global credential.helper cache

# 配置密码的缓存时间
# 缓存时间单位：秒
$ git config --global credential.helper 'cache --timeout=<缓存时间>'

# 配置长期存储密码
$ git config --global credential.helper store
```

## 配置文件

Git 环境配置决定了 Git 在各个环节的具体工作方式和行为。

Git 配置文件可以存放在以下三个不同的地方：

| 级别   | 文件路径           | 作用范围           |
| :----- | :----------------- | :----------------- |
| system | `/etc/gitconfig`   | 系统中对所有用户都普遍适用的配置 |
| global | `~/.gitconfig`     | 用户目录下的配置文件，只适用于该用户 |
| local  | `.git/config`      | 仅对当前项目有效 |

> 对于同一配置项，三个配置文件的优先级是：**local > global > system**。

## 配置分类

`git config` 命令用来配置或读取相应的工作环境变量：

```
git config [--local|--global|--system] section.key value
```

- `[--local|--global|--system]`：可选，对应本地、全局、系统不同级别的设置
- `section`：区域
- `section.key`：区域下的键
- `value`：对应的值

## 查看配置

要检查已有的配置信息，可以使用 `git config --list` 命令查看不同级别的配置文件：

```bash
$ git config [--local|--global|--system] --list
```

也可以直接查阅某个环境变量的值：

```bash
$ git config <配置项>
```

示例：

```bash
$ git config user.name  # 获得用户名
```

可能会看到重复的变量名，因为 Git 会从不同的文件中读取同一个配置（例如：`/etc/gitconfig` 与 `~/.gitconfig`）。这种情况下，Git 会使用它找到的每一个变量的最后一个配置，此时可以通过 `[--local|--global|--system]` 指定查询配置的级别。

## 配置用户名与邮箱

用户名与邮箱作为用户标识，当你安装 Git 后，必须要设置。因为每次 Git 提交都会使用该信息，它会自动嵌入到你的提交中：

```bash
# 配置提交记录中的用户信息
$ git config --global user.name <用户名>
$ git config --global user.email <邮箱地址>
```

## 删除配置

```bash
$ git config [--local|--global|--system] --unset section.key
```

## 编辑配置文件

```bash
$ git config -e [--local|--global|--system]
```

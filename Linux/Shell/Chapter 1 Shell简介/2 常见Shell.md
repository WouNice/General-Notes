# 常见Shell

Shell 是一种脚本语言。那么，就必须要有对应的解释器来执行这些脚本。

`/etc/shells` 文件记录了系统上可使用的 shell 软件，使用 `cat` 命令查看该文件：

```bash
# cat /etc/shells
/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash
```

可以看到，一个操作系统上可能内置了多个 shell 软件，其中最常见的两个是：

- `/bin/sh`：一般情况下，也是系统的默认 shell。
- `/bin/bash`: 在 sh 的基础上增加了一些实用特性，是使用最广泛的 shell。

## bash

bash 是 Linux 标准默认的 shell，具有以下的特色：

- 可以使用类似 DOS 下面的 doskey 的功能，用方向键查阅和快速输入并修改命令。
- 自动通过查找匹配的方式给出以某字符串开头的命令。
- 包含了自身的帮助功能，你只要在提示符下面键入 help 就可以得到相关的帮助。

## sh

sh 由 Steve Bourne 开发，是 Bourne Shell 的缩写，sh 是 Unix 标准默认的 shell。

> 一般情况下，我们不区分bash和sh，比如`#!/bin/sh` 和 `#!/bin/bash`是一样的。

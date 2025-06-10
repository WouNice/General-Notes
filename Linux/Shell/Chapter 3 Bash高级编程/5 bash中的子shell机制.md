# bash中的子shell机制

执行脚本或者 bash 命令时，会创建一个子 shell，子 shell 会继承父 shell 的环境变量（不包括普通变量），子 shell 中设置的环境变量不会影响到父 shell。

类似于父进程与子进程之间的关系，在 `shell` 中创建的 `shell` 实例被称为子 `shell`。在 `bash` 中，子 `shell` 可以分为以下两类：

-   `sub-shell` : 通过进程替换、命令替换、`(LIST)`、`|` 或 `&` 隐式生成的子 `shell` 被称作 `sub-shell` 。父 `shell` 通过 `fork` 创建 `sub-shell`，因此 `sub-shell` 可以得到父 `shell` 中代码段、数据段、堆、栈的完整拷贝。
-   `child-shell` : 通过以可执行文件的方式运行 `shell` 脚本或直接在当前 `shell` 中启动 `shell` 解释器的方式得到的子 `shell` 被称作 `child-shell` 。父 `shell` 通过 `fork-exec` 模式创建 `child-shell`，这导致 `child-shell` 中的代码段、数据段、堆、栈完全被新的 `shell` 实例替换，但可以访问父 `shell` 通过 `export` 导出的环境变量。

接下来我们通过一些具体的例子进一步理解这两种子 `shell` 的区别。

## sub-shell

在 `sub-shell` 中通过 `fork` 获得了父 `shell` 中变量、函数、别名等数据的完整拷贝，因此二者之间的数据是相互独立的。这意味着在 `sub-shell` 中修改这些数据无法对父 `shell` 中的原始数据造成影响。此外，通过同一个命令创建的多个 `sub-shell` 中的数据也是相互独立的，例如：在管道中修改变量的值无法被保存。

```bash
> a=10
> { a=5; echo left $a; }>&2 | echo right $a
right 10
left 5  # 左右两侧都在各自创建的 subshell 中执行 互不影响
> echo $a
10  # 在 subshell 中修改变量不会影响父 shell
```

为了方便管理 `sub-shell`，在 `bash` 中提供了 `BASH_SUBSHELL` 变量用于记录 `sub-shell` 的嵌套层级。

```bash
> function bs() { echo $BASH_SUBSHELL >&2; }
> bs;`bs`;(`bs`);(`bs &`);(`cat <(bs &)`)
0  # 父 shell 中的 sub-shell 嵌套层级为 0
1  # 命令替换嵌套一层
2  # 命令替换加小括号嵌套两层
3  # 命令替换加小括号加后台任务嵌套三层
4  # 命令替换加小括号加后台任务加进程替换嵌套四层
```

>   注意：`BASH_SUBSHELL` 变量是在 `3.0.0` 版本的 `bash` 中引入的，但直到目前最新的 `5.0.3` 版本仍然存在着管道单命令 `BASH_SUBSHELL` 变量值显示异常的 `BUG`，可以暂时使用 `{ ... }` 包裹命令的方式避免这个问题。

在 `bash` 进行初始化时会将当前的进程号保存在变量 `$$` 中，而在创建 `sub-shell` 时由于直接使用 `fork` 拷贝了父 `shell` 中的数据，因此没有初始化过程，这导致在 `sub-shell` 中使用 `$$` 会获得父 `shell` 的进程号。

```bash
# { pstree -ap $$; echo $BASH_SUBSHELL $$; } | { cat; }
bash,1552
  |-bash,108847
  |   `-pstree,108850 -ap 1552
  `-bash,108848
      `-cat,108849
1 1552  # sub-shell 层数不为零却输出了父 shell 的进程号
```

通过 `$BASHPID` 可以动态获取当前 `bash` 的进程号，这种方式可以正确获取 `sub-shell` 的进程号。`BASHPID` 是特殊变量，普通的变量赋值方式对其无效。但通过 `unset` 删除该变量之后重新赋值可以让其变为普通变量，失去动态获取当前进程号的功能。

```bash
> pf () { echo $BASH_SUBSHELL $$ $PPID $BASHPID; }
> pf;(pf;(pf;(pf;pstree -ap $$)))
0 9411 18443 9411
1 9411 18443 24142
2 9411 18443 24143
3 9411 18443 24144  # 通过 BASHPID 可以获得正确的进程号
bash,9411
  └─bash,24142
      └─bash,24143
          └─bash,24144
              └─pstree,24145 -ap 9411
```

父 `shell` 中的某些选项不会被 `sub-shell` 继承，例如: `sub-shell` 中可以看到父 `shell` 设置的 `trap` 但并不会生效，因为 `trap` 的工作原理是将回调函数注册到当前进程并维护注册信息，而在 `sub-shell` 中只继承了 `trap` 的注册信息却没有进行回调函数注册。

```bash
> trap "echo done" SIGTERM
> (trap -p; kill $BASHPID; echo "Never print")
trap -- 'echo done' SIGTERM
Terminated  # 没有执行设置的 trap

> (trap "echo done" SIGTERM; trap -p; kill $BASHPID; echo "Never print")
trap -- 'echo done' SIGTERM
done  # 再次注册回调函数 因此可以正确地进行拦截
Never print
```

## child-shell

显式获得 `child-shell` 的方法是在当前 `shell` 中以执行外部命令的方式启动 `shell` 程序。与 `sub-shell` 不同的是，`child-shell` 只能继承父 `shell` 通过 `export` 导出的变量。本质上 `child-shell` 就是一个通过 `fork-exec` 模式启动的子进程，只不过这个子进程恰好是个 `shell`。

```bash
> unset a; a=1
> (echo "a is $a in the subshell")
a is 1 in the subshell  # 仅 fork 完全拷贝
> bash -c 'echo "a is $a in the child shell"'
a is  in the child shell  # fork 并 exec 导致数据被覆盖
```

但 `child-shell` 确实是一个特殊的子进程，为了方便管理，在 `sh` 和 `bash` 中提供了 `SHLVL` 变量用于记录 `child-shell` 的嵌套层级，以父 `shell` 的嵌套值为 `1` 开始递增。

```bash
> (echo $SHLVL $BASH_SUBSHELL;(echo $SHLVL $BASH_SUBSHELL;(echo $SHLVL $BASH_SUBSHELL)))
1 1
1 2
1 3
> bash  # 启动一个 child-shell
> (echo $SHLVL $BASH_SUBSHELL;(echo $SHLVL $BASH_SUBSHELL;(echo $SHLVL $BASH_SUBSHELL)))
2 1
2 2
2 3
```

在 `shell` 中以可执行文件的方式运行脚本时，在 `fork` 之后会通过 `execl` 系统调用将可执行文件传递给内核处理，如果在可执行文件第一行设置了 `#![解释器]`，则内核会断定该文件为脚本文件，并新建解释器实例来执行该脚本，如果解释器恰好是 `bash` 或 `sh` 之类的 `shell` 则会生成 `child-shell`。另一种执行脚本的方式是在当前 `shell` 中以外部命令的方式启动解释器并传递脚本文件作为参数。

```bash
> my.sh
#!/bin/bash
pstree -pa $PPID

> bash my.sh  # 显式创建 child-shell
bash,27454
  └─bash,27574 my.sh
      └─pstree,27575 -pa 27454

> ./my.sh  # 隐式创建 child-shell, 解释器由 shabang 决定
bash,27454
  └─ko.sh,27584 ./my.sh
      └─pstree,27585 -pa 27454

> python real.py  # 创建了个子进程但不是 child-shell
```

使用内置命令 `source` 和 `.` 可以在当前 `shell` 中执行脚本文件，其工作原理是以行为单位将脚本文件内容读到缓冲区中，每行命令都像直接来自键盘一样被读取、解释和执行，这样可以避免创建 `child-shell` 但也可能会污染当前 `shell` 中的环境。

```bash
. my.sh  # 不会创建 child-shell
source my.sh  # 不会创建 child-shell，因为 `source` 是 `.` 的别称
```

此外，内置命令 `exec` 也可以用于执行脚本文件，如果脚本文件中没有使用 `#!` 指定解释器，则会使用当前 `shell` 的新实例替换当前 `shell` 并解释脚本。

```bash
> cat ko.sh
echo a=$a pid=$BASHPID $SHLVL $BASH_SUBSHELL
> bash -c "a=5; source ko.sh; exec ./ko.sh"
a=5 pid=24540 2 0
a= pid=24540 2 0  # PID 未变且不可以访问变量 a, 说明 exec 用新的 bash 实例替换了当前 shell

> sh -c "exec ./ko.sh"
a= pid= 1  # 当前 shell 为 sh, 调用 exec 之后使用 sh 解释脚本
```


# Bash 子 Shell 机制

> **本章概述**：Bash 中的子 Shell 分为 sub-shell 和 child-shell 两种，它们在创建方式、数据继承、变量可见性等方面存在本质差异。理解这两种子 Shell 的机制，是掌握 Bash 变量作用域、进程关系和脚本执行方式的关键。

## 子 Shell 的分类

| 类型 | 创建方式 | 创建机制 | 数据继承 | 进程关系 |
|------|---------|---------|---------|---------|
| **sub-shell** | `(...)`、管道 `|`、命令替换 `$()`、进程替换 `<()`、后台 `&` | `fork`（不 exec） | 完整拷贝父 Shell 的代码段、数据段、堆、栈 | 与父 Shell 共享同一进程的程序映像 |
| **child-shell** | 执行脚本、`bash` 命令 | `fork` + `exec` | 仅继承 `export` 导出的环境变量 | 全新的 Shell 实例，程序映像被替换 |

**核心区别**：

- **sub-shell**：通过 `fork` 直接复制父 Shell 的内存，因此能访问所有变量（包括未 export 的），但修改不影响父 Shell
- **child-shell**：通过 `fork-exec` 加载全新 Shell 实例，原有数据被完全替换，只能看到环境变量

## sub-shell 详解

### 创建方式

sub-shell 在以下场景中被隐式创建：

| 创建方式 | 示例 | 说明 |
|---------|------|------|
| 小括号 `(LIST)` | `(a=5; echo $a)` | 在独立 sub-shell 中执行命令列表 |
| 管道 `|` | `echo hi | cat` | 管道两侧各自在 sub-shell 中运行 |
| 命令替换 `$(...)` 或 `` `...` `` | `echo $(date)` | 在 sub-shell 中执行命令并获取输出 |
| 进程替换 `<(...)` / `>(...)` | `diff <(sort a) <(sort b)` | 在 sub-shell 中运行并暴露为文件描述符 |
| 后台任务 `&` | `sleep 10 &` | 在 sub-shell 中后台运行 |

### 变量隔离性

sub-shell 中对变量的修改**不会影响父 Shell**：

```bash
a=10
{ a=5; echo left $a; }>&2 | echo right $a
# right 10  ← 管道右侧在 sub-shell 中，看到原始值
# left 5    ← 管道左侧在 sub-shell 中，修改仅局部生效

echo $a
# ← 父 Shell 中的值未变
```

管道中修改变量无法保存：

```bash
echo "hello" | read line
echo "$line"
# 空  ← read 在 sub-shell 中执行，变量赋值丢失
```

> **解决方法**：使用进程替换或 `lastpipe` 选项避免 sub-shell：

```bash
# 方法1：进程替换
read line < <(echo "hello")
echo "$line"  # hello

# 方法2：shopt lastpipe（Bash 4.2+）
shopt -s lastpipe
echo "hello" | read line
echo "$line"  # hello
```

### 嵌套层级：BASH_SUBSHELL

`BASH_SUBSHELL` 变量记录 sub-shell 的嵌套层级，父 Shell 中值为 0，每进入一层 sub-shell 加 1。

```bash
function bs() { echo $BASH_SUBSHELL >&2; }

bs;`bs`;(`bs`);(`bs &`);(`cat <(bs &)`)
# 0  ← 父 Shell
# 1  ← 命令替换嵌套一层
# 2  ← 命令替换 + 小括号嵌套两层
# 3  ← 命令替换 + 小括号 + 后台任务嵌套三层
# 4  ← 命令替换 + 小括号 + 后台任务 + 进程替换嵌套四层
```

> **已知 Bug**：Bash 5.0.3 及之前版本存在管道单命令 `BASH_SUBSHELL` 值显示异常的问题，可使用 `{ ... }` 包裹命令避免。

### 进程号：$$ 与 BASHPID

| 变量 | 行为 | 说明 |
|------|------|------|
| `$$` | 返回父 Shell 的 PID | sub-shell 通过 `fork` 创建，未重新初始化，`$$` 仍为父 Shell 的值 |
| `$BASHPID` | 返回当前 Bash 进程的真实 PID | 动态获取，每次展开都返回当前进程号 |
| `$PPID` | 返回父进程的 PID | sub-shell 中的父进程即为父 Shell |

```bash
pf () { echo "$BASH_SUBSHELL $$ $PPID $BASHPID"; }

pf;(pf;(pf;(pf;pstree -ap $$)))
# 0 9411 18443 9411    ← 父 Shell: $$ == BASHPID
# 1 9411 18443 24142   ← sub-shell: $$ 仍为父 PID，BASHPID 为真实 PID
# 2 9411 18443 24143
# 3 9411 18443 24144

# 进程树验证
bash,9411
  └─bash,24142
      └─bash,24143
          └─bash,24144
              └─pstree,24145 -ap 9411
```

> **BASHPID 的特殊性**：它是特殊变量，普通赋值无效。但 `unset BASHPID` 后重新赋值可使其变为普通变量，失去动态获取功能。

### trap 在 sub-shell 中的行为

sub-shell 可以**看到**父 Shell 的 `trap` 注册信息（因为 `fork` 拷贝了数据），但**不会生效**——因为 `trap` 的工作原理是将回调注册到当前进程，sub-shell 只继承了注册信息却没有进行注册。

```bash
# 设置 trap
trap "echo done" SIGTERM

# sub-shell 中可以看到但不会执行
(trap -p; kill $BASHPID; echo "Never print")
# 输出: trap -- 'echo done' SIGTERM
#        Terminated  ← 没有执行自定义 trap

# 需要重新注册才能生效
(trap "echo done" SIGTERM; trap -p; kill $BASHPID; echo "Never print")
# 输出: trap -- 'echo done' SIGTERM
#        done  ← 重新注册后生效
#        Never print
```

## child-shell 详解

### 创建方式

child-shell 通过以下方式显式创建：

| 创建方式 | 示例 | 说明 |
|---------|------|------|
| 启动新的 Bash | `bash` | 在当前 Shell 中启动新的 Shell 实例 |
| 使用 `bash -c` | `bash -c 'echo hi'` | 在 child-shell 中执行字符串命令 |
| 执行脚本（有 shebang） | `./script.sh` | 由 shebang 指定的解释器创建 child-shell |
| 执行脚本（显式指定） | `bash script.sh` | 使用指定的 Shell 创建 child-shell |

### 变量可见性

child-shell **只能**继承父 Shell 通过 `export` 导出的环境变量：

```bash
unset a; a=1

# sub-shell：完整拷贝，可访问
(echo "a is $a in the subshell")
# a is 1 in the subshell

# child-shell：仅继承环境变量
bash -c 'echo "a is $a in the child shell"'
# a is  in the child shell  ← 未 export，不可见

# export 后可见
export a
bash -c 'echo "a is $a in the child shell"'
# a is 1 in the child shell
```

### 嵌套层级：SHLVL

`SHLVL` 变量记录 child-shell 的嵌套层级，父 Shell 的值为 1，每启动一层 child-shell 加 1。

```bash
# sub-shell 不影响 SHLVL
(echo $SHLVL $BASH_SUBSHELL;(echo $SHLVL $BASH_SUBSHELL;(echo $SHLVL $BASH_SUBSHELL)))
# 1 1
# 1 2
# 1 3

# child-shell 增加 SHLVL
bash
(echo $SHLVL $BASH_SUBSHELL;(echo $SHLVL $BASH_SUBSHELL))
# 2 1
# 2 2
```

### 脚本执行方式对比

| 执行方式 | Shell 类型 | 变量可见性 | 对当前 Shell 的影响 |
|---------|-----------|-----------|-------------------|
| `./script.sh` | child-shell（由 shebang 决定） | 仅环境变量 | 无影响 |
| `bash script.sh` | child-shell | 仅环境变量 | 无影响 |
| `source script.sh` / `. script.sh` | 当前 Shell | 所有变量 | **可能污染当前环境** |
| `exec ./script.sh` | 替换当前 Shell | 仅环境变量 | 当前 Shell 被替换 |

**脚本执行过程的底层机制**：

```
./script.sh（有 shebang: #!/bin/bash）
  ↓
父 Shell fork 子进程
  ↓
内核读取 shebang，确定解释器
  ↓
内核创建解释器实例（child-shell）执行脚本

bash script.sh
  ↓
父 Shell fork 子进程
  ↓
exec 加载 bash，创建 child-shell 执行脚本
```

### source 与 exec

**source / . 命令**：

在当前 Shell 中执行脚本，不创建子 Shell。以行为单位将脚本内容读入缓冲区执行：

```bash
source my.sh   # 不会创建 child-shell
. my.sh        # 等价写法（. 是 source 的别称）
```

> **风险**：脚本中的变量赋值、函数定义、环境修改都会影响当前 Shell，可能造成环境污染。

**exec 命令**：

用新程序替换当前进程。如果脚本没有 shebang，则用当前 Shell 的新实例替换：

```bash
# 替换当前 Shell
exec ./script.sh
# 当前 Shell 被 script.sh 的解释器替换，PID 不变

# 替换为指定程序
exec python3 script.py
# 当前 Shell 被替换为 Python 进程
```

**验证 exec 替换行为**：

```bash
cat ko.sh
# echo a=$a pid=$BASHPID $SHLVL $BASH_SUBSHELL

bash -c "a=5; source ko.sh; exec ./ko.sh"
# a=5 pid=24540 2 0  ← source 在当前 shell 执行，可访问 a
# a= pid=24540 2 0   ← exec 替换后，新实例无法访问变量 a

sh -c "exec ./ko.sh"
# a= pid= 1  ← sh 环境下 exec，变量和环境都不同
```

## 变量继承关系总览

### sub-shell vs child-shell 变量可见性

| 变量类型 | sub-shell | child-shell | 说明 |
|---------|:---------:|:----------:|------|
| 普通变量 | ✅ 可见 | ❌ 不可见 | `a=1` |
| 环境变量（export） | ✅ 可见 | ✅ 可见 | `export a=1` |
| 只读变量 | ✅ 可见 | ❌ 不可见 | `readonly a=1` |
| 函数 | ✅ 可见 | ❌ 不可见 | `func() { ... }` |
| 别名 | ✅ 可见 | ❌ 不可见 | `alias ll='ls -l'` |
| trap 设置 | ✅ 可见但不生效 | ❌ 不可见 | — |
| 选项（shopt） | ✅ 可见 | 部分可见 | — |

### 特殊变量行为对比

| 变量 | sub-shell | child-shell | 说明 |
|------|-----------|-------------|------|
| `$$` | 父 Shell PID | 自身 PID | child-shell 有初始化过程 |
| `$BASHPID` | 自身 PID | 自身 PID | 动态获取，始终正确 |
| `$PPID` | 父进程 PID | 父进程 PID | — |
| `$BASH_SUBSHELL` | 嵌套层级 | 始终为 0 | child-shell 是独立实例 |
| `$SHLVL` | 不变 | +1 | child-shell 嵌套加 1 |
| `$?` | 继承父 Shell | 独立 | — |

## 相关命令参考

### exec 命令

`exec` 是 Bash 内置命令，用于替换当前进程或在当前 Shell 中操作文件描述符。

**语法**：

```bash
exec [-cl] [-a 名称] [命令 [参数 ...]] [重定向 ...]
```

**参数说明**：

| 参数 | 说明 |
|------|------|
| `-a 名称` | 将名称作为第 0 个参数传递给命令 |
| `-c` | 在空环境中执行命令 |
| `-l` | 在第 0 个参数前加短横线（模拟登录 Shell） |
| `命令` | 替换当前进程执行的命令；无命令时仅处理重定向 |
| `重定向` | 在当前 Shell 中打开/关闭/重定向文件描述符 |

**常用示例**：

```bash
# 替换当前 Shell 为新程序
exec bash            # 重新启动当前 Shell
exec /bin/zsh        # 切换到 zsh

# 用脚本替换当前 Shell
exec ./my_script.sh

# 在当前 Shell 中重定向（不替换进程）
exec 3>/tmp/output   # 打开文件描述符 3 写入
exec 3>&-            # 关闭文件描述符 3
exec >logfile 2>&1   # 将当前 Shell 的所有输出重定向到文件

# 读取文件描述符
exec 3</tmp/input
read line <&3
exec 3<&-
```

### source / . 命令

**语法**：

```bash
source 文件名 [参数 ...]
. 文件名 [参数 ...]
```

| 参数 | 说明 |
|------|------|
| `文件名` | 要在当前 Shell 中执行的脚本文件 |
| `参数` | 传递给脚本的位置参数（`$1`, `$2`, ...） |

**示例**：

```bash
# 在当前 Shell 中执行配置文件
source ~/.bashrc
. ~/.bashrc

# 带参数执行
source mylib.sh arg1 arg2
```

> **注意**：`source` 和 `.` 完全等价，`source` 是 Bash 扩展，`.` 是 POSIX 标准。在 `sh` 脚本中应使用 `.`。

### export 命令

**语法**：

```bash
export [-fn] [名称[=值] ...]
export -p
```

**参数说明**：

| 参数 | 说明 |
|------|------|
| `-f` | 导出函数 |
| `-n` | 取消导出（变量变为普通变量） |
| `-p` | 显示所有已导出的变量 |
| `名称=值` | 定义并导出变量 |

**示例**：

```bash
# 导出变量
export PATH="$HOME/bin:$PATH"
export MY_VAR="hello"

# 定义并导出
MY_VAR="hello"
export MY_VAR

# 导出函数
myfunc() { echo "hi"; }
export -f myfunc

# 取消导出
export -n MY_VAR

# 查看所有导出变量
export -p
```

## 最佳实践总结

| 场景 | 推荐做法 | 说明 |
|------|---------|------|
| 需要修改变量但不污染父 Shell | 使用 sub-shell `( ... )` | 变量修改自动隔离 |
| 需要在子 Shell 中访问变量 | `export` 导出 | child-shell 仅继承环境变量 |
| 管道中保留变量修改 | 使用进程替换 `< <(...)` 或 `lastpipe` | 避免 sub-shell 中的变量丢失 |
| 获取真实 PID | 使用 `$BASHPID` 而非 `$$` | `$$` 在 sub-shell 中不准确 |
| 加载配置文件 | 使用 `source` / `.` | 不创建子 Shell |
| 替换当前 Shell | 使用 `exec` | PID 不变，进程映像被替换 |
| 在 sub-shell 中使用 trap | 需要重新注册 | sub-shell 继承注册信息但不生效 |

# Bash 信号处理机制

> **本章概述**：信号是 Linux 进程间通信的核心机制之一，Bash 通过内置命令 `trap` 和 `kill` 实现对信号的捕获与发送。掌握信号处理机制，是编写健壮 Shell 脚本（如守护进程、临时文件清理、优雅退出）的关键前提。

## Linux 信号基础

### 信号的概念

信号（Signal）是操作系统中用于通知进程某个事件已经发生的异步机制。信号的产生来源包括：

- **硬件异常**：如除零、非法内存访问
- **终端操作**：如用户按下 `Ctrl+C`、`Ctrl+Z`
- **软件触发**：如通过 `kill` 命令发送信号
- **内核事件**：如子进程退出时父进程收到 `SIGCHLD`

进程接收到信号时，有以下三种处理方式：

| 处理方式 | 说明 | 对应操作 |
|---------|------|---------|
| **捕获并自定义处理** | 为信号注册自定义回调函数，进程收到信号时执行该函数 | `signal(SIGINT, handler)` |
| **忽略信号** | 内核直接丢弃该信号，目标进程不会收到 | `signal(SIGINT, SIG_IGN)` |
| **执行默认操作** | 按内核预定义的默认方式处理（通常为终止进程） | `signal(SIGINT, SIG_DFL)` |

```c
// C 语言中的信号处理示例
signal(SIGINT, SIG_IGN);    // 忽略 SIGINT 信号
signal(SIGTERM, SIG_DFL);   // 恢复 SIGTERM 的默认处理
signal(SIGSTOP, m_handler); // 自定义 SIGSTOP 的处理函数
```

> **重要提示**：`SIGKILL`（信号9）和 `SIGSTOP`（信号19）**不能被捕获、阻塞或忽略**，它们始终执行默认操作。这是内核保证的手段，用于在进程失控时强制终止或暂停。

### 信号的生命周期

信号从产生到处理经历以下阶段：

```
产生(generate) → 未决(pending) → 递送(delivery) → 处理(handle)
```

- **产生**：内核或进程发送信号
- **未决**：信号已产生但尚未递送给目标进程（可能因为被阻塞或进程尚未调度）
- **递送**：内核将信号传递给目标进程
- **处理**：进程执行信号的默认操作、自定义处理函数或忽略

信号的递送状态记录在目标进程 `task_struct` 的 `signal` 变量（类型为 `sigset_t`，又称**信号位图**）中，每一位对应一个信号的处理状态。

### 可靠信号与不可靠信号

在 Linux 中，信号按照处理策略分为两类：

| 分类 | 信号编号 | 处理策略 | POSIX 名称 | 特点 |
|------|---------|---------|-----------|------|
| **不可靠信号** | 1–31 | 丢弃策略：如果信号处于未决或忽略状态，后续同类信号直接丢弃 | 非实时信号 | 可能丢失信号 |
| **可靠信号** | 32–64 | 排队策略：未决的信号被放入队列，只要不超过内核上限就不会丢失 | 实时信号 | 不会丢失（在队列未满时） |

**不可靠信号的丢失场景**：

```
发送 SIGINT → 进程正在处理 SIGINT（未决） → 又发送 SIGINT → 被丢弃
```

**可靠信号的排队机制**：内核在 `task_struct` 中维护信号队列，新产生的信号入队等待处理。

> **提示**：可通过 `man 7 signal` 查看系统支持的完整信号列表，通过 `kill -l` 列出所有信号名称及编号。

### 常用信号一览

以下是 Bash 脚本编程中最常用的信号：

| 信号 | 编号 | 默认操作 | 触发方式 | 说明 |
|------|------|---------|---------|------|
| `SIGHUP` | 1 | 终止进程 | 终端关闭 / 守护进程重读配置 | 用户终端连接结束时发出，通知同一 session 内的作业终止；对守护进程则用于通知其重读配置文件 |
| `SIGINT` | 2 | 终止进程 | `Ctrl+C` | 键盘中断信号，终止前台运行进程 |
| `SIGKILL` | 9 | 终止进程 | `kill -9 <pid>` | 不可捕获、不可忽略、不可阻塞的强制终止信号 |
| `SIGTERM` | 15 | 终止进程 | `kill <pid>`（默认） | 程序终止信号，可被捕获和处理，通常用于要求程序正常退出 |
| `SIGSTOP` | 19 | 停止进程 | 编程方式发送 | 不可捕获、不可忽略的暂停信号 |
| `SIGTSTP` | 20 | 停止进程 | `Ctrl+Z` | 终端停止信号，将当前任务挂起并交还 shell 控制权 |
| `SIGALRM` | 14 | 终止进程 | 定时器到期 | 由 `alarm()` 系统调用设置的定时器到期时产生 |

> **SIGHUP 详解**：登录 Linux 时，系统为登录用户分配一个终端（Session），该终端运行的所有程序都属于这个 Session。当用户退出登录时，前台进程组和有终端输出的后台进程会收到 `SIGHUP` 信号而终止。这也是守护进程利用 `SIGHUP` 重读配置的原理。

## 发送信号：kill 命令族

### 内置命令 kill

Bash 内置的 `kill` 命令用于向进程或作业发送信号。

**语法**：

```bash
kill [-s sigspec | -n signum | -sigspec] [pid | jobspec] ...
kill -l [exit_status]
```

**参数说明**：

| 参数 | 说明 |
|------|------|
| `-s sigspec` | 指定信号名称，如 `-s HUP`、`-s SIGTERM` |
| `-n signum` | 指定信号编号，如 `-n 9` |
| `-sigspec` | 简写形式，如 `-9`、`-HUP`、`-SIGTERM` |
| `pid` | 目标进程号（正数指定进程，0 表示当前进程组，-1 表示所有进程，负数表示进程组） |
| `jobspec` | 作业号，如 `%1`、`%ping` |
| `-l` | 列出所有信号名称；若指定 `exit_status`，则显示对应的信号名称 |
| `-l exit_status` | 将退出状态码转换为信号名称 |

**常用示例**：

```bash
# 发送默认信号 SIGTERM
kill 1234
kill -15 1234
kill -s TERM 1234
kill -s SIGTERM 1234

# 强制终止进程
kill -9 1234
kill -s KILL 1234

# 向进程组发送信号（负数 PGID）
kill -TERM -1234

# 向当前 shell 的所有子进程发送信号
kill -TERM 0

# 查看信号列表
kill -l
# 输出: 1) SIGHUP  2) SIGINT  3) SIGQUIT  4) SIGILL  ...

# 根据编号查看信号名称
kill -l 9
# 输出: KILL
```

**注意事项**：
- 普通用户只能向自己的进程发送信号
- `kill -9` 是最后手段，进程无法执行任何清理操作即被终止
- Bash 内置 `kill` 与外部命令 `/bin/kill` 功能略有差异，内置版本支持作业号 `%n`

### 外部命令 killall

`killall` 按进程名发送信号，可一次终止所有同名进程。

**语法**：

```bash
killall [选项] [-信号] 进程名 ...
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| `-信号` | 指定信号，如 `-9`、`-HUP` |
| `-e, --exact` | 精确匹配长进程名（超过15字符时有用） |
| `-I, --ignore-case` | 进程名匹配忽略大小写 |
| `-i, --interactive` | 交互式确认后再终止 |
| `-r, --regexp` | 将进程名解释为正则表达式 |
| `-u, --user 用户` | 仅终止指定用户拥有的进程 |
| `-w, --wait` | 等待所有被终止的进程退出后才返回 |
| `-v, --verbose` | 显示详细输出 |
| `-q, --quiet` | 静默模式，无进程被终止时不输出 |

**常用示例**：

```bash
# 终止所有 chrome 进程（发送 SIGTERM）
killall chrome

# 强制终止所有 firefox 进程
killall -9 firefox

# 交互式终止
killall -i nginx

# 仅终止指定用户的进程
killall -u www-data nginx

# 让 nginx 重读配置（发送 SIGHUP）
killall -HUP nginx

# 等待进程完全退出
killall -w chrome
```

**注意事项**：
- `killall` 默认发送 `SIGTERM` 信号
- 进程名匹配默认为子串匹配，使用 `-e` 可精确匹配
- 不是所有系统都安装了 `killall`（如 Alpine Linux 需额外安装）

### 外部命令 pkill

`pkill` 与 `killall` 类似，但支持更灵活的匹配模式。

**语法**：

```bash
pkill [选项] 模式
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| `-信号` | 指定信号，如 `-9`、`-HUP` |
| `-f` | 匹配完整命令行（默认只匹配进程名） |
| `-u, --uid 用户` | 仅匹配指定用户的进程 |
| `-t, --terminal 终端` | 仅匹配指定终端的进程 |
| `-P, --parent PPID` | 仅匹配指定父进程的子进程 |
| `-n` | 仅匹配最新启动的进程 |
| `-o` | 仅匹配最早启动的进程 |
| `-c, --count` | 显示匹配的进程数 |
| `-l` | 同时列出进程名和 PID |
| `-x` | 精确匹配进程名 |

**常用示例**：

```bash
# 终止所有名为 chrome 的进程
pkill chrome

# 匹配完整命令行（如包含参数的命令）
pkill -f "python3 server.py"

# 强制终止指定用户的进程
pkill -9 -u guest

# 仅终止最新启动的 sshd 进程
pkill -n sshd

# 查看匹配数量
pkill -c nginx

# 列出匹配的进程名和 PID
pkill -l chrome
```

### 外部命令 pgrep

`pgrep` 是 `pkill` 的配套命令，用于查找进程，不发送信号。

**语法**：

```bash
pgrep [选项] 模式
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| `-f` | 匹配完整命令行 |
| `-u, --uid 用户` | 仅匹配指定用户的进程 |
| `-t, --terminal 终端` | 仅匹配指定终端的进程 |
| `-P, --parent PPID` | 仅匹配指定父进程的子进程 |
| `-n` | 仅匹配最新启动的进程 |
| `-o` | 仅匹配最早启动的进程 |
| `-l` | 同时列出进程名 |
| `-a` | 列出完整命令行 |
| `-c` | 显示匹配数量 |
| `-x` | 精确匹配进程名 |

**常用示例**：

```bash
# 查找 chrome 进程的 PID
pgrep chrome

# 查找并显示进程名
pgrep -l chrome

# 查找并显示完整命令行
pgrep -a chrome

# 配合 kill 使用
kill $(pgrep chrome)
```

### 外部命令 pidof

`pidof` 用于查找指定程序名的进程 PID。

**语法**：

```bash
pidof [选项] 程序名
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| `-s` | 仅返回一个 PID |
| `-c` | 仅返回具有相同根目录的进程（仅 root 可用） |
| `-x` | 同时返回运行该脚本的 shell 的 PID |
| `-o 省略PID` | 忽略指定 PID 的进程 |

**常用示例**：

```bash
# 查找 nginx 的所有 PID
pidof nginx
# 输出: 1234 5678

# 配合 kill 使用
kill $(pidof nginx)

# 忽略当前 shell 进程
pidof -o $$ bash
```

### 命令对比

| 特性 | `kill` | `killall` | `pkill` | `pgrep` | `pidof` |
|------|--------|-----------|---------|---------|---------|
| 目标指定方式 | PID / 作业号 | 进程名 | 模式匹配 | 模式匹配 | 程序名 |
| 发送信号 | ✅ | ✅ | ✅ | ❌（仅查找） | ❌（仅查找） |
| 支持作业号 | ✅（内置版） | ❌ | ❌ | ❌ | ❌ |
| 正则匹配 | ❌ | ✅（`-r`） | ✅ | ✅ | ❌ |
| 匹配完整命令行 | ❌ | ❌ | ✅（`-f`） | ✅（`-f`） | ❌ |
| 精确匹配 | 按PID精确 | ✅（`-e`） | ✅（`-x`） | ✅（`-x`） | ✅ |

**终止 chrome 进程的多种写法**：

```bash
kill $(pgrep chrome)                     # pgrep + kill
kill $(pidof chrome)                     # pidof + kill
kill $(ps -ef | grep chrome | awk '{print $2}')  # ps + grep + awk + kill
killall chrome                           # killall 直接按名终止
pkill chrome                             # pkill 直接按名终止
```

## 捕获信号：trap 命令

### trap 语法

`trap` 是 Bash 内置命令，用于捕获信号并执行自定义处理逻辑。

**语法**：

```bash
trap [命令] 信号 ...
trap -l
trap -p [信号 ...]
```

**参数说明**：

| 参数 | 说明 |
|------|------|
| `命令` | 捕获信号后执行的命令（用引号包裹，可以是函数调用） |
| `信号` | 信号名称或编号，多个信号以空格分隔；信号名大小写不敏感，`SIG` 前缀可省略 |
| `-l` | 列出系统支持的所有信号（等价于 `kill -l`） |
| `-p` | 显示当前已设置的 trap 处理命令 |

**信号名称的等价写法**：

```bash
# 以下写法完全等价
trap "echo 123" SIGINT
trap "echo 123" INT
trap "echo 123" 2
trap "echo 123" int
trap "echo 123" Int
```

### 典型用法

#### 临时文件清理

在脚本退出时自动清除临时文件，是最常见的 `trap` 用途：

```bash
#!/bin/bash

# 创建临时文件
tempfile=$(mktemp) || exit 1

# 注册退出时清理
trap 'rm -f "$tempfile"' EXIT

# 脚本正常逻辑...
echo "Working with $tempfile"
# 无论脚本正常退出还是被信号终止，都会执行清理
```

> **最佳实践**：使用 `EXIT` 伪信号（而非单独捕获 `SIGINT`、`SIGTERM` 等），可以覆盖所有退出场景（正常退出、信号终止、`exit` 命令等）。

#### 守护进程重读配置

在守护进程中捕获 `SIGHUP`，实现配置热重载：

```bash
#!/usr/bin/env bash

if [ ! -r "$1" ]; then
    echo "Usage: $0 <cfg-file-path>"
    exit 1
fi

echo "PID: $$"

CONFIG=$1

read_config () {
    echo "reading cfg from $CONFIG"
    source "$CONFIG"
}

read_config
trap "read_config" HUP

while :; do
    echo "$var"
    sleep 15
done
```

测试方法：

```bash
# 终端一：启动守护进程
bash daemon.sh ./config/cfg1
# PID: 8807
# reading cfg from ./config/cfg1
# from cfg1

# 终端二：修改配置并发送 SIGHUP
cat > cfg1 << EOF
var="after change"
EOF
kill -s HUP 8807

# 终端一输出：
# reading cfg from ./config/cfg1
# after change
```

#### 捕获多个信号

`trap` 可以同时捕获多个信号：

```bash
#!/usr/bin/env bash

trap "echo Boom!" SIGINT SIGTERM

echo "PPID=$PPID PID=$$"
while :; do
    sleep 60
done
```

#### 保存与重设信号处理

```bash
# 设置自定义处理
trap "printf BOOM" INT

# 保存当前处理方式
traps=$(trap)

# 重设为默认处理
trap INT

# 重新加载之前保存的处理方式
eval $traps
```

### 特殊伪信号

`trap` 除了捕获标准信号外，还支持以下特殊伪信号：

| 伪信号 | 说明 | 触发时机 |
|--------|------|---------|
| `EXIT`（编号 0） | 进程退出时触发 | 脚本执行完毕、`exit` 命令、收到终止信号等所有退出场景 |
| `ERR` | 命令执行出错时触发 | 命令返回非零退出状态时（需注意 `set -e` 的影响） |
| `DEBUG` | 每条命令执行前触发 | 在每条命令执行之前执行预定义代码块，可用于调试和跟踪 |

**EXIT 伪信号详解**：

```bash
trap "code" EXIT
# 当 shell 退出时（包括 Ctrl+D 退出交互式 shell），会执行 code
```

> **注意**：在标准 Shell（sh）中，`trap ... 0` 仅捕捉 `exit` 命令；在 Bash 中，`EXIT` 覆盖所有退出情况。

**ERR 伪信号应用**：

```bash
#!/bin/bash

trap 'echo "Error at line $LINENO: command returned $?"' ERR

# 任何命令失败都会触发 ERR trap
cp /nonexistent /tmp/backup
# 输出: Error at line 7: command returned 1
```

**DEBUG 伪信号应用**：

```bash
#!/bin/bash

trap 'echo "Executing: $BASH_COMMAND"' DEBUG

echo "Hello"
# 输出: Executing: echo "Hello"
#       Hello

ls /tmp
# 输出: Executing: ls /tmp
#       (ls 的输出...)
```

### trap 查看命令

```bash
# 列出所有信号
trap -l

# 查看已设置的 trap
trap -p
# 输出示例：
# trap -- 'code' EXIT
# trap -- 'echo SIGINT' SIGINT

# 查看特定信号的 trap 设置
trap -p SIGINT
```

## 信号处理的特殊情况

### Bash 执行外部命令时的信号延迟

Bash 在执行外部命令时，会将前台任务的信号处理优先级提高。当前台任务执行完毕或被终止后，Bash 才会处理自身收到的信号。

```bash
# 终端一
sleep 100  # 外部命令，Bash 等待其完成
# 100秒后打印空行并显示提示符

# 终端二
pstree -ap 16003
# bash,16003
#   `-sleep,17249 100

kill -s INT 16003  # 向 Bash 发送 SIGINT
# Bash 不会立即响应，等 sleep 执行完后才处理
```

**关键区别**：使用 `Ctrl+C` 会对**整个前台进程组**发送 `SIGINT`，因此 `sleep` 会立即结束：

```bash
# 终端一
sleep 100
^C  # Ctrl+C → 整个进程组收到 SIGINT → sleep 立即结束 → Bash 也收到信号
```

### WCE 原则（Wait and Cooperative Exit）

Bash 接收到退出信号时遵循 **WCE** 原则处理。当按下 `Ctrl+C` 时，当前进程组中所有进程都会收到 `SIGINT`，具体情况如下：

| 子进程对 SIGINT 的处理 | Bash 的行为 | 结果 |
|----------------------|------------|------|
| 处理信号后自己 kill（以信号退出） | Bash 检测到子进程因信号非正常退出 | **立即退出**当前脚本 |
| 处理信号后用 `exit` 正常退出 | Bash 认为子进程正常执行完毕 | **继续执行**下一条命令 |
| 不处理 SIGINT（默认处理） | 同"处理信号后自己 kill" | **立即退出**当前脚本 |

**示例：ping 循环（需要多次 Ctrl+C 才能退出）**

```bash
for i in $(seq 254); do
    ping -c 2 "192.168.1.$i"
done
# ping 会处理 SIGINT 并以 exit(0) 正常退出
# Bash 认为上条命令正常完成，继续执行下一次循环
# 因此需要按 254 次 Ctrl+C 才能完全退出
```

**示例：sleep 循环（一次 Ctrl+C 即可退出）**

```bash
i=1
while [ "$i" -le 100 ]; do
    printf "%d " "$i"
    i=$((i+1))
    sleep 10
done
# sleep 以默认方式处理 SIGINT（被信号终止）
# Bash 检测到子进程因信号非正常退出，立即退出脚本
```

**推荐的安全 SIGINT 捕获方式**：

```bash
trap 'rm -f "$tempfile"; trap - INT; kill -s INT "$$"' INT
```

这个写法的含义：
1. 执行清理操作 `rm -f "$tempfile"`
2. 恢复 SIGINT 的默认处理 `trap - INT`
3. 向自己发送 SIGINT `kill -s INT "$$"`，使 Bash 以信号非正常退出，确保父 shell 也能正确退出

### 子 Shell 中的 trap 继承问题

`trap` 设置只作用于当前进程。通过 `fork` 创建的子 Shell（`child-shell`）不会继承当前 Shell 设置的 `trap`：

```bash
# 设置 trap
trap "printf book" 2

# 当前 shell 中生效
^C  # 输出: book

# child-shell 中无效
bash -c "trap -p"  # 无输出

bash
^C  # 无效，执行默认处理
```

**原因**：`child-shell` 通过 `fork-exec` 创建，新的 Shell 实例不会继承父 Shell 的 `trap` 注册信息。

**sub-shell 的特殊情况**：`sub-shell`（通过 `()`、管道等创建）可以看到父 Shell 的 `trap` 注册信息（因为 `fork` 拷贝了数据），但不会生效——因为 `trap` 的工作原理是将回调函数注册到当前进程，`sub-shell` 只继承了注册信息却没有进行回调函数注册：

```bash
trap "echo done" SIGTERM

# sub-shell 中可以看到注册信息但不生效
(trap -p; kill $BASHPID; echo "Never print")
# 输出: trap -- 'echo done' SIGTERM
#        Terminated  ← 没有执行自定义 trap

# 需要重新注册才能生效
(trap "echo done" SIGTERM; trap -p; kill $BASHPID; echo "Never print")
# 输出: trap -- 'echo done' SIGTERM
#        done  ← 重新注册后生效
#        Never print
```

### 函数中的 trap

在函数中设置的 `trap` 是**全局生效**的。重复设置同一个信号，只有最后一次 `trap` 有效：

```bash
set_trap() {
    trap "echo from function" INT
}

trap "echo from global" INT
set_trap

^C  # 输出: from function（函数中的设置覆盖了全局设置）
```

### 后台任务与孤儿进程

当 Bash 退出时仍有未完成的后台任务，这些任务会变为**孤儿进程**，其父进程变为 PID=1 的 `init` 进程：

```bash
(sleep 50 & sleep 50 & wait)

# 进程树
bash,15158
  `-bash,7629
      |-sleep,7630 50
      `-sleep,7631 50

kill 7629

# 进程树
bash,15158

ps -ef | grep "sleep 50"
# PPID 已变为 1
```

如果需要在进程退出时将后台任务也终止，需要记录后台任务的 PID 并在退出前 kill：

```bash
#!/usr/bin/env bash

BPIDARRAY=()

for i in {0..9}; do
    sleep 20 &
    BPIDARRAY[$i]=$!
done

sleep 3
trap "kill ${BPIDARRAY[@]}" EXIT
wait
```

## 相关系统命令参考

### pstree

`pstree` 以树状图显示进程间的关系。

**语法**：

```bash
pstree [选项] [PID | 用户]
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| `-a` | 显示完整命令行（包含参数） |
| `-p` | 显示 PID |
| `-g` | 显示 PGID（进程组 ID） |
| `-n` | 按 PID 排序（默认按名称排序） |
| `-h` | 高亮当前进程及其祖先 |
| `-H PID` | 高亮指定进程及其祖先 |
| `-s` | 显示父进程 |
| `-S` | 显示命名空间切换 |
| `-l` | 不截断长行 |

**示例**：

```bash
# 显示当前 shell 的进程树（含 PID 和参数）
pstree -ap $$

# 显示指定进程的进程树
pstree -ap 16003

# 显示完整进程树
pstree -ap
```

### ps

`ps` 报告当前系统的进程快照。

**语法**：

```bash
ps [选项]
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| `-e` | 显示所有进程 |
| `-f` | 全格式显示（含 UID, PID, PPID, C, STIME, TTY, TIME, CMD） |
| `-j` | 作业格式显示（含 PGID, SID） |
| `-l` | 长格式显示 |
| `-H` | 树状层级显示 |
| `-T` | 显示线程 |
| `-u 用户` | 显示指定用户的进程 |
| `-p PID` | 显示指定 PID 的进程 |
| `T` | 仅显示当前终端的进程 |

**示例**：

```bash
# 查看当前终端关联的所有进程
ps T

# 全格式查看所有进程
ps -ef

# 查看进程树
ps -ejfH

# 查看指定进程
ps -fp 1234

# 查看进程状态
ps -eo pid,ppid,pgid,sid,stat,cmd
```

**进程状态（STAT 列）**：

| 状态码 | 含义 |
|--------|------|
| `R` | 运行中（Running） |
| `S` | 可中断睡眠（Interruptible Sleep） |
| `T` | 被作业控制信号停止（Stopped） |
| `Z` | 僵尸进程（Zombie） |
| `D` | 不可中断睡眠（Uninterruptible Sleep） |
| `s` | 会话领导进程（Session Leader） |
| `+` | 前台进程组 |
| `<` | 高优先级进程 |

### mktemp

`mktemp` 安全地创建临时文件或目录。

**语法**：

```bash
mktemp [选项] [模板]
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| `-d` | 创建临时目录而非文件 |
| `-u` | 不安全模式：仅生成名称，不实际创建（不推荐） |
| `-t 前缀` | 使用指定前缀（在 `$TMPDIR` 下创建） |
| `-p 目录` | 在指定目录下创建 |
| `--suffix 后缀` | 为模板添加后缀 |

**模板语法**：模板中必须包含至少 3 个连续的 `X`，`mktemp` 会将其替换为随机字符。

**示例**：

```bash
# 创建临时文件
tempfile=$(mktemp)
echo $tempfile
# /tmp/tmp.AbCdEf1234

# 创建带前缀的临时文件
tempfile=$(mktemp -t myapp.XXXXXX)

# 创建临时目录
tempdir=$(mktemp -d)

# 在指定目录下创建
tempfile=$(mktemp -p /var/tmp myapp.XXXXXX)

# 典型用法：配合 trap 清理
tempfile=$(mktemp) || exit 1
trap 'rm -f "$tempfile"' EXIT
```

> **安全提示**：始终使用 `mktemp` 而非自行拼接临时文件路径，避免符号链接攻击和文件名冲突。

## 最佳实践总结

| 场景 | 推荐做法 | 示例 |
|------|---------|------|
| 临时文件清理 | 使用 `trap ... EXIT` | `trap 'rm -f "$tmp"' EXIT` |
| 守护进程重载 | 捕获 `SIGHUP` | `trap "reload_config" HUP` |
| 优雅退出 | 先清理再重设信号再自杀 | `trap 'cleanup; trap - INT; kill -INT $$' INT` |
| 终止同名进程 | 优先使用 `killall` 或 `pkill` | `killall chrome` |
| 避免信号丢失 | 关键信号使用可靠信号（编号 ≥ 32） | `kill -42 $pid` |
| 信号调试 | 使用 `trap -p` 查看当前设置 | `trap -p` |
| 防止僵尸进程 | 退出时主动 kill 后台子进程 | `trap "kill ${PIDS[@]}" EXIT` |

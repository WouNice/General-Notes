# Bash 作业控制机制

> **本章概述**：作业控制（Job Control）是 Shell 对前后台进程进行管理的机制。通过作业控制，用户可以在前台与后台之间切换任务、挂起/恢复进程、以及处理终端退出时的信号。掌握作业控制是高效使用终端和编写可靠脚本的基础。

## 作业控制基础

### 作业与进程的区别

| 概念 | 说明 | 生命周期 |
|------|------|---------|
| **进程（Process）** | 操作系统资源分配的基本单位，拥有独立的 PID、地址空间 | 从 `fork()` 创建到 `exit()` 退出 |
| **进程组（Process Group）** | 一个或多个进程的集合，共享同一个 PGID，用于信号分发 | 从组长创建到组内最后一个进程退出 |
| **作业（Job）** | Shell 层面的调度单位，维护命令执行的上下文 | 作业仅追踪**初始进程**，初始进程全部退出则作业结束 |

**关键区别**：作业只维护其初始进程，一旦所有初始进程退出则代表作业执行完毕，未结束的子进程不会被作业追踪。

```bash
#!/usr/bin/env bash

(sleep 20 & sleep 2) &
jobs -l
# [1]+ 19318 Running  ( sleep 20 & sleep 2 ) &
# 作业初始进程为 19318

sleep 2
jobs -l
# [1]+ 19318 Done  ( sleep 20 & sleep 2 )
# sleep 2 退出 → 初始进程 19318 退出 → 作业结束

# 但 sleep 20 仍在运行（变为孤儿进程，PPID=1）
```

### 作业的类型

一个 Shell 进程在同一时刻只能有**一个前台作业**，但可以有**多个后台作业**。作业可以包含不同数量的进程：

**作业中包含单个进程**：

```bash
sleep 50 &
[1] 20536  # 作业号为1，进程号为20536

jobs -l
[1]+ 20536 Running  sleep 50 &
```

**作业中包含进程及其子进程**：

```bash
(sleep 20 & sleep 10 & wait) &
[1] 21111

pstree -ap $$
# bash,21023
#   ├─bash,21111      ← 作业的初始进程
#   │   ├─sleep,21112 20
#   │   └─sleep,21113 10
```

**作业中包含多个进程（管道）**：

```bash
sleep 10 | sleep 8 &
[1] 20689

jobs -l
# [1]+ 20688 Running  sleep 10
# | sleep 8 &
# 管道两侧分别在 sub-shell 中运行，作业包含两个初始进程
```

## 前台作业管理

### 前台作业的特点

直接在 Shell 会话中输入命令执行的作业会在前台运行。前台作业具有以下特点：

- **阻塞 Shell**：前台作业运行期间，Shell 提示符不会出现
- **占用终端 I/O**：前台作业独占标准输入、标准输出和标准错误
- **优先处理信号**：前台进程组优先接收终端信号

### 挂起前台作业

要在不退出前台作业的情况下拿回终端控制权，可以使用 `Ctrl+Z` 发送 `SIGTSTP` 信号：

```bash
ping 127.0.0.1
^Z  # 按下 Ctrl+Z

[1]+  Stopped  ping 127.0.0.1
# ping 被暂停，Shell 控制权归还
```

通过 `ps T` 查看当前终端关联的进程：

```bash
ps T
  PID TTY      STAT   TIME COMMAND
 4731 pts/1    Ss     0:00 /bin/bash
 4738 pts/1    T      0:00 ping 127.0.0.1  # T 状态 = 被停止
 4891 pts/1    R+     0:00 ps T
```

> **进程状态 `T`**：表示进程被作业控制信号停止（Stopped），并非终止。可通过 `fg` 恢复到前台或 `bg` 在后台继续运行。

### suspend 命令

内置命令 `suspend` 可以挂起当前 Shell 本身（非作业）。

**语法**：

```bash
suspend [-f]
```

| 参数 | 说明 |
|------|------|
| `-f` | 强制挂起，即使是登录 Shell（默认登录 Shell 不允许挂起） |

**示例**：

```bash
# 在子 Shell 中挂起
bash
suspend
# 此时该子 Shell 被挂起，回到父 Shell

# 强制挂起登录 Shell
suspend -f
```

## 后台作业管理

### 创建后台作业

在命令末尾添加 `&` 符号即可将命令放入后台运行：

```bash
command &
```

后台作业不会阻塞 Shell，用户可以继续输入其他命令。但后台作业在需要读取标准输入时会被自动挂起。

### jobs 命令

`jobs` 是 Bash 内置命令，用于查看当前 Shell 中维护的作业列表。

**语法**：

```bash
jobs [选项] [作业号 ...]
jobs -x 命令 [参数 ...]
```

**参数说明**：

| 参数 | 说明 |
|------|------|
| `-l` | 额外显示进程号（PID） |
| `-n` | 仅显示状态发生变化的作业（自上次通知后） |
| `-p` | 仅显示进程号 |
| `-r` | 仅显示运行中的作业 |
| `-s` | 仅显示已停止的作业 |
| `-x 命令` | 将作业号替换为对应的进程号后执行命令 |

**作业状态类型**：

| 状态 | 含义 |
|------|------|
| `Running` | 正在运行 |
| `Stopped` | 被挂起（如 `Ctrl+Z`） |
| `Done` | 执行完毕退出（仅显示一次） |
| `Terminated` | 被信号终止 |
| `Hangup` | 收到 `SIGHUP` 后退出 |

**示例**：

```bash
# 列出所有作业（含 PID）
jobs -l
[1]+  6528 Stopped  ping 127.0.0.1
[2]   7101 Running  xeyes &
[3]-  7104 Running  xload &

# 仅列出运行中的作业
jobs -r

# 仅列出已停止的作业
jobs -s

# 查看特定作业
jobs %1
```

### 作业标识符

在作业控制命令中，可以通过以下方式引用作业：

| 标识符 | 含义 | 示例 |
|--------|------|------|
| `%n` | 作业号为 n 的作业 | `kill %1` |
| `%s` | 名称以字符串 s 开头的作业 | `kill %xe` |
| `%?s` | 名称包含字符串 s 的作业 | `fg %?eye` |
| `%%` 或 `%+` | 当前作业（最近的被调往前台的作业） | `fg %%` |
| `%-` | 当前作业的上一个作业 | `bg %-` |

**`+` 与 `-` 的含义**：在 `jobs -l` 输出中，`+` 标记当前作业，`-` 标记当前作业的前一个作业。

```bash
jobs -l
[1]-  Stopped  ping 127.0.0.1   # - 表示前一个作业
[2]+  Stopped  xeyes             # + 表示当前作业
[3]   Running  xload &
```

### fg 命令

`fg` 将后台作业提到前台运行。

**语法**：

```bash
fg [作业号]
```

**示例**：

```bash
# 将作业 2 提到前台
fg %2
fg 2     # fg 只对作业有效，可省略 %

# 将当前作业提到前台
fg
fg %+    # 等价
fg %%    # 等价

# 将名称以 xeye 开头的作业提到前台
fg %xeye
```

### bg 命令

`bg` 让已停止的作业在后台继续运行。

**语法**：

```bash
bg [作业号]
```

**示例**：

```bash
# 先挂起前台作业
sleep 100
^Z
[1]+  Stopped  sleep 100

# 在后台恢复运行
bg %1
[1]+ sleep 100 &

jobs
[1]+  Running  sleep 100 &
```

> **注意**：如果后台作业执行到需要读取标准输入的代码段，会被自动挂起（停止），此时 `bg` 命令无法使其恢复运行。

```bash
while true; do read line; done &
[1] 20780
# 被输入挂起，仍处于停止状态
[1]+  Stopped  while true; do read line; done
# bg 无法使其恢复，因为仍需要输入
bg %1
[1]+ while true; do read line; done &
[1]+  Stopped  while true; do read line; done
```

### 作业控制命令与符号汇总

| 符号 / 命令 | 描述 | 示例 |
|-------------|------|------|
| `&` | 将作业放到后台 | `command &` |
| `%n` | 作业号为 n 的作业 | `kill %1` |
| `%s` | 名称以 s 开头的作业 | `kill %xe` |
| `%?s` | 名称包含 s 的作业 | `jobs %?ping` |
| `%%` / `%+` | 当前作业 | `kill %%` |
| `%-` | 当前作业的上一个作业 | `bg %-` |
| `Ctrl+Z` | 挂起前台作业（发送 SIGTSTP） | — |
| `Ctrl+C` | 终止前台作业（发送 SIGINT） | — |
| `jobs -l` | 列出所有作业（含 PID） | `jobs -l` |
| `jobs -r` | 列出运行中的作业 | `jobs -r` |
| `jobs -s` | 列出已停止的作业 | `jobs -s` |
| `fg` | 将作业提到前台 | `fg %2` |
| `bg` | 让作业在后台运行 | `bg %%` |
| `kill %n` | 向作业发送信号 | `kill -9 %1` |
| `wait` | 等待后台作业完成 | `wait %1` |

## 处理 SIGHUP 信号

### SIGHUP 的传播

当终端退出时，Shell 会向所有作业发送 `SIGHUP` 信号进行清理：

```
终端关闭 → Shell 收到 SIGHUP → Shell 向所有作业发送 SIGHUP → 作业终止
```

如果 Shell 退出时存在停止的后台作业，会提示 `There are stopped jobs.`。如果忽略提示继续退出，所有作业都会收到 `SIGHUP`。

以下方法可以让后台进程在终端退出后继续运行。

### nohup 命令

`nohup` 使启动的作业忽略 `SIGHUP` 信号。

**语法**：

```bash
nohup 命令 [参数 ...] [&]
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| （无特殊参数） | `nohup` 本身不接受命令行选项 |

**行为特点**：

| 特点 | 说明 |
|------|------|
| 忽略 SIGHUP | 进程不再受终端退出的影响 |
| 忽略标准输入 | 防止后台作业阻塞 |
| 输出重定向 | 默认重定向到 `~/nohup.out`；手动指定则自动将 stderr 重定向到 stdout |
| 进程关系 | 终端退出后变为孤儿进程（PPID=1） |

**常用示例**：

```bash
# 基本用法
nohup ping 127.0.0.1 &
# 输出: nohup: ignoring input and appending output to 'nohup.out'

# 手动指定输出文件
nohup ping 127.0.0.1 > outfile &
# 输出: ignoring input and redirecting stderr to stdout

# 合并 stdout 和 stderr
nohup ping 127.0.0.1 &> outfile &
```

**验证**：退出当前终端后在另一个终端查找进程：

```bash
ps -ef | awk '$8~/^ping/ {print "PID:"$2", PPID:"$3}'
# PID:21222, PPID:1  ← 变为孤儿进程继续运行
```

**注意事项**：

1. `nohup` 后不可以通过 `(...)` 的方式在 sub-shell 中执行命令，会导致语法错误：

```bash
# 错误写法
nohup (sleep 120; echo "job done") &

# 正确写法：使用 bash -c
nohup bash -c 'sleep 120; echo "job done"' &
```

2. `nohup` 必须在程序运行**前**指定，无法对已运行的作业使用。

### disown 命令

`disown` 是 Bash 内置命令，用于从作业列表中移除作业或让作业忽略 `SIGHUP`。与 `nohup` 不同，`disown` 可以对**已运行**的作业进行操作。

**语法**：

```bash
disown [选项] [作业号 ...]
```

**参数说明**：

| 参数 | 说明 |
|------|------|
| `-h` | 让作业忽略退出时的 `SIGHUP`，但不从作业列表中删除（可继续使用作业控制命令） |
| `-r` | 从作业列表中移除所有运行中的作业 |
| `-a` | 从作业列表中移除所有作业 |
| （无选项） | 从作业列表中移除指定作业（移除后无法再进行作业控制） |

**常用示例**：

```bash
# 将作业 1 从作业列表中剥离
ping 127.0.0.1 > /dev/null &
[1] 29748
disown %1
jobs  # 无输出

# 让作业 2 忽略 SIGHUP 但保留在作业列表中
disown -h %2
# 退出终端后该作业仍继续运行，且可继续通过 jobs 查看

# 移除所有运行中的作业
disown -r

# 移除所有作业
disown -a
```

**disown 与 nohup 的对比**：

| 特性 | `nohup` | `disown` |
|------|---------|----------|
| 使用时机 | 必须在启动命令前 | 可对已运行的作业使用 |
| SIGHUP 处理 | 忽略 SIGHUP | `-h` 忽略；无选项则从列表移除 |
| 输出重定向 | 自动重定向到 `nohup.out` | 不自动重定向 |
| 作业控制 | 仍在作业列表中 | `-h` 保留；无选项则移除 |
| sub-shell 支持 | 不支持 `(...)` 语法 | 支持对已有作业操作 |

**运行时重定向输出的技巧**（使用 gdb）：

```bash
python3 logerr.py &
sudo gdb -p $(pgrep -n python3)

(gdb) p close(2)              # 关闭标准错误
$1 = 0
(gdb) p creat("/tmp/pyout", 0600)  # 创建文件并连接到标准错误
$2 = 2
(gdb) q                       # 选 yes

# 验证
sudo ls -l /proc/31586/fd/2
# l-wx------ → /tmp/pyout
```

### huponexit 选项

`huponexit` 是 Bash 的 `shopt` 选项，控制 Shell 退出时是否向作业发送 `SIGHUP`。

**语法**：

```bash
shopt huponexit          # 查看选项状态
shopt -s huponexit       # 开启选项
shopt -u huponexit       # 关闭选项
```

| 状态 | 行为 |
|------|------|
| **关闭**（默认） | Shell 退出时向所有作业发送 `SIGHUP` |
| **开启** | Shell 执行 `exit` 退出时**不**向作业发送 `SIGHUP` |

**信号传播流程**：

```
会话退出 → SIGHUP → Bash（huponexit 开启）→ 不向作业分发 SIGHUP
会话退出 → SIGHUP → Bash（huponexit 关闭）→ 向所有作业分发 SIGHUP
```

> **注意**：`huponexit` 只影响 Shell 执行 `exit` 退出时的行为。如果是终端异常关闭（如网络断开），`SIGHUP` 仍然会传递给所有作业。

### 其他方式

**利用孤儿进程**：让后台作业变为孤儿进程，从而忽略父 Shell 传来的 `SIGHUP`：

```bash
# sub-shell 没有 wait 自己的后台任务，因此提前退出
# 子进程变为孤儿进程继续运行
(ping 127.0.0.1 >/dev/null &)

ps -f $(pgrep ping)
# PPID 已经变为 1
```

**终端复用工具**：提供更高级的会话管理功能。

| 工具 | 特点 |
|------|------|
| `screen` | GNU 终端复用器，支持会话 detach/attach |
| `tmux` | 更现代的终端复用器，支持窗格分割、脚本化配置 |
| `dtach` | 轻量级 detach 工具，仅提供最小化功能 |

```bash
# screen 示例
screen -S mysession      # 创建命名会话
# 在 screen 中运行命令
ping 127.0.0.1
# Ctrl+A D 分离会话
screen -r mysession      # 重新连接

# tmux 示例
tmux new -s mysession    # 创建命名会话
# 在 tmux 中运行命令
ping 127.0.0.1
# Ctrl+B D 分离会话
tmux attach -t mysession # 重新连接
```

## 相关系统命令参考

### wait 命令

`wait` 是 Bash 内置命令，用于等待后台作业完成并获取其退出状态。

**语法**：

```bash
wait [选项] [作业号或PID ...]
```

**参数说明**：

| 参数 | 说明 |
|------|------|
| `-n` | 等待任意一个指定的作业完成 |
| `-f` | 在作业状态变更前持续等待（Bash 5.1+） |
| `-p 变量` | 将完成作业的作业号赋给指定变量（Bash 5.1+） |
| `作业号或PID` | 等待指定的作业或进程；省略则等待所有当前 Shell 的子进程 |

**示例**：

```bash
# 等待所有后台作业
wait

# 等待特定作业
wait %1
wait 12345

# 等待任意一个完成
wait -n

# 获取后台作业的退出状态
sleep 10 &
wait $!
echo "Exit status: $?"

# 等待多个后台作业
sleep 5 &
pid1=$!
sleep 10 &
pid2=$!
wait $pid1 $pid2
echo "Both done"
```

### coproc 命令

`coproc` 是 Bash 内置命令，用于创建协作进程（在 sub-shell 中后台运行，同时建立双向管道）。

**语法**：

```bash
coproc [名称] 命令 [参数 ...]
coproc [名称] { 命令列表; }
```

**示例**：

```bash
# 创建命名协作进程
coproc MYPROC { while read line; do echo "Got: $line"; done; }

# 向协作进程写入
echo "hello" >&${MYPROC[1]}

# 从协作进程读取
read reply <&${MYPROC[0]}
echo "$reply"  # Got: hello
```

> **注意**：`coproc` 自身的退出状态始终为 0，实际命令的退出状态可通过 `wait` 获取。

## 最佳实践总结

| 场景 | 推荐做法 | 命令 |
|------|---------|------|
| 临时执行耗时命令 | 放入后台 | `command &` |
| 需要拿回终端控制权 | 挂起前台作业 | `Ctrl+Z`，然后 `bg` |
| 保持后台进程在退出后运行 | `nohup` 或 `disown` | `nohup command &` / `disown -h %1` |
| 长期运行的远程任务 | 使用终端复用器 | `tmux` / `screen` |
| 等待后台作业完成 | 使用 `wait` | `wait $!` |
| 防止后台作业阻塞 | 确保不需要终端输入 | 重定向 stdin：`command < /dev/null &` |

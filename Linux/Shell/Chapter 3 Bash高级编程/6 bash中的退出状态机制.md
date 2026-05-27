# Bash 退出状态机制

> **本章概述**：退出状态（Exit Status）是进程向调用者传递执行结果的核心机制。在 Bash 中，每条命令执行完毕都会返回一个 0–255 的退出状态码，通过 `$?` 变量获取。理解退出状态的传递规则和各类命令的退出状态约定，是编写可靠脚本和正确处理错误的基础。

## 退出状态基础

### 进程退出状态

当进程调用 `exit()`、`_exit()` 或从 `main()` 返回时，内核通过 `WEXITSTATUS` 宏取其参数的低 8 位作为退出状态码：

```
exit(5)  → WEXITSTATUS(status) = 5
exit(256) → WEXITSTATUS(status) = 0  (256 的低 8 位为 0)
exit(-1)  → WEXITSTATUS(status) = 255 (-1 的补码低 8 位为 255)
```

**C 语言视角**：

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/wait.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();
    if (pid == 0) {
        // 子进程
        exit(5);
    } else {
        int status;
        waitpid(pid, &status, 0);
        if (WIFEXITED(status)) {
            printf("Exit status: %d\n", WEXITSTATUS(status));
            // Exit status: 5
        }
    }
    return 0;
}
```

### 信号退出状态

当进程被信号终止时（而非主动调用 `exit`），Shell 将退出状态设为 `128 + 信号编号`：

| 终止方式 | 退出状态计算 | 示例 |
|---------|------------|------|
| 被信号 N 终止 | `128 + N` | `SIGKILL`(9) → 137 |
| 主动 `exit(N)` | `N`（低 8 位） | `exit(1)` → 1 |

```bash
sleep 50 &
kill -9 $!   # 被信号 9 终止
wait $!
echo $?      # 137 = 128 + 9

sleep 50 &
kill -TERM $!  # 被信号 15 终止
wait $!
echo $?        # 143 = 128 + 15
```

### Bash 中的退出状态变量

| 变量 / 语法 | 说明 |
|-------------|------|
| `$?` | 最近一条前台命令的退出状态 |
| `0` | 表示成功（true） |
| `非零` | 表示失败（false），不同值可表示不同错误类型 |

```bash
true; echo $?    # 0
false; echo $?   # 1
ls /nonexistent 2>/dev/null; echo $?  # 2
```

## 各类命令的退出状态

### 内置命令

Bash 内置命令遵循以下退出状态约定：

| 退出状态 | 含义 | 示例 |
|---------|------|------|
| `0` | 执行成功 | `cd /tmp` 成功 |
| `1` | 一般性错误 | `cd /nonexistent` |
| `2` | 用法错误（参数不正确） | `cd`（缺少参数或参数无效） |

**示例**：

```bash
cd /tmp; echo $?       # 0  ← 成功
cd /nonexistent; echo $?  # 1  ← 目标不存在
cd -; cd; echo $?         # 0  ← 无参数 cd 回到 HOME（成功）
cd -- -opt; echo $?       # 1  ← 把 -opt 当目录（不存在）
cd -wrong; echo $?        # 2  ← 无效选项，用法错误
```

> **区分 `1` 和 `2`**：退出状态 `1` 表示命令执行过程中出错（运行时错误），退出状态 `2` 表示命令的用法不正确（参数错误）。

### 外部命令

外部命令被信号终止时，退出状态为 `128 + 信号编号`；正常退出时遵循程序自身的退出状态约定。

```bash
sleep 100
^C  # Ctrl+C 发送 SIGINT（信号2）
echo $?  # 130 = 128 + 2

sleep 100
kill -9 $!  # SIGKILL（信号9）
wait $!
echo $?    # 137 = 128 + 9
```

### Shell 函数

Shell 函数的退出状态为函数体中最后一条命令的退出状态，或 `return` 指定的值。

**语法**：

```bash
return [n]
```

| 参数 | 说明 |
|------|------|
| `n` | 指定退出状态（0–255）；省略则为最后一条命令的退出状态 |

**示例**：

```bash
myfunc() {
    echo "hello"
    return 5
}

myfunc; echo $?   # 5

myfunc2() {
    ls /nonexistent 2>/dev/null
}

myfunc2; echo $?  # 2（ls 返回的退出状态）
```

> **注意**：`return` 只能在函数或 `source` 执行的脚本中使用，在非 `source` 执行的脚本中使用 `return` 会报错。

### 表达式

#### 算术表达式 `((...))`

算术表达式的退出状态基于表达式的求值结果：

| 求值结果 | 退出状态 | 说明 |
|---------|---------|------|
| 非零 | `0`（成功） | 表达式为真 |
| 零 | `1`（失败） | 表达式为假 |

```bash
((1+1)); echo $?    # 0  ← 结果为 2（非零）
((1-1)); echo $?    # 1  ← 结果为 0
((5>3)); echo $?    # 0  ← 5>3 为真，值为 1
((3>5)); echo $?    # 1  ← 3>5 为假，值为 0
```

#### let 命令

`let` 的退出状态与 `((...))` 相同。

**语法**：

```bash
let 表达式 [表达式 ...]
```

```bash
let a=1+1; echo $?   # 0  ← 结果为 2（非零）
let a=1-1; echo $?   # 1  ← 结果为 0
```

### 命令列表

通过逻辑运算符组合命令时，退出状态为最后一条**被执行**的命令的退出状态：

| 运算符 | 行为 | 退出状态 |
|--------|------|---------|
| `cmd1 && cmd2` | cmd1 成功才执行 cmd2 | 最后执行的命令的退出状态 |
| `cmd1 || cmd2` | cmd1 失败才执行 cmd2 | 最后执行的命令的退出状态 |

```bash
true && echo "yes"   # yes（退出状态 0）
false && echo "yes"  # 无输出（退出状态 1，cmd2 未执行）
true || echo "no"    # 无输出（退出状态 0，cmd2 未执行）
false || echo "no"   # no（退出状态 0）
```

**混合运算**：

```bash
true && false || echo "hello"
# hello（true → 执行 false → 失败 → 执行 echo → 成功）

false && true || echo "hello"
# hello（false → 不执行 true → 执行 echo → 成功）
```

### 脚本的退出状态

| 执行方式 | 退出状态来源 |
|---------|------------|
| `source script.sh` | 脚本中最后一条命令的退出状态，或 `return N` 指定的值 |
| `./script.sh` / `bash script.sh` | 脚本中最后一条命令的退出状态，或 `exit N` 指定的值 |

```bash
# source 执行
cat > t.sh << 'EOF'
echo "hello"
exit 5  # source 执行时，exit 会导致当前 Shell 退出
EOF

source t.sh
# hello
# 当前 Shell 直接退出（exit 在 source 中生效）

# child-shell 执行
bash t.sh
echo $?  # 5
```

> **关键区别**：`source` 中的 `exit` 会终止当前 Shell；child-shell 中的 `exit` 仅终止子 Shell。

### 后台作业与 wait

**语法**：

```bash
wait [选项] [作业号或PID ...]
```

| 场景 | `$?` 值 |
|------|--------|
| `wait` 等待的后台作业正常退出 | 作业的退出状态 |
| `wait` 等待的后台作业被信号终止 | `128 + 信号编号` |
| `wait` 指定不存在的 PID | `127` |
| 不使用 `wait` 直接检查 `$?` | 始终为 `0`（`&` 本身的退出状态） |

```bash
sleep 3 &     # $! = 1234
wait 1234
echo $?       # 0（正常退出）

sleep 3 &
kill -9 $!
wait $!
echo $?       # 137（被 SIGKILL 终止）

wait 99999
echo $?       # 127（PID 不存在）
```

### 管道的退出状态

**默认行为**：管道的退出状态为**最后一个命令**的退出状态。

```bash
true | false; echo $?    # 1
false | true; echo $?    # 0
```

**pipefail 选项**：开启后，管道的退出状态为最后一个**非零**退出状态的命令的值；如果所有命令都成功退出，则为 0。

```bash
set -o pipefail

true | false | true; echo $?    # 1（false 的退出状态）
false | true | true; echo $?    # 1（false 的退出状态）
true | true | false; echo $?    # 1（false 的退出状态）
true | true | true; echo $?     # 0（全部成功）
```

**PIPESTATUS 数组**：`PIPESTATUS` 数组记录管道中每个命令的退出状态。

```bash
true | false | true | false
echo ${PIPESTATUS[@]}   # 0 1 0 1

ls /nonexistent | cat | sort
echo ${PIPESTATUS[@]}   # 2 0 0
```

> **注意**：`PIPESTATUS` 会在每次管道执行（甚至前台命令）后被重置。

```bash
ls /nonexistent | cat
echo ${PIPESTATUS[@]}   # 2 0
echo ${PIPESTATUS[@]}   # 0  ← 已被上一条 echo 重置
```

## 退出状态值约定

### 常见退出状态码

| 退出状态 | 含义 | 说明 |
|---------|------|------|
| `0` | 成功 | 命令执行成功 |
| `1` | 一般错误 | 通用错误（如文件不存在、操作失败） |
| `2` | 用法错误 | 命令参数不正确（Bash 内置命令约定） |
| `126` | 不可执行 | 文件存在但无执行权限 |
| `127` | 命令未找到 | 命令不在 PATH 中或不存在 |
| `128+N` | 被信号终止 | 进程被信号 N 终止 |

```bash
# 不可执行
chmod -x script.sh
./script.sh; echo $?  # 126

# 命令未找到
nonexistent_cmd; echo $?  # 127
```

### 自定义退出状态码

在脚本中可以使用 `exit N` 指定退出状态：

```bash
#!/bin/bash

if [ $# -lt 1 ]; then
    echo "Usage: $0 <filename>" >&2
    exit 2  # 用法错误
fi

if [ ! -f "$1" ]; then
    echo "Error: $1 not found" >&2
    exit 1  # 一般错误
fi

echo "Processing $1..."
exit 0  # 成功
```

> **建议**：在脚本中定义有意义的退出状态码，方便调用者根据退出状态判断错误类型。

## 相关命令参考

### exit 命令

**语法**：

```bash
exit [n]
```

| 参数 | 说明 |
|------|------|
| `n` | 指定退出状态码（0–255）；省略则为最后一条命令的退出状态 |

**行为说明**：

- 在脚本中：终止脚本并返回退出状态
- 在交互式 Shell 中：退出当前 Shell
- 在 `source` 执行的脚本中：终止当前 Shell

### return 命令

**语法**：

```bash
return [n]
```

| 参数 | 说明 |
|------|------|
| `n` | 指定退出状态码（0–255）；省略则为函数中最后一条命令的退出状态 |

**使用范围**：仅能在 Shell 函数或 `source` 执行的脚本中使用。

### test / [ 命令

`test` 和 `[` 用于条件判断，退出状态表示条件是否成立。

**语法**：

```bash
test 表达式
[ 表达式 ]
[[ 表达式 ]]  # Bash 扩展版本
```

| 退出状态 | 含义 |
|---------|------|
| `0` | 条件为真 |
| `1` | 条件为假 |
| `2` | 表达式语法错误 |

**常用条件表达式**：

| 表达式 | 说明 |
|--------|------|
| `-f 文件` | 文件存在且为普通文件 |
| `-d 文件` | 文件存在且为目录 |
| `-e 文件` | 文件存在 |
| `-r 文件` | 文件可读 |
| `-w 文件` | 文件可写 |
| `-x 文件` | 文件可执行 |
| `-z 字符串` | 字符串长度为 0 |
| `-n 字符串` | 字符串长度不为 0 |
| `字符串1 = 字符串2` | 字符串相等 |
| `数值1 -eq 数值2` | 数值相等 |
| `数值1 -ne 数值2` | 数值不等 |
| `数值1 -lt 数值2` | 数值1 < 数值2 |
| `数值1 -gt 数值2` | 数值1 > 数值2 |

### set 命令（退出状态相关）

**语法**：

```bash
set [-e] [-o pipefail]
```

| 选项 | 说明 |
|------|------|
| `-e` | 任何命令返回非零退出状态时立即退出脚本 |
| `-o pipefail` | 管道中任何命令失败时，管道返回该命令的退出状态 |

**示例**：

```bash
#!/bin/bash
set -euo pipefail

# -e: 命令失败时脚本立即退出
# -u: 引用未定义变量时报错
# -o pipefail: 管道中任何命令失败都会导致管道失败

false | true  # 没有 pipefail 时退出状态为 0，有 pipefail 时为 1
```

> **注意**：`set -e` 对 `&&`、`||`、`if` 条件中的命令不生效，这些命令的失败不会导致脚本退出。

## 最佳实践总结

| 场景 | 推荐做法 | 示例 |
|------|---------|------|
| 检查命令是否成功 | 使用 `$?` 或 `if` / `&&` | `cmd && echo "ok"` |
| 脚本中处理错误 | 使用 `set -e` + `set -o pipefail` | `set -euo pipefail` |
| 区分错误类型 | 使用不同的退出状态码 | `exit 1` / `exit 2` |
| 管道错误检测 | 使用 `PIPESTATUS` 数组 | `${PIPESTATUS[@]}` |
| 函数返回状态 | 使用 `return` 而非 `exit` | `return 1` |
| 获取管道各命令状态 | `PIPESTATUS` + `pipefail` | `set -o pipefail` |
| 判断信号终止 | 检查退出状态是否 ≥ 128 | `if (( $? >= 128 ))` |

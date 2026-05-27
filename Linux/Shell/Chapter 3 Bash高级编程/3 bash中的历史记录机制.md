# Bash 历史记录机制

> **本章概述**：Bash 的历史记录机制允许用户回顾、搜索和重复执行之前输入的命令。通过合理配置历史记录变量和选项，可以大幅提升命令行操作效率，并避免因误操作导致的安全问题。

## 基本用法

### history 命令

`history` 是 Bash 内置命令，用于查看和管理命令历史记录。

**语法**：

```bash
history [选项] [数量]
history -d 偏移量
history -c
history -a [文件名]
history -n [文件名]
history -r [文件名]
history -w [文件名]
history -p 参数 ...
history -s 参数 ...
```

**参数说明**：

| 参数 | 说明 |
|------|------|
| `数量` | 显示最近指定数量的历史记录 |
| `-d 偏移量` | 删除指定偏移量（序号）的历史记录 |
| `-c` | 清空当前会话的历史记录缓冲队列（不影响历史文件） |
| `-a [文件名]` | 将当前会话新增的命令追加写入历史文件（默认 `$HISTFILE`） |
| `-n [文件名]` | 从历史文件中读取尚未加载的新记录 |
| `-r [文件名]` | 从历史文件中读取全部记录并替换当前缓冲队列 |
| `-w [文件名]` | 将当前缓冲队列覆盖写入历史文件 |
| `-p 参数` | 对参数执行历史扩展但不写入历史记录 |
| `-s 参数` | 将参数作为单条记录追加到历史列表（不执行） |

**常用示例**：

```bash
# 显示全部历史记录
history
    1  ip a
    2  exit
    3  ls -la
    4  history

# 显示最后两条
history 2
    4  history
    5  history 2

# 删除第 5 条记录
history -d 5

# 清空当前会话历史
history -c

# 手动保存到文件
history -w    # 覆盖写入
history -a    # 追加写入
```

### 搜索历史记录

**交互式搜索（Ctrl+R）**：

按 `Ctrl+R` 进入增量搜索模式，输入字符后 Bash 按照最近最相似原则匹配历史命令。再次按 `Ctrl+R` 可继续向上搜索更早的匹配项。

```bash
# 按 Ctrl+R 后输入搜索关键词
(reverse-i-search)`ffmpeg': ffmpeg -y -f gif -i input.gif output.mp4
# 按 Enter 执行；按 Esc 编辑；按 Ctrl+C 取消
```

**管道搜索**：

```bash
history | grep ffmpeg | grep gif | grep yuv444p
  119  ffmpeg -y -f gif -i 79557166.gif -c:v libx264 -vf format=yuv444p yuv444p.mp4
  120  ffmpeg -y -f gif -i 79557166.gif -c:v libx264 -vf format=yuv444p yuv444p.mkv
```

### 快速执行历史命令

| 语法 | 说明 | 危险等级 |
|------|------|---------|
| `!!` | 执行上一条命令 | ⚠️ 中等 |
| `!n` | 执行编号为 n 的历史命令 | 🔴 高 |
| `!-n` | 执行倒数第 n 条历史命令 | 🔴 高 |
| `!字符串` | 执行最近一条以指定字符串开头的历史命令 | 🔴 极高 |
| `!?字符串` | 执行最近一条包含指定字符串的历史命令 | 🔴 极高 |

> **安全提示**：`!` 开头的命令会直接执行，不会要求确认。在高权限环境下（如 root），误执行可能造成严重后果。

**安全查看方式**（先预览再决定是否执行）：

```bash
!!:p        # 查看最后一条命令（不执行）
!123:p      # 查看第 123 条命令
!sys:p      # 查看以 sys 开头的最近一条命令
```

预览后，如果确认无误，可用 `!!` 执行上一条（即刚才预览的命令）。

**示例**：

```bash
> date
Sun 23 Feb 2020 06:18:16 PM CST
> !!        # 重复上一条
date
Sun 23 Feb 2020 06:18:18 PM CST

> !907      # 执行编号 907 的命令（危险！）
make
make: *** No targets specified and no makefile found.  Stop.

> !shut     # 执行最近以 shut 开头的命令（极度危险！）
shutdown now
```

### 删除历史记录

```bash
# 删除指定编号的记录
history -d 5

# 执行不被记录的命令（利用 history -d 删除自身）
echo "secret command"; history -d $(history 1)

# 清空当前会话的所有历史记录（仅清空缓冲队列，不影响文件）
history -c
```

> **警告**：`unset HISTFILE` 可直接取消当前会话的历史记录功能，所有后续命令都不会被记录。这在服务器维护场景中非常危险，因为无法审计操作。

### 文件操作

在终端退出时，Bash 会自动将当前会话的命令写入历史文件。也可以手动操作：

```bash
history -w    # 将缓冲队列覆盖写入文件（清空文件后写入）
history -a    # 将当前会话新增的命令追加写入文件
history -r    # 从文件重新读取全部记录（替换当前缓冲队列）
history -n    # 从文件读取尚未加载的新记录
```

## 高级配置

### 历史记录相关变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `HISTFILE` | `~/.bash_history` | 历史记录存储文件路径 |
| `HISTSIZE` | `1000` | 内存中保存的历史记录条数（`history` 命令显示的数量） |
| `HISTFILESIZE` | `2000` | 磁盘文件中保存的历史记录条数 |
| `HISTTIMEFORMAT` | （空） | 时间戳格式字符串，如 `'%F %T '` |
| `HISTCONTROL` | `ignoredups` | 控制记录策略 |
| `HISTIGNORE` | （空） | 指定要忽略记录的命令（冒号分隔） |
| `HISTCMD` | — | 当前命令的历史编号（只读） |

### 添加时间戳

通过 `HISTTIMEFORMAT` 变量为历史记录添加时间戳：

```bash
# 临时生效
export HISTTIMEFORMAT='%F %T '

# 永久生效
echo 'export HISTTIMEFORMAT="%F %T "' >> ~/.bashrc
source ~/.bashrc

# 效果
history 1
1032  2020-02-23 17:01:26 history
```

**常用格式说明**：

| 格式符 | 含义 | 示例 |
|--------|------|------|
| `%F` | 日期（等价于 `%Y-%m-%d`） | `2020-02-23` |
| `%T` | 时间（等价于 `%H:%M:%S`） | `17:01:26` |
| `%Y` | 四位年份 | `2020` |
| `%m` | 两位月份 | `02` |
| `%d` | 两位日期 | `23` |
| `%H` | 24小时制小时 | `17` |
| `%M` | 分钟 | `01` |
| `%S` | 秒 | `26` |

### 修改记录策略（HISTCONTROL）

`HISTCONTROL` 变量控制哪些命令被记录：

| 可选值 | 策略 | 说明 |
|--------|------|------|
| `ignoredups` | 不记录连续重复命令 | 仅当与上一条相同时忽略（默认） |
| `ignorespace` | 不记录空格开头的命令 | 以空格开头的命令不会被记录 |
| `ignoreboth` | ignoredups + ignorespace | 同时启用两种策略 |
| `erasedups` | 不记录所有重复命令 | 每次记录前删除历史中所有相同的旧记录 |

```bash
# 临时设置
export HISTCONTROL=ignoreboth:erasedups

# 永久设置
echo 'export HISTCONTROL="ignoreboth:erasedups"' >> ~/.bashrc
source ~/.bashrc
```

> **安全技巧**：设置 `ignorespace` 后，在敏感命令前加空格可避免记录：` space secret_command`

### 忽略指定命令（HISTIGNORE）

`HISTIGNORE` 变量指定不记录的命令，多个命令用冒号 `:` 分隔，支持 `&` 匹配上一条命令：

```bash
# 忽略 ls 和 cd 命令
export HISTIGNORE="ls:cd"

# 忽略 ls、cd 以及所有以 pwd 开头的命令
export HISTIGNORE="ls:cd:pwd*"

# 忽略 ls、cd 和连续重复的命令（& 匹配上一条）
export HISTIGNORE="ls:cd:&"

# 永久设置
echo 'export HISTIGNORE="ls:cd:pwd"' >> ~/.bashrc
source ~/.bashrc
```

### 修改存储文件

```bash
# 临时修改
export HISTFILE="~/my_history"

# 永久修改
echo 'export HISTFILE="~/my_history"' >> ~/.bashrc
source ~/.bashrc
```

### 修改存储大小

```bash
# 查看当前值
echo $HISTSIZE        # 内存中保存条数
echo $HISTFILESIZE    # 文件中保存条数

# 临时修改
HISTSIZE=5000
HISTFILESIZE=10000

# 永久修改
sed -i 's/^HISTSIZE=1000/HISTSIZE=5000/' ~/.bashrc
sed -i 's/^HISTFILESIZE=2000/HISTFILESIZE=10000/' ~/.bashrc
source ~/.bashrc
```

> **建议**：将 `HISTSIZE` 和 `HISTFILESIZE` 设置为较大值（如 50000），避免有价值的命令历史被截断。

### 修改存储策略

**问题**：默认情况下，多个终端同时使用时，后关闭的终端会覆盖先关闭终端的历史记录。

```
开启终端A → 开启终端B → 关闭终端A → 关闭终端B
# 终端A 的历史记录全部丢失！
```

**解决方案 1：追加写入（shopt -s histappend）**

```bash
# 在 ~/.bashrc 中添加
shopt -s histappend
```

开启 `histappend` 后，终端退出时以追加方式写入历史文件，而非覆盖。

**解决方案 2：实时追加写入（PROMPT_COMMAND）**

```bash
# 在 ~/.bashrc 中添加
PROMPT_COMMAND="history -a"
```

`PROMPT_COMMAND` 变量的值会在每次显示命令提示符之前作为命令执行。设置为 `history -a` 后，每条命令执行后都会立即追加写入文件，即使异常断电也不会丢失。

**推荐的综合配置**：

```bash
# 追加到 ~/.bashrc
shopt -s histappend                    # 退出时追加写入
PROMPT_COMMAND="history -a"            # 每条命令实时写入
export HISTSIZE=5000                   # 内存保存 5000 条
export HISTFILESIZE=10000              # 文件保存 10000 条
export HISTTIMEFORMAT='%F %T '         # 添加时间戳
export HISTCONTROL=ignoreboth:erasedups # 去重 + 忽略空格开头
export HISTIGNORE="ls:cd:pwd:exit"     # 忽略简单命令
```

## 历史扩展语法

### 事件指示符

| 语法 | 说明 | 示例 |
|------|------|------|
| `!n` | 第 n 条历史命令 | `!100` |
| `!-n` | 倒数第 n 条历史命令 | `!-3` |
| `!!` | 上一条命令（等价于 `!-1`） | `!!` |
| `!字符串` | 最近一条以字符串开头的命令 | `!vim` |
| `!?字符串?` | 最近一条包含字符串的命令 | `!?config?` |
| `^旧^新` | 替换上一条命令中的字符串 | `^old^new` |

**示例**：

```bash
> echo "hello world"
hello world
> ^hello^hi            # 替换上一条中的 hello → hi
echo "hi world"
hi world
```

### 词指示符

在事件指示符后可追加 `:词指示符` 来选择命令的特定部分：

| 指示符 | 说明 | 示例（假设上一条是 `echo hello world`） |
|--------|------|--------------------------------------|
| `:0` | 命令名 | `!!:0` → `echo` |
| `:n` | 第 n 个参数（从1开始） | `!!:2` → `world` |
| `:^` | 第一个参数 | `!!:^` → `hello` |
| `:$` | 最后一个参数 | `!!:$` → `world` |
| `:*` | 所有参数 | `!!:*` → `hello world` |
| `:n-m` | 第 n 到 m 个参数 | `!!:1-2` → `hello world` |
| `:n*` | 第 n 个到最后一个参数 | `!!:1*` → `hello world` |
| `:n-` | 第 n 到倒数第二个参数 | `!!:1-` → `hello` |

### 修饰符

在词指示符后可追加修饰符进行进一步处理：

| 修饰符 | 说明 | 示例 |
|--------|------|------|
| `:h` | 删除文件名，保留目录 | `/usr/bin/ls` → `/usr/bin` |
| `:t` | 删除目录，保留文件名 | `/usr/bin/ls` → `ls` |
| `:r` | 删除后缀 | `file.tar.gz` → `file.tar` |
| `:e` | 仅保留后缀 | `file.tar.gz` → `.gz` |
| `:p` | 仅打印，不执行 | `!!:p` |
| `:s/旧/新` | 替换第一个匹配 | `!!:s/old/new/` |
| `:gs/旧/新` | 替换所有匹配 | `!!:gs/old/new/` |

## 历史记录相关命令参考

### fc 命令

`fc`（Fix Command）是 POSIX 标准的历史命令编辑工具。

**语法**：

```bash
fc [-e 编辑器] [-lnr] [首条] [末条]
fc -s [模式=替换 ...] [命令]
```

**参数说明**：

| 参数 | 说明 |
|------|------|
| `-e 编辑器` | 指定编辑器（默认 FCEDIT 或 EDITOR） |
| `-l` | 列出历史记录（不编辑） |
| `-n` | 列出时不显示编号 |
| `-r` | 逆序列出 |
| `-s 模式=替换` | 直接替换并执行 |
| `首条 末条` | 指定范围（编号或负数偏移） |

**示例**：

```bash
# 列出最近 10 条历史
fc -l -10

# 用 vi 编辑第 100-110 条命令
fc -e vi 100 110

# 替换上一条命令中的 old 为 new 并执行
fc -s old=new

# 列出不带编号的历史
fc -ln -5
```

### shopt 命令

`shopt` 是 Bash 内置命令，用于查看和设置 Shell 选项。

**语法**：

```bash
shopt [选项] [选项名 ...]
```

**与历史记录相关的选项**：

| 选项名 | 默认 | 说明 |
|--------|------|------|
| `histappend` | 关闭 | 开启后退出时追加写入历史文件（而非覆盖） |
| `histreedit` | 关闭 | 开启后历史扩展失败时可重新编辑 |
| `histverify` | 关闭 | 开启后历史扩展结果不立即执行，而是加载到编辑缓冲区 |
| `lithist` | 关闭 | 开启后多行命令以换行符而非分号保存 |

**常用示例**：

```bash
# 查看所有选项
shopt

# 查看特定选项
shopt histappend

# 开启选项
shopt -s histappend
shopt -s histverify    # 历史扩展后可先编辑再执行

# 关闭选项
shopt -u histappend
```

## 最佳实践总结

| 场景 | 推荐做法 | 配置 |
|------|---------|------|
| 防止多终端历史丢失 | 开启追加写入 | `shopt -s histappend` |
| 防止断电丢失 | 实时写入 | `PROMPT_COMMAND="history -a"` |
| 敏感命令不留记录 | 空格前缀 + ignorespace | `HISTCONTROL=ignorespace` |
| 去除重复命令 | erasedups | `HISTCONTROL=ignoreboth:erasedups` |
| 追踪命令执行时间 | 添加时间戳 | `HISTTIMEFORMAT='%F %T '` |
| 保留足够历史 | 增大存储容量 | `HISTSIZE=5000 HISTFILESIZE=10000` |
| 快速重复上一条 | `!!` | — |
| 安全预览历史命令 | `!:p` | — |
| 忽略简单命令 | HISTIGNORE | `HISTIGNORE="ls:cd:pwd:exit"` |

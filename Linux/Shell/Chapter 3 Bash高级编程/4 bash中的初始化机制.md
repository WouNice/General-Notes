# Bash 初始化机制

> **本章概述**：Bash 在启动时会根据不同的运行模式（交互式/非交互式、登录/非登录）加载不同的配置文件。理解初始化机制对于正确配置环境变量、别名和函数至关重要，也是排查 Shell 环境问题的必备知识。

## Shell 的运行模式

### 四种基本模式

Bash 有两种独立维度的运行模式分类：

| 维度 | 模式 | 说明 |
|------|------|------|
| **交互性** | 交互式（Interactive） | 可接受用户输入，有命令提示符 |
| | 非交互式（Non-interactive） | 执行脚本，不与用户交互 |
| **登录** | 登录 Shell（Login Shell） | 通过系统身份验证获得的 Shell |
| | 非登录 Shell（Non-login Shell） | 在已有会话中启动的 Shell |

组合后形成四种模式，每种加载不同的配置文件：

| 模式 | 加载的配置文件 |
|------|---------------|
| 交互式登录 Shell | `/etc/profile` → `~/.bash_profile` / `~/.bash_login` / `~/.profile`（首个找到的） → 退出时 `~/.bash_logout` |
| 交互式非登录 Shell | `~/.bashrc` |
| 非交互式登录 Shell | `/etc/profile` → `~/.bash_profile` / `~/.bash_login` / `~/.profile` |
| 非交互式非登录 Shell | `$BASH_ENV` 指向的文件 |

### 判断当前 Shell 模式

**方法一：检查特殊变量 `$-`**

```bash
echo $-
himBHs  # 包含 'i' 表示交互式 Shell
```

| 字符 | 含义 |
|------|------|
| `i` | 交互式 Shell |
| `h` | 启用哈希命令查找 |
| `m` | 启用作业控制 |
| `B` | 启用花括号扩展 |
| `H` | 启用 `!` 风格的历史扩展 |
| `s` | 从标准输入读取命令 |

**方法二：检查 PS1 变量**

```bash
if [ -z "$PS1" ]; then
    echo "非交互式"
else
    echo "交互式"
fi
```

**方法三：检查是否为登录 Shell**

```bash
# 使用 shopt
shopt -q login_shell && echo '登录 Shell' || echo '非登录 Shell'

# 尝试 logout 命令（仅登录 Shell 可用）
logout  # 非登录 Shell 会提示: bash: logout: not login shell: use `exit'
```

## 各模式的初始化流程

### 交互式登录 Shell

**获得登录 Shell 的场景**：

| 场景 | 示例 |
|------|------|
| 通过本地终端登录 | Ctrl+Alt+F1 切换到 TTY |
| 通过 SSH 远程登录 | `ssh user@host` |
| 使用 `--login` 选项 | `bash --login` |
| 使用 `su -` 切换用户 | `su - username` |

**初始化流程**：

```
启动
 ↓
读取 /etc/profile（系统全局配置）
 ↓
查找并读取首个可读的用户配置文件（按顺序）：
  ~/.bash_profile → ~/.bash_login → ~/.profile
 ↓
Shell 运行中...
 ↓
退出时读取 ~/.bash_logout
```

**`/etc/profile` 的默认行为**：

- 定义 `PATH`、`USER`、`MAIL`、`HOSTNAME`、`HISTSIZE` 等全局环境变量
- 导入 `/etc/bash.bashrc`（系统级 Shell 函数和别名）
- 导入 `/etc/profile.d/*.sh`（针对特定程序的初始化脚本）

> **重要**：`~/.bash_profile`、`~/.bash_login`、`~/.profile` 三个文件中 Bash **只读取首个找到且可读的**。在大多数 Linux 发行版中，`~/.bash_profile` 会 `source ~/.bashrc`，确保两种模式获得相同配置。

**典型 `~/.bash_profile` 内容**：

```bash
# 加载 .bashrc
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi

# 设置 PATH 等环境变量
export PATH="$HOME/bin:$PATH"
```

### 交互式非登录 Shell

**获得非登录 Shell 的场景**：

| 场景 | 示例 |
|------|------|
| GUI 桌面中打开终端 | Ubuntu 终端、GNOME Terminal |
| 在已有 Shell 中输入 `bash` | `bash` |
| 使用 `su` 不带 `-` | `su username` |
| 使用 `screen` / `tmux` 新建窗口 | 新建窗格 |

**初始化流程**：

```
启动
 ↓
读取 ~/.bashrc
 ↓
Shell 运行中...
```

> **最佳实践**：将所有个人配置（别名、函数、环境变量）定义在 `~/.bashrc` 中，在 `~/.bash_profile` 中 `source ~/.bashrc`，确保登录和非登录 Shell 都能获得相同配置。

### 非交互式 Shell

**获得非交互式 Shell 的场景**：

| 场景 | 示例 |
|------|------|
| 执行 Shell 脚本 | `bash script.sh` / `./script.sh` |
| 管道中的命令 | `echo "hello" | bash` |
| 命令替换 | `$(bash -c 'echo hi')` |

**初始化流程**：

```
启动
 ↓
读取 $BASH_ENV 指向的文件（如果已设置）
 ↓
执行脚本内容
 ↓
退出
```

> **注意**：
> - `BASH_ENV` 的值必须是**绝对路径**，因为非交互式 Shell 不会加载 `PATH`
> - 非交互式 Shell 不会读取 `/etc/profile`、`~/.bashrc` 等常规配置文件
> - 脚本首行的 `#!/bin/bash --login` 可让脚本以非交互式登录 Shell 运行

## 特殊模式

### 兼容模式（sh）

当使用 `sh` 命令调用 Bash 时，为保证兼容性会按 `sh` 的方式初始化：

| 模式 | 加载文件 |
|------|---------|
| 登录 Shell | `/etc/profile` → `~/.profile` |
| 非登录 Shell | `$ENV` 指向的文件 |

> **原因**：`sh` 是 POSIX 标准的 Shell，Bash 在以 `sh` 启动时会禁用自身扩展功能，尽量模拟 POSIX 行为。

### POSIX 模式

通过以下方式可让 Bash 进入 POSIX 模式：

```bash
bash --posix
# 或
set -o posix
# 或
export POSIXLY_CORRECT=1
```

**POSIX 模式下的行为变化**：

- 仅读取 `$ENV` 指向的文件
- 禁用大部分 Bash 扩展功能
- 函数定义和使用遵循 POSIX 语法
- 命令查找遵循 POSIX 规则

### 远程启动

通过 `rshd` 远程启动脚本时，仅加载 `~/.bashrc` 文件。

> **安全警告**：`rlogin`、`telnet`、`rsh`、`rcp` 等远程命令传输**未加密的明文信息**，存在严重安全隐患。应始终使用 SSH 替代。

```bash
# ❌ 不安全
rsh remotehost command

# ✅ 安全替代
ssh user@remotehost command
```

### UID 与 EUID 不匹配

**概念说明**：

| 概念 | 全称 | 说明 |
|------|------|------|
| **UID** | 真实用户 ID（Real User ID） | 创建进程的用户 ID |
| **EUID** | 有效用户 ID（Effective User ID） | 进程当前的权限级别 |

一般情况下 `UID = EUID`。但当可执行文件的 **SUID 位**被设置时（如 `-rwsr-xr-x`，用户执行位 `x` 变为 `s`），进程以文件所有者的权限运行，导致 `EUID ≠ UID`。

**对 Bash 的影响**：

如果给 `bash` 可执行文件设置了 SUID 位（所有者为 root），非 root 用户运行时 `UID ≠ EUID`。为安全起见，Bash 在这种情况下**不会加载任何配置文件**。

```bash
# 查看 bash 的权限
ls -l /bin/bash
# -rwxr-xr-x 1 root root ... /bin/bash  ← 默认无 SUID

# 如果设置了 SUID（极度危险！）
chmod u+s /bin/bash
# -rwsr-xr-x 1 root root ... /bin/bash  ← 其他用户运行时 EUID=0
# Bash 检测到 UID ≠ EUID，不加载任何配置文件
```

> **警告**：绝对不要给 Bash 设置 SUID 位，这等同于为所有用户提供了 root 权限。

### 受限制的 Shell（rbash）

通过以下方式启动受限制的 Shell：

```bash
rbash
bash --restricted
bash -r
```

**功能限制清单**：

| 限制项 | 说明 |
|--------|------|
| 禁止 `cd` | 无法切换目录 |
| 禁止路径中的 `/` | 命令和文件名中不能包含 `/` |
| 禁止修改关键变量 | 不能更改 `SHELL`、`PATH`、`ENV`、`BASH_ENV` |
| 禁止 `source` 含路径的文件 | `source` 参数不能含 `/` |
| 禁止 `hash -p` 含路径 | `hash -p` 的路径参数不能含 `/` |
| 禁止导入函数 | 初始化时不导入文件中的函数 |
| 忽略 `SHELLOPTS` | 不继承父 Shell 的选项设置 |
| 禁止重定向 | 不能使用 `>`、`>>`、`<>` 等 |
| 禁止 `exec` | 不能使用 `exec` 替换进程 |
| 禁止加载内置命令 | 不能使用 `enable -f/-d` |
| 禁止 `command -p` | 不能使用 `command -p` 指定路径 |
| 禁止退出限制模式 | 无法主动关闭限制模式 |

**安全漏洞**：如果环境变量配置不当，用户可以轻松绕过限制：

```bash
rbash
cd /etc  # rbash: cd: restricted
bash     # 直接启动一个不受限的 Bash
cd /etc  # 成功！
```

**安全配置方案**：创建受限用户并严格控制可执行命令：

```bash
# 创建使用 rbash 的用户
useradd -s /bin/rbash ruser

# 设置配置文件权限（root 拥有，ruser 组只读）
chown -R root:ruser /home/ruser/.bashrc /home/ruser/.bash_profile
chmod 640 /home/ruser/.bashrc /home/ruser/.bash_profile

# 创建受限命令目录
mkdir /home/ruser/bin

# 限制 PATH 为该目录
echo "export PATH=/home/ruser/bin" >> /home/ruser/.bash_profile

# 仅链接允许的命令
ln -s /usr/bin/ftp /home/ruser/bin/ftp
ln -s /usr/bin/ls /home/ruser/bin/ls
# 只链接必要的命令，不链接 bash、sh 等

# 禁止用户写入 bin 目录
chown root:root /home/ruser/bin
chmod 755 /home/ruser/bin
```

## 配置文件速查表

### 文件加载流程图

```
                    ┌─────────────────────┐
                    │    Bash 启动        │
                    └─────────┬───────────┘
                              │
                    ┌─────────▼───────────┐
                    │  是否为登录 Shell？  │
                    └────┬───────────┬────┘
                    是   │           │  否
              ┌─────────▼──┐    ┌───▼──────────┐
              │/etc/profile│    │              │
              └─────┬──────┘    │ 是否交互式？ │
                    │           └──┬───────┬───┘
              ┌─────▼──────┐   是  │       │  否
              │~/.bash_    │       │       │
              │profile or  │  ┌────▼──┐ ┌──▼──────┐
              │~/.bash_    │  │~/.    │ │$BASH_   │
              │login or    │  │bashrc │ │ENV      │
              │~/.profile  │  └───────┘ └─────────┘
              └────────────┘
```

### 配置文件职责分工

| 文件 | 加载场景 | 推荐用途 |
|------|---------|---------|
| `/etc/profile` | 所有登录 Shell | 系统级环境变量和 PATH 设置 |
| `/etc/bash.bashrc` | 所有交互式 Shell | 系统级别名和函数 |
| `/etc/profile.d/*.sh` | 所有登录 Shell | 各程序独立的初始化脚本 |
| `~/.bash_profile` | 用户登录 Shell | 环境变量 + source ~/.bashrc |
| `~/.bashrc` | 用户交互式 Shell | 别名、函数、提示符、shopt 设置 |
| `~/.bash_logout` | 用户登录 Shell 退出 | 清理临时文件、历史记录等 |
| `$BASH_ENV` | 非交互式非登录 Shell | 脚本运行环境配置 |

### 各模式加载文件对照表

| 配置文件 | 交互式登录 | 交互式非登录 | 非交互式登录 | 非交互式非登录 | POSIX | sh 兼容 |
|---------|:---------:|:----------:|:----------:|:------------:|:-----:|:------:|
| `/etc/profile` | ✅ | ❌ | ✅ | ❌ | ❌ | ✅(登录) |
| `~/.bash_profile` | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ |
| `~/.bash_login` | ✅* | ❌ | ✅* | ❌ | ❌ | ❌ |
| `~/.profile` | ✅* | ❌ | ✅* | ❌ | ❌ | ✅(登录) |
| `~/.bashrc` | ❌** | ✅ | ❌** | ❌ | ❌ | ❌ |
| `$BASH_ENV` | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ |
| `$ENV` | ❌ | ❌ | ❌ | ❌ | ✅ | ✅(非登录) |
| `~/.bash_logout` | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |

> *仅当前面的文件未找到时才读取
> **如果 `~/.bash_profile` 中 `source ~/.bashrc` 则间接加载

## 相关命令参考

### su 命令

`su`（Switch User）用于切换当前用户身份。

**语法**：

```bash
su [选项] [用户名]
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| `-` 或 `-l` 或 `--login` | 模拟完整登录（启动登录 Shell，重置环境） |
| `-c 命令` | 以目标用户身份执行命令后返回 |
| `-s Shell` | 指定登录 Shell |
| `-m` / `-p` | 保留当前环境变量（不重置） |

**示例**：

```bash
# 切换到 root（获得登录 Shell）
su - root

# 切换到指定用户（获得非登录 Shell，保留环境）
su username

# 以 root 身份执行单条命令
su -c "apt update"

# 指定 Shell
su -s /bin/zsh username
```

### shopt 命令

`shopt` 用于查看和设置 Bash 的 Shell 选项。

**语法**：

```bash
shopt [选项] [选项名 ...]
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| `-s` | 开启（Set）指定选项 |
| `-u` | 关闭（Unset）指定选项 |
| `-q` | 静默模式（用于脚本判断） |
| `-o` | 限制为 `set -o` 可设置的选项 |
| `-p` | 显示所有可设置的选项及其状态 |

**与初始化相关的选项**：

| 选项名 | 说明 |
|--------|------|
| `login_shell` | 只读，标识当前是否为登录 Shell |
| `huponexit` | 控制退出时是否向作业发送 SIGHUP |
| `histappend` | 控制历史记录是否追加写入 |

## 最佳实践总结

| 场景 | 推荐做法 |
|------|---------|
| 个人环境变量 | 定义在 `~/.bashrc` 中，由 `~/.bash_profile` source |
| 系统级配置 | 定义在 `/etc/profile.d/*.sh` 中（勿直接修改 `/etc/profile`） |
| 脚本运行环境 | 设置 `BASH_ENV` 为包含必要变量的文件（绝对路径） |
| 受限用户 | 使用 `rbash` + 严格 PATH 控制 |
| 排查环境问题 | 检查 `echo $-` 和 `shopt login_shell` 确定当前模式 |
| 避免远程明文 | 始终使用 SSH，禁止 rsh/telnet |

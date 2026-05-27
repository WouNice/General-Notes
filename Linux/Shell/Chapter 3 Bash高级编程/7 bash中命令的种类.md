# Bash 中命令的种类

> **本章概述**：Bash 中的命令分为 Shell 函数、内置命令、外部可执行文件和别名四大类，它们有不同的优先级和实现方式。理解命令的分类和解析顺序，是掌握 Bash 行为和扩展 Shell 功能的基础。本章还将介绍如何实现自定义的 Shell 函数命令、内置命令和外部命令。

## 命令的种类与优先级

### 四类命令

| 类型 | 说明 | 示例 | 速度 |
|------|------|------|------|
| **别名（Alias）** | 命令的简写或缩写 | `alias ll='ls -l'` | 最快 |
| **Shell 函数** | 用 Shell 语法编写的函数 | `myfunc() { ... }` | 快 |
| **内置命令** | 编译在 Bash 内部的命令 | `cd`、`echo`、`type` | 快 |
| **外部可执行文件** | 独立的二进制程序或脚本 | `/bin/ls`、`/usr/bin/find` | 较慢（需 fork+exec） |

### 命令解析顺序

当输入一条命令时，Bash 按以下顺序查找：

```
1. 别名（Alias）
   ↓ （如果找到，展开后重新从第1步开始查找）
2. Shell 关键字（if, for, while 等）
   ↓
3. Shell 函数
   ↓
4. 内置命令
   ↓
5. 外部可执行文件（按 PATH 顺序搜索）
```

> **注意**：别名展开后会重新从第 1 步开始查找，可能产生递归。Bash 通过检测递归来避免无限循环。

```bash
alias ls='ls --color=auto'
# 输入 ls → 展开为 "ls --color=auto" → 重新查找 → 内置/外部 ls → 执行
```

### type 命令

`type` 是 Bash 内置命令，用于查看命令的类型和定义来源。

**语法**：

```bash
type [-afptP] 名称 [名称 ...]
```

**参数说明**：

| 参数 | 说明 |
|------|------|
| `-a` | 显示所有匹配项（包括别名、函数、内置、外部） |
| `-f` | 抑制函数查找 |
| `-p` | 仅显示外部命令的路径 |
| `-P` | 强制按 PATH 查找（忽略别名和函数） |
| `-t` | 仅显示类型（alias / function / builtin / file / keyword） |

**示例**：

```bash
type ls
# ls is aliased to `ls --color=auto'

type -t ls
# alias

type -a ls
# ls is aliased to `ls --color=auto'
# ls is /usr/bin/ls

type cd
# cd is a shell builtin

type myfunc
# myfunc is a function
# myfunc () { ... }

type -p grep
# /usr/bin/grep

type -P echo
# /usr/bin/echo  ← 忽略内置 echo，找到外部版本
```

### command 与 builtin 命令

**command 命令**：绕过别名和函数查找，直接执行内置命令或外部命令。

**语法**：

```bash
command [-pVv] 命令 [参数 ...]
```

| 参数 | 说明 |
|------|------|
| `-p` | 使用默认 PATH 查找命令 |
| `-V` | 显示命令的详细描述 |
| `-v` | 显示命令的路径或定义 |

**builtin 命令**：强制执行内置命令，忽略同名函数和外部命令。

**语法**：

```bash
builtin 命令 [参数 ...]
```

**示例**：

```bash
# 绕过同名函数
cd() { echo "fake cd"; }
cd /tmp        # fake cd（调用了函数）
command cd /tmp  # 真正切换目录（绕过函数）
builtin cd /tmp  # 真正切换目录（强制使用内置命令）

# 查看命令来源
command -v ls     # alias ls='ls --color=auto'
command -v cd     # cd
command -V cd     # cd is a shell builtin
```

### enable 命令

`enable` 用于启用或禁用 Bash 内置命令。

**语法**：

```bash
enable [-a] [-d] [-f 文件] [-n] [-s] [内置命令 ...]
```

**参数说明**：

| 参数 | 说明 |
|------|------|
| `-a` | 列出所有内置命令及其状态 |
| `-d` | 删除通过 `-f` 加载的内置命令 |
| `-f 文件` | 从动态库文件加载新的内置命令 |
| `-n` | 禁用指定内置命令 |
| `-s` | 仅列出 POSIX 特殊内置命令 |

**示例**：

```bash
# 列出所有内置命令
enable -a

# 禁用内置命令（使用外部版本替代）
enable -n cd
type cd  # cd is /usr/bin/cd

# 重新启用
enable cd

# 禁用后使用外部命令
enable -n echo
echo "test"  # 调用 /usr/bin/echo
```

## 实现 Shell 函数命令

### 函数定义语法

```bash
[function] 函数名 [()] {
    命令列表
    [return n]
}
```

### 示例：pskill 函数

实现一个按名称杀死进程的命令：

```bash
pskill() {
    local pid
    if [ $# -lt 1 ]; then
        echo "Usage: pskill process_name" >&2
        return 2
    fi

    pid=$(ps -e -o pid,comm | awk -v proc="$1" '$2 ~ proc {print $1}')

    if [ -z "$pid" ]; then
        echo "No matching process found for: $1" >&2
        return 1
    fi

    echo "Killing: $pid"
    kill -s SIGTERM $pid
    return 0
}
```

### 持久化函数

将函数定义写入 `~/.bashrc` 或 `~/.bash_functions`（然后在 `.bashrc` 中 source）：

```bash
# ~/.bashrc 中添加
if [ -f ~/.bash_functions ]; then
    . ~/.bash_functions
fi
```

**使用 `declare -f` 导出函数**：

```bash
# 查看函数定义
declare -f pskill

# 导出函数（使 child-shell 可用）
export -f pskill

# 查看所有已导出的函数
declare -Fx
```

## 实现内置命令

### Bash 内置命令的实现原理

Bash 的内置命令是用 C 语言编写并编译到 Bash 可执行文件中的。其实现流程如下：

```
1. 编写 .def 文件（命令定义）
     ↓
2. mkbuiltins 工具生成 .c 和 .h 文件
     ↓
3. 编译为动态库（.so）或静态链接到 Bash
     ↓
4. Bash 启动时注册内置命令
     ↓
5. 用户输入命令时调用对应的 C 函数
```

### Bash 5.0 源码结构

```
bash-5.0/
├── builtins/
│   ├── scarlet.def      ← 各内置命令的定义文件
│   ├── cd.def
│   ├── echo.def
│   ├── kill.def
│   ├── trap.def
│   └── ...
├── mkbuiltins/           ← 内置命令生成工具
├── Makefile.in           ← 编译配置
└── config.h.in
```

### .def 文件格式

每个 `.def` 文件定义一个内置命令，包含以下部分：

```c
// echo.def 示例结构
$PROGNAME = echo
$BUILTIN = echo
$FUNCTION = echo_builtin
$SHORT_DOC = echo [-neE] [arg ...]
$LONG_DOC = ...
$DEPENDS_ON = 1

#include <stdio.h>
#include "builtins.h"
#include "shell.h"

int echo_builtin(WORD_LIST *list) {
    // 实现逻辑
    return EXECUTION_SUCCESS;
}
```

### mkbuiltins 工具

`mkbuiltins` 读取 `.def` 文件生成：

| 生成文件 | 说明 |
|---------|------|
| `builtins.c` | 内置命令分发表 |
| `builtext.h` | 函数声明和结构体定义 |
| `*.c` | 每个内置命令的 C 源码 |

### 动态加载内置命令

Bash 支持通过 `enable -f` 从动态库加载内置命令：

```bash
# 编译自定义内置命令为 .so
gcc -shared -fPIC -o mycmd.so mycmd.c

# 加载
enable -f ./mycmd.so mycmd

# 使用
mycmd arg1 arg2

# 卸载
enable -d mycmd
```

> **注意**：动态加载内置命令需要 C 语言编程能力，且需要注意与 Bash 版本的兼容性。实际场景中较少使用。

## 实现外部命令

### 外部命令的执行过程

```
用户输入命令
  ↓
Bash 在 PATH 中搜索可执行文件
  ↓
fork 创建子进程
  ↓
exec 加载可执行文件
  ↓
内核根据文件头判断类型
  ├── ELF 格式 → 直接执行
  └── shebang → 使用指定解释器执行
  ↓
子进程执行完毕，退出状态传回父进程
```

### C 语言实现外部命令示例

```c
// myls.c - 简化版 ls
#include <stdio.h>
#include <stdlib.h>
#include <dirent.h>
#include <sys/stat.h>
#include <string.h>
#include <pwd.h>
#include <grp.h>
#include <time.h>

void print_file_info(const char *name, const char *path) {
    struct stat st;
    char fullpath[1024];
    snprintf(fullpath, sizeof(fullpath), "%s/%s", path, name);

    if (lstat(fullpath, &st) < 0) {
        perror(name);
        return;
    }

    // 文件类型
    char type = '?';
    if (S_ISREG(st.st_mode))  type = '-';
    else if (S_ISDIR(st.st_mode))  type = 'd';
    else if (S_ISLNK(st.st_mode))  type = 'l';

    // 权限
    char perms[10] = "---------";
    if (st.st_mode & S_IRUSR) perms[0] = 'r';
    if (st.st_mode & S_IWUSR) perms[1] = 'w';
    if (st.st_mode & S_IXUSR) perms[2] = 'x';
    if (st.st_mode & S_IRGRP) perms[3] = 'r';
    if (st.st_mode & S_IWGRP) perms[4] = 'w';
    if (st.st_mode & S_IXGRP) perms[5] = 'x';
    if (st.st_mode & S_IROTH) perms[6] = 'r';
    if (st.st_mode & S_IWOTH) perms[7] = 'w';
    if (st.st_mode & S_IXOTH) perms[8] = 'x';

    // 所有者和组
    struct passwd *pw = getpwuid(st.st_uid);
    struct group *gr = getgrgid(st.st_gid);

    // 时间
    char timebuf[64];
    strftime(timebuf, sizeof(timebuf), "%b %d %H:%M",
             localtime(&st.st_mtime));

    printf("%c%s %3lu %s %s %8ld %s %s\n",
           type, perms, (unsigned long)st.st_nlink,
           pw ? pw->pw_name : "unknown",
           gr ? gr->gr_name : "unknown",
           (long)st.st_size, timebuf, name);
}

int main(int argc, char *argv[]) {
    const char *path = (argc > 1) ? argv[1] : ".";
    DIR *dir = opendir(path);
    if (!dir) {
        perror(path);
        return 1;
    }

    struct dirent *entry;
    while ((entry = readdir(dir)) != NULL) {
        if (entry->d_name[0] == '.') continue;  // 跳过隐藏文件
        print_file_info(entry->d_name, path);
    }

    closedir(dir);
    return 0;
}
```

**编译与安装**：

```bash
# 编译
gcc -o myls myls.c

# 测试
./myls /tmp

# 安装到系统路径
sudo cp myls /usr/local/bin/
```

### 参考书籍

| 书名 | 说明 |
|------|------|
| 《Advanced Programming in the UNIX Environment》（APUE） | UNIX 系统编程经典，涵盖文件 I/O、进程控制、信号等 |
| 《The Linux Programming Interface》（TLPI） | Linux 系统编程权威指南 |
| 《UNIX Systems Programming》（USP） | UNIX 系统编程入门 |

## Shell 脚本基础

### 脚本结构

```bash
#!/bin/bash
# ^^^ shebang 行，指定解释器路径
# 必须是脚本的第一行，以 #! 开头

# 脚本元信息
# Description: 示例脚本
# Author: xxx
# Date: 2024-01-01

# 严格模式
set -euo pipefail

# 变量定义
readonly SCRIPT_NAME=$(basename "$0")
readonly SCRIPT_DIR=$(cd "$(dirname "$0")" && pwd)

# 函数定义
usage() {
    cat <<EOF
Usage: $SCRIPT_NAME [OPTIONS] <arg>

Options:
  -h, --help     Show this help message
  -v, --version  Show version
EOF
}

# 主逻辑
main() {
    # 处理参数
    while [[ $# -gt 0 ]]; do
        case "$1" in
            -h|--help) usage; exit 0 ;;
            *) echo "Unknown option: $1" >&2; exit 2 ;;
        esac
        shift
    done
}

main "$@"
```

### echo vs printf

| 特性 | `echo` | `printf` |
|------|--------|----------|
| 可移植性 | 内置行为不一致（不同 Shell/系统） | POSIX 标准，行为一致 |
| 格式化 | 不支持 | 支持 `%s`, `%d`, `%f` 等 |
| 转义序列 | 部分支持（`-e` 选项，非便携） | 原生支持 `\n`, `\t` 等 |
| 自动换行 | 默认添加 | 不自动换行（需手动 `\n`） |
| 推荐场景 | 简单输出 | 格式化输出、脚本中输出 |

```bash
# echo 简单输出
echo "Hello, World!"

# printf 格式化输出
printf "Name: %-10s Age: %3d\n" "Alice" 25
printf "Name: %-10s Age: %3d\n" "Bob" 30

# printf 不自动换行
printf "Processing..."   # 无换行
printf "Done\n"          # 手动换行
```

> **建议**：在脚本中使用 `printf` 替代 `echo`，行为更可预测。

### 脚本执行方式对比

| 执行方式 | Shell 类型 | 变量影响 | 初始化 | 退出方式 |
|---------|-----------|---------|--------|---------|
| `source script.sh` | 当前 Shell | 影响当前环境 | 无 | `return N` |
| `. script.sh` | 当前 Shell | 影响当前环境 | 无 | `return N` |
| `./script.sh` | child-shell | 无影响 | 加载 BASH_ENV | `exit N` |
| `bash script.sh` | child-shell | 无影响 | 加载 BASH_ENV | `exit N` |
| `exec ./script.sh` | 替换当前 Shell | 当前 Shell 消失 | 加载 BASH_ENV | `exit N` |

### 初始化脚本与运行级别

**System V init 系统**：

```
/etc/init.d/          ← 服务脚本存放目录
/etc/rc0.d/           ← 运行级别 0（关机）的符号链接
/etc/rc1.d/           ← 运行级别 1（单用户）的符号链接
/etc/rc2.d/           ← 运行级别 2（多用户，无 NFS）
/etc/rc3.d/           ← 运行级别 3（完整多用户）
/etc/rc5.d/           ← 运行级别 5（图形界面）
/etc/rc6.d/           ← 运行级别 6（重启）
```

**符号链接命名规则**：

| 前缀 | 含义 | 执行时机 |
|------|------|---------|
| `SXX服务名` | Start | 进入该运行级别时启动（XX 为两位数字，表示启动顺序） |
| `KXX服务名` | Kill | 离开该运行级别时停止 |

```bash
# 示例
ls /etc/rc3.d/
S10network  S55sshd  S80httpd  K50xinetd

# 创建符号链接
sudo ln -s /etc/init.d/nginx /etc/rc3.d/S80nginx
sudo ln -s /etc/init.d/nginx /etc/rc0.d/K20nginx
```

**systemd 系统**（现代 Linux 发行版）：

```bash
# 服务单元文件位置
/etc/systemd/system/
/usr/lib/systemd/system/

# 常用命令
systemctl start nginx      # 启动
systemctl stop nginx       # 停止
systemctl enable nginx     # 开机自启
systemctl disable nginx    # 取消自启
systemctl status nginx     # 查看状态
systemctl list-units       # 列出所有单元
```

## 别名详解

### alias 命令

**语法**：

```bash
alias [名称[=值] ...]
alias -p
```

| 参数 | 说明 |
|------|------|
| `名称=值` | 定义别名 |
| `-p` | 以可重用格式列出所有别名 |
| （无参数） | 列出所有别名 |

**示例**：

```bash
# 定义别名
alias ll='ls -la'
alias gs='git status'
alias grep='grep --color=auto'

# 列出所有别名
alias

# 查看特定别名
alias ll
# alias ll='ls -la'
```

### unalias 命令

**语法**：

```bash
unalias [-a] 名称 [名称 ...]
```

| 参数 | 说明 |
|------|------|
| `-a` | 删除所有别名 |
| `名称` | 删除指定别名 |

**示例**：

```bash
# 删除指定别名
unalias ll

# 删除所有别名
unalias -a
```

### 别名的局限性

| 局限 | 说明 |
|------|------|
| 不支持参数 | 别名只是简单的文本替换，无法像函数那样接受参数 |
| 脚本中默认禁用 | 非交互式 Shell 默认不展开别名 |
| 递归风险 | 别名展开可能产生递归（Bash 会检测并阻止） |
| 不可嵌套 | 别名定义中引用其他别名时，行为可能不符合预期 |

> **建议**：对于需要参数的复杂命令，使用 Shell 函数替代别名。

## 最佳实践总结

| 场景 | 推荐做法 | 说明 |
|------|---------|------|
| 简单命令缩写 | 使用别名 | `alias ll='ls -la'` |
| 需要参数的命令 | 使用 Shell 函数 | `myfunc() { ... }` |
| 高性能命令 | 考虑内置命令或 C 外部命令 | 避免 fork+exec 开销 |
| 调试命令来源 | 使用 `type -a` | 查看所有匹配项 |
| 绕过别名/函数 | 使用 `command` 或 `\` 前缀 | `\ls` 或 `command ls` |
| 脚本输出 | 使用 `printf` | 行为可预测，可移植 |
| 禁用危险内置命令 | 使用 `enable -n` | 临时禁用 |
| 持久化自定义函数 | 写入 `~/.bashrc` 或独立文件 | `source` 加载 |

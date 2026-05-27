# Linux 初始化

## 文本字符界面详解

在文本字符界面终端，读者必须了解终端提示符`[root@localhost ~]#`表示的含义，从而顺利地往下学习。

- `~root`：当前登录的用户名。
- `localhost`：当前系统的计算机名。
- `~`：当前的工作路径位于用户的宿主目录下。
- `#`：当前登录用户是管理员用户。如果显示的是`$`符号，则表示当前用户为普通用户。

## 电源管理

### 注销命令

注销命令如下：

```shell
# 使用 exit 命令注销
exit

# 使用 logout 命令注销
logout
```

需要注意的是，注销命令只能在文本字符模式下看出效果，在图形化界面的终端控制台中输入以上命令时会直接退出终端。在图形化界面中可以直接单击“关机”按钮进行注销操作。

### 重启命令

在 Linux 系统中实现重启的命令如下：

```shell
# 使用 shutdown 命令
shutdown -r now

# 使用 reboot 命令
reboot

# 使用 init 命令
init 6
```

### 关机命令

常用的关机命令如下：

```shell
# 使用 shutdown 命令
shutdown -h now

# 使用 poweroff 命令
poweroff

# 使用 init 命令
init 0

# 使用 halt 命令，关机时同时关闭电源
halt -p
```

下面简单介绍以上命令的差异：

- `shutdown`命令在关机的同时也会关闭电源，但是只有 root 用户才可以执行这个命令，普通用户需要 root 用户进行授权后才可以执行。
- halt 命令在关机时不会关闭电源，如果加上“-p”选项就可以在关机的同时也关闭电源。
- poweroff 命令在关机时也会关闭电源，其实“halt -p”就等同于 poweroff 命令。
- init 命令是 CentOS 6 以下发行版本使用的，init 0 表示关机，init 6 表示重启，在 CentOS 7 中可以兼容这种命令。

### 定时关机命令

按预定时间关闭系统

```shell
shutdown -h hours:minutes &
```

取消按预定时间关闭系统

```shell
shutdown -c
```

## 查看系统信息

```shell
# cat /etc/os-release

NAME="Rocky Linux"
VERSION="9.5 (Blue Onyx)"
ID="rocky"
ID_LIKE="rhel centos fedora"
VERSION_ID="9.5"
PLATFORM_ID="platform:el9"
PRETTY_NAME="Rocky Linux 9.5 (Blue Onyx)"
ANSI_COLOR="0;32"
LOGO="fedora-logo-icon"
CPE_NAME="cpe:/o:rocky:rocky:9::baseos"
HOME_URL="https://rockylinux.org/"
VENDOR_NAME="RESF"
VENDOR_URL="https://resf.org/"
BUG_REPORT_URL="https://bugs.rockylinux.org/"
SUPPORT_END="2032-05-31"
ROCKY_SUPPORT_PRODUCT="Rocky-Linux-9"
ROCKY_SUPPORT_PRODUCT_VERSION="9.5"
REDHAT_SUPPORT_PRODUCT="Rocky Linux"
REDHAT_SUPPORT_PRODUCT_VERSION="9.5"
```

## 查看系统内核版本号和发行版本号

Linux 系统的版本号分为内核版本和发行版本，该如何查看已经安装好的 Linux 系统的这两个版本号呢？

### 1.查看内核版本号

查看系统的内核版本号可以通过命令行或者直接查看文件的方式来完成，具体命令如下：

```shell
# 直接查看文件内容
# cat /proc/version
Linux version 5.14.0-503.35.1.el9_5.x86_64 (mockbuild@iad1-prod-build001.bld.equ.rockylinux.org) (gcc (GCC) 11.5.0 20240719 (Red Hat 11.5.0-5), GNU ld version 2.35.2-54.el9) #1 SMP PREEMPT_DYNAMIC Thu Apr 3 12:12:16 UTC 2025

# 通过命令行方式进行查看
# uname -a
Linux localhost.localdomain 5.14.0-503.35.1.el9_5.x86_64 #1 SMP PREEMPT_DYNAMIC Thu Apr 3 12:12:16 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
```

### 2.查看发行的版本号

查看系统的发行版本号，可以直接查看文件，具体命令如下：

```shell
# cat /etc/redhat-release
Rocky Linux release 9.5 (Blue Onyx)
```

## 查看系统的位数

操作系统分为 32 位和 64 位两种，如果不清楚自己使用的系统的位数，可以通过以下命令进行查看：

```shell
# getconf LONG_BIT
64
```

## UI 切换

### 从图形化界面切换到文本字符模式

从图形化界面切换到文本字符模式的命令如下：

```shell
# 使用 init 命令，适用 CentOS 的所有版本
init 3

# 使用 systemd 命令。只能适用于 Cent0S 7 以上的版本
systemctl isolate multi-user. target
```

需要注意的是，切换成功的前提是已经在图形化界面中打开了终端控制台。

### 从文本字符模式切换到图形化界面

从文本字符模式切换到图形化界面的命令如下：

```shell
# 使用 init 命令，适用于 CentOS 的所有版本
init 5

# 使用 systemd 命令，只适用于 CentOS 7 以上的版本
systemctl isolate graphical.target
```

需要注意的是，当安装系统时如果没有选择最小桌面安装，那么输入以上命令时虽然不会报错，但是也无法切换到图形化界面。

# DEB 安装

DEB 软件包是以".deb"结尾的文件，属于 Debian 系统（含 Debian 和 Ubuntu 等发行版本中）的专属软件包，它和 RPM 包非常类似，二者可以通过工具相互转换。DEB 软件包通过 dpkg（Debian Package，Debian 包管理器）或 apt 命令进行管理和维护。

相比较而言，RPM 安装包应用更为广泛，基本已成为 Linux 系统中软件安装包事实上的标准。但在 Debian 系发行版中，DEB 包管理是最常用的软件安装方式。

## dpkg 命令

dpkg 是 Debian 系发行版中底层的包管理工具，主要用于安装、卸载和管理 .deb 软件包。

### 安装软件包

使用 dpkg 命令安装 DEB 软件包：

```shell
dpkg -i 软件包名.deb
```

例如，安装一个名为 nginx 的 DEB 包：

```shell
[root@localhost ~]# dpkg -i nginx_1.18.0-2_amd64.deb
```

### 卸载软件包

使用 dpkg 命令卸载已安装的软件包：

```shell
dpkg -r 软件包名
```

例如，卸载 nginx 软件包：

```shell
[root@localhost ~]# dpkg -r nginx
```

> 注意：`-r` 选项会保留配置文件。如果需要完全删除（包括配置文件），应使用 `-P` 选项：
> ```shell
> dpkg -P nginx
> ```

### 查询已安装的软件包

```shell
# 查询系统中是否已安装指定软件包
dpkg -l | grep 软件包名

# 查看已安装软件包的详细信息
dpkg -s 软件包名

# 查看软件包安装的文件列表
dpkg -L 软件包名
```

### 常用 dpkg 命令选项

| 选项 | 作用 |
|------|------|
| `-i` | 安装软件包 |
| `-r` | 卸载软件包（保留配置文件） |
| `-P` | 完全卸载软件包（删除配置文件） |
| `-l` | 列出已安装的软件包 |
| `-s` | 显示已安装软件包的详细信息 |
| `-L` | 列出软件包安装的文件列表 |
| `-S` | 查询某个文件属于哪个软件包 |

## apt 命令

apt（Advanced Package Tool）是 Debian 系发行版中高级的包管理工具，它在 dpkg 的基础上提供了更强大的功能，能够自动处理软件包之间的依赖关系。

> 在较早的 Debian/Ubuntu 版本中，使用 apt-get 命令。从 Debian 8 和 Ubuntu 16.04 开始，推荐使用更简洁的 apt 命令。

### 更新软件包列表

在执行安装操作前，建议先更新软件包列表：

```shell
[root@localhost ~]# apt update
```

### 安装软件包

```shell
[root@localhost ~]# apt install 软件包名
```

例如，安装 nginx：

```shell
[root@localhost ~]# apt install nginx
```

apt 会自动解决并安装所有依赖的软件包。

### 卸载软件包

```shell
# 卸载软件包（保留配置文件）
[root@localhost ~]# apt remove 软件包名

# 完全卸载软件包（包括配置文件）
[root@localhost ~]# apt purge 软件包名
```

### 升级软件包

```shell
# 升级所有已安装的软件包
[root@localhost ~]# apt upgrade

# 完整系统升级（可能包含内核升级，会删除旧包）
[root@localhost ~]# apt full-upgrade
```

### 搜索软件包

```shell
[root@localhost ~]# apt search 关键词
```

例如，搜索与 nginx 相关的软件包：

```shell
[root@localhost ~]# apt search nginx
```

### 显示软件包信息

```shell
[root@localhost ~]# apt show 软件包名
```

### 清理缓存

```shell
# 删除已下载的软件包文件（/var/cache/apt/archives/）
[root@localhost ~]# apt clean

# 删除不再需要的依赖包
[root@localhost ~]# apt autoremove
```

### 常用 apt 命令汇总

| 命令 | 作用 |
|------|------|
| `apt update` | 更新软件包列表 |
| `apt install 包名` | 安装软件包 |
| `apt remove 包名` | 卸载软件包（保留配置） |
| `apt purge 包名` | 完全卸载软件包 |
| `apt upgrade` | 升级所有软件包 |
| `apt full-upgrade` | 完整系统升级 |
| `apt search 关键词` | 搜索软件包 |
| `apt show 包名` | 显示软件包详细信息 |
| `apt list` | 列出所有可用或已安装的软件包 |
| `apt autoremove` | 删除不再需要的依赖包 |
| `apt clean` | 清理已下载的软件包缓存 |

## apt 与 dpkg 的关系

- **dpkg** 是底层的包管理工具，直接操作 .deb 文件，但不处理依赖关系
- **apt** 是高级包管理工具，基于 dpkg 构建，能够自动解决依赖关系
- 在生产环境中，**推荐使用 apt** 进行软件包管理，因为它能自动处理复杂的依赖问题

## 与 RPM/YUM 的对比

| 特性 | DEB/apt | RPM/YUM |
|------|---------|---------|
| 适用发行版 | Debian, Ubuntu | RHEL, CentOS, Fedora |
| 包格式 | .deb | .rpm |
| 底层工具 | dpkg | rpm |
| 高级工具 | apt | yum/dnf |
| 依赖处理 | 自动 | 自动 |
| 软件包命名 | `软件名_版本-发布号_架构.deb` | `软件名-版本-发布号.架构.rpm` |

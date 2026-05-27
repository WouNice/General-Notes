# DNF 安装

DNF（Dandified YUM）是新一代的 RPM 软件包管理器，旨在替代传统的 YUM 工具。DNF 使用 RPM、libsolv 和 hawkey 库进行包管理操作，从 Fedora 22 开始默认使用 DNF，RHEL 8 和 CentOS 8 也采用 DNF 作为默认的包管理工具。

## DNF 与 YUM 的关系

DNF 是 YUM 的下一代版本，二者在命令行使用上高度兼容，但 DNF 在内部实现上有显著改进：

- **更高的性能**：DNF 使用 C 语言编写的 libsolv 库进行依赖解析，速度更快
- **更低的内存占用**：优化了内存使用效率
- **更好的依赖处理**：使用 SAT 求解器处理复杂的依赖关系
- **支持模块化内容**：支持 Fedora/RHEL 的模块化软件包管理
- **稳定的 API**：提供设计良好的 API，便于与其他工具集成

> 在 RHEL 8/CentOS 8 及以后的版本中，`yum` 命令实际上是 DNF 的符号链接，执行 `yum` 即调用 DNF。因此在这类系统中，使用 `yum` 或 `dnf` 命令效果相同。

## DNF 配置文件

DNF 的配置文件位于 `/etc/dnf/dnf.conf`，仓库配置文件位于 `/etc/yum.repos.d/` 目录下（与 YUM 共享）。

```shell
# 查看 DNF 配置文件
[root@localhost ~]# cat /etc/dnf/dnf.conf
```

## 常用 DNF 命令

### 软件包管理

```shell
# 安装软件包
[root@localhost ~]# dnf install 软件包名

# 卸载软件包
[root@localhost ~]# dnf remove 软件包名

# 重新安装软件包
[root@localhost ~]# dnf reinstall 软件包名

# 升级单个软件包
[root@localhost ~]# dnf update 软件包名

# 升级所有软件包
[root@localhost ~]# dnf update

# 检查可用的更新
[root@localhost ~]# dnf check-update
```

### 查询操作

```shell
# 搜索软件包
[root@localhost ~]# dnf search 关键词

# 显示软件包详细信息
[root@localhost ~]# dnf info 软件包名

# 列出所有已安装的软件包
[root@localhost ~]# dnf list installed

# 列出所有可用的软件包
[root@localhost ~]# dnf list available

# 列出软件包组
[root@localhost ~]# dnf group list

# 显示某个文件属于哪个软件包
[root@localhost ~]# dnf provides /usr/bin/命令名
```

### 仓库管理

```shell
# 列出所有已配置的仓库
[root@localhost ~]# dnf repolist

# 列出所有仓库（包括禁用的）
[root@localhost ~]# dnf repolist all

# 启用仓库
[root@localhost ~]# dnf config-manager --set-enabled 仓库名

# 禁用仓库
[root@localhost ~]# dnf config-manager --set-disabled 仓库名
```

### 历史记录与回滚

DNF 提供了强大的历史记录功能，可以查看和撤销之前的操作：

```shell
# 查看操作历史
[root@localhost ~]# dnf history

# 查看某次操作的详细信息
[root@localhost ~]# dnf history info 事务ID

# 撤销某次操作
[root@localhost ~]# dnf history undo 事务ID

# 重做某次操作
[root@localhost ~]# dnf history redo 事务ID

# 回滚到某次操作之前的状态
[root@localhost ~]# dnf history rollback 事务ID
```

### 清理与维护

```shell
# 清理缓存
[root@localhost ~]# dnf clean all

# 删除不再需要的依赖包
[root@localhost ~]# dnf autoremove

# 验证已安装软件包的完整性
[root@localhost ~]# dnf verify
```

### 模块化操作

DNF 支持 Fedora/RHEL 的模块化内容管理：

```shell
# 列出可用的模块流
[root@localhost ~]# dnf module list

# 启用模块流
[root@localhost ~]# dnf module enable 模块名:流名

# 安装模块
[root@localhost ~]# dnf module install 模块名:流名/配置文件

# 禁用模块
[root@localhost ~]# dnf module disable 模块名

# 重置模块（恢复到默认状态）
[root@localhost ~]# dnf module reset 模块名
```

## DNF 与 YUM 命令对比

| 功能 | DNF 命令 | YUM 命令 |
|------|----------|----------|
| 安装软件包 | `dnf install` | `yum install` |
| 卸载软件包 | `dnf remove` | `yum remove` |
| 升级软件包 | `dnf update` | `yum update` |
| 搜索软件包 | `dnf search` | `yum search` |
| 显示软件包信息 | `dnf info` | `yum info` |
| 列出仓库 | `dnf repolist` | `yum repolist` |
| 清理缓存 | `dnf clean all` | `yum clean all` |
| 查看历史 | `dnf history` | `yum history` |
| 自动删除依赖 | `dnf autoremove` | `yum autoremove` |

## 常用 DNF 命令速查表

| 命令 | 作用 |
|------|------|
| `dnf install 包名` | 安装软件包 |
| `dnf remove 包名` | 卸载软件包 |
| `dnf reinstall 包名` | 重新安装软件包 |
| `dnf update` | 升级所有软件包 |
| `dnf update 包名` | 升级指定软件包 |
| `dnf check-update` | 检查可用更新 |
| `dnf search 关键词` | 搜索软件包 |
| `dnf info 包名` | 显示软件包信息 |
| `dnf list installed` | 列出已安装的软件包 |
| `dnf list available` | 列出可用的软件包 |
| `dnf repolist` | 列出已启用的仓库 |
| `dnf repolist all` | 列出所有仓库 |
| `dnf history` | 查看操作历史 |
| `dnf history undo ID` | 撤销指定操作 |
| `dnf clean all` | 清理所有缓存 |
| `dnf autoremove` | 删除不需要的依赖 |
| `dnf provides 文件` | 查找提供某文件的软件包 |
| `dnf group list` | 列出软件包组 |
| `dnf group install 组名` | 安装软件包组 |

## 从 YUM 迁移到 DNF

对于习惯使用 YUM 的用户，迁移到 DNF 非常简单：

1. **命令兼容**：绝大多数 YUM 命令在 DNF 中同样适用，只需将 `yum` 替换为 `dnf`
2. **配置文件兼容**：仓库配置文件（.repo）格式相同，位于 `/etc/yum.repos.d/`
3. **缓存位置**：DNF 缓存位于 `/var/cache/dnf/`

在 RHEL 8/CentOS 8 及以后版本中，可以直接继续使用 `yum` 命令，系统会自动调用 DNF 处理。

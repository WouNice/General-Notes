# YUM 安装

## yum 的配置文件

在执行 yum 命令时需要查看下 yum 源中的仓库地址是否可用，仓库地址可以是目录也可以是链接地址，存放着软件包和索引文件。

仓库可以分为本地仓库和在线仓库

-   本地仓库就是将所有软件包存放到本地或局域网内的服务器上，不需要外网权限，一般可以通过镜像文件中的软件包制作本地 yum 源仓库；
-   在线仓库就是直接通过一个网络链接地址在互联网中搜索并下载软件包，网络上有很多开源的镜像网站可以作为在线的 yum 源仓库地址。

CentOS 系统中默认的在线软件仓库是可以直接使用的。

查询仓库地址有哪些的命令如下：

```shell
yum repolist all
```

### 配置文件说明

yum 配置文件主要分为两部分：main 和 repository。

-   main 部分定义了全局配置选项，这些选项针对每一个 repository 都生效，位于/etc/yum.conf 文件中，整个 YUM 配置文件只能有一个 main。
-   repository 部分定义每个 repo 源（可以理解为软件包的仓库地址）的具体配置，可以存在一个或多个，文件都是以`.repo`为后缀名，位于/etc/yum.repos.d/目录下，yum 命令会从该目录逐步查询可用的软件仓库。

（1）查看/etc/yum.conf 配置文件，main 部分定义了全局配置选项。

```shell
# 查看/etc/yum.conf 配置文件
[root@localhost /]# cat /etc/yum.conf
[main]
cachedir=/var/cache/yum/$basearch/$releasever
keepcache=0
debuglevel=2
logfile=/var/log/yum.log
exactarch=1
obsoletes=1
gpgcheck=1
plugins=1
installonly_limit=5
bugtracker_url=http://bugs.centos.org/set_project.php?project_id=23&ref
=http://bugs.centos.org/bug_report_page.php?category=yum
distroverpkg=centos-release
```

配置文件中参数的详细说明如下：

- [main]：定义全局配置选项。
- cachedir：yum 下载 RPM 包的缓存目录，`$basearch`表示系统架构的硬件平台，比如 i386、x86_64 等；$releasever 定义发行版本，比如 6、7、8 等数字。
- keepcache：是否缓存目录下的 RPM 包，0 表示不保存，安装完成后直接删除（默认）；1 表示保存，安装完成后保留。
- debuglevel：错误日志级别（0～10），默认值为 2，表示只保留安装和删除日志。
- logfile：yum 日志文件目录。
- exactarch：是否允许更新不同硬件（CPU 体系）平台的 RPM 包。1 表示不允许（默认），意思就是 i386 不允许更新 i686 的 RPM 包；0 表示允许。
- obsoletes：是否允许更新旧版本的 RPM 包，1 表示允许（默认），0 表示不允许。
- gpgcheck：是否校验 GPG（GNU Private Guard）密钥签名方式，确定 RPM 包来源的安全性。1 表示校验（默认），无该参数也表示校验；0 表示不校验。
- plugins：是否启用插件，1 表示启用（默认），0 表示不启用。
- installonly_limit：允许保留内核包的个数。
- bugtracker_url：追踪 bug 路径。
- distroverpkg：执行软件包的发行版本。

除上述参数外，也可以添加一些其他参数，具体如下：

- pkgpolicy：软件包策略，对应两个值 newest 和 last。其中 newest 表示存在多个 yum 源时选择最新版本的 yum 源；last 表示将以 yum 源服务器 ID 的字母表排序，选择在名次排在最后面的服务器上安装软件包。
- tolerent：yum 是否显示命令行执行时发生的软件包相关的错误，对应两个值 0 和 1。其中 1 表示不显示出现的错误信息，0 表示显示错误信息。
- retries：网络连接错误后的重试次数，0 表示无限重试。
- exclude：排除指定软件包，多个 RPM 包之间使用空格分离，也可以使用通配符。
- reposdir：指定 repo 源的文件位置。

（2）查看/etc/yum.repos.d 目录下的.repo 源配置文件，相关信息如下：

```shell
# 查看/etc/yum.repos.d/目录下的所有文件
[root@localhost /]# ls /etc/yum.repos.d/
CentOS-Base.repo  CentOS-CR.repo  CentOS-Debuginfo.repo  CentOS-fasttrack.
repo  CentOS-Media.repo  CentOS-Sources.repo  CentOS-Vault.repo

# 查看 CentOS-Base.repo 文件的内容
[root@localhost ~]# cat /etc/yum.repos.d/CentOS-Base.repo
……
# additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$base
arch&repo=centosplus&infra=$infra

# baseurl=http://mirror.centos.org/centos/$releasever/centosplus/$basearch/

gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

`*.repo`配置文件中参数的详细说明如下：

-   [centosplus]：代表容器的名字。中括号一定要存在，里面的名称可以随意起，但不能有两个相同的容器名称，否则 yum 会不知道去哪里寻找容器的相关软件列表文件。
-   name：只是说明一下这个容器的意义而已，描述语，可有可无。
-   mirrorlist：列出这个容器可以使用的镜像站点，如果不想使用可以注释掉这一行。
-   baseurl：这个最重要，因为后面接的就是容器的实际网址。mirrorlist 是由 yum 程序自行去找镜像站点，baseurl 则是指定固定的一个容器网址。
-   gpgcheck：指定是否需要查阅 RPM 文件内的数字证书。
-   enabled：是否启动容器仓库，默认值为 1，要关闭这个容器可以设置 enable=0。默认情况下该参数是省略的，代表整个容器仓库是启用状态。
-   gpgkey：数字证书的公钥文件所在位置，使用默认值即可。

### yum 仓库地址格式

一般来讲在配置文件中 baseurl 参数用来填写仓库地址，手动添加 yum 源时仓库地址可以通过以下几种方式填写。

- 使用 ftp 服务器地址，写法是 ftp://域名或 IP 地址。
- 使用网络域名地址，写法是 http://域名或者 https://域名。
- 使用网络文件系统，写法是 nfs://域名或 IP 地址。
- 使用本地目录地址，写法是 file:///路径名。一般用于将本地镜像文件的软件包制作成本地仓库时使用。

```shell
# 将镜像文件中的软件包制作成本地仓库
# 在光驱中添加镜像文件，然后将光驱设备挂载到/mnt/a 下
[root@localhost /]# mount /dev/cdrom /mnt/a
mount: /dev/sr0 is write-protected，mounting read-only
[root@localhost /]# cd /mnt/a
[root@localhost a]# ls
CentOS_BuildTag  EFI  EULA  GPL  images  isolinux  LiveOS  Packages
repodata  RPM-GPG-KEY-CentOS-7  RPM-GPG-KEY-CentOS-Testing-7  TRANS.TBL

# 为方便直观地查看效果，可以先将/etc/yum.repos.d 目录下的所有 yum 源剪切备份到一个目录中
[root@localhost /]# cd /etc/yum.repos.d/
[root@localhost yum.repos.d]# mkdir bak
[root@localhost yum.repos.d]# ls
bak  CentOS-Base.repo  CentOS-CR.repo  CentOS-Debuginfo.repo  CentOS-fasttrack.
repo  CentOS-Media.repo  CentOS-Sources.repo  CentOS-Vault.repo
[root@localhost yum.repos.d]# mv *.repo bak
[root@localhost yum.repos.d]# ls
bak

# 使用 vi 编辑器编写一个本地 yum 源的配置文件 local.repo，内容如下
[root@localhost yum.repos.d]# cat local.repo
[local]                                                 #仓库名称
name=CentOS7.7-ISO                              #描述语
baseurl=file:///mnt/a                   #仓库地址
gpgcheck=0                                              #不用检查数字证书
[root@localhost yum.repos.d]# yum repolist all
Loaded plugins: fastestmirror，langpacks
Loading mirror speeds from cached hostfile
repo id                         repo name                       status
local                           CentOS7.7-ISO                   enabled: 4，067
repolist: 4，067
```

## YUM 源管理

### 配置 YUM 源

参考教程：https://developer.aliyun.com/mirror/centos

采用 YUM 安装方式前，必须先配置好 YUM 源。YUM 源也称为 YUM 仓库（YUMrepository，YUM repo），其中存放了大量的 RPM 软件包，以及与软件包相关的元数据文件。这些元数据文件一般放置于特定的名为 repodata 的目录下。

设置 YUM 源需要配置定义文件，定义文件必须存放在指定的``/etc/yum.repos.d/``目录中，而且必须以``.repo``作为文件名后缀。

我们通常所用的 YUM 源主要有两种类型：一种来自网络上的服务器，另一种来自本地的系统安装光盘。例如，在 CentOS 系统的``/etc/yum.repos.d/``目录中默认已经存在很多文件名后缀为`.repo`的 YUM 源文件，以`CentOS-Base.repo`为例，这就是一个以网络上的 CentOS 服务器作为 YUM 源的配置文件。

在一些网络环境中，访问 CentOS 的官网可能会比较慢，因而推荐采用像阿里云这类的镜像站作为 YUM 源。为了避免因系统中同时存在多个 YUM 源而造成混乱，建议先将系统中默认的 YUM 源文件全部删除。

```shell
[root@localhost ~]# rm -f /etc/yum.repos.d/
```

然后，可以从 http://mirrors.aliyun.com/repo/Centos-8.repo 下载 YUM 源配置文件，并将其存放到/etc/yum.repos.d/目录中。如果 Linux 系统可以访问 Internet，那么可以直接利用 wget 命令进行下载，并用`-O`选项指定下载文件的存放位置。

```text
[root@localhost ~]# wget http://mirrors.aliyun.com/repo/Centos-8.repo -O /etc/yum.repos.d/CentOS8.repo
```

某些情况下，主机可能不方便连接外网，这时可以将系统光盘配置为 YUM 源。在 CentOS 的系统光盘中已经集成了绝大多数应用软件的 RPM 包。这些软件的版本虽然不是最新的，但非常稳定，完全可以满足我们的需求。下面介绍将 CentOS 的系统光盘配置为 YUM 源的过程。

挂载光驱。

```shell
[root@localhost ~]# mount /dev/cdrom /mnt/cdrom/
```

查看光盘的目录结构，所有的 RPM 软件包都存放在`/mnt/cdrom/Packages/`目录中，但在设置 YUM 源时，不能将这个目录指定为 YUM 源路径。这是由于按照规定，只能将存放元数据文件的 repodata 目录所在的位置指定为 YUM 源路径（/mnt/cdrom）。

下面配置一个名为`dvd.repo`的 YUM 源定义文件。

```text
[root@localhost ~]# vim /etc/yum.repos.d/dvd.repo
[dvd]
name=CentOS dvd
baseurl=file:///mnt/cdrom/
enabled=1
gpgcheck=0
```

注意，文件中`=`的左右两侧不要留有空格。

文件中各行的含义如下：

①[dvd]：YUM 源的名称。

由于系统允许同时配置多个 YUM 源，因此这个名称在整个系统中必须是唯一的。名称的具体内容可自由定义。

②name：对 YUM 源的描述。

这部分内容可由用户自由定义。

③baseurl：指定 YUM 源的访问路径。

这是整个定义文件中最重要的一行，访问路径可以有多种不同的表示方法。

- 指向网络中的 Web 服务器：baserul=http://…。
- 指向网络中的 FTP 服务器：baserul=ftp://…。
- 指向本地的某个目录：baserul=file://…。

`baseurl=file:///mnt/cdrom/`表示访问路径指向的是本地的`/mnt/cdrom/`目录。

在同一个 YUM 源定义文件中可以设置多个 baseurl，即可以指定多个 YUM 源。在安装软件时会从这些 YUM 源中自动选择最新版本。如果版本都一样，那么就选择网络开销最小的。

④enabled：是否启用当前的 YUM 源。

`1`表示启用，`0`表示禁用。如果文件中没有这一行，则系统默认为 1。

⑤gpgcheck：是否检查 RPM 包的来源合法性。

我们所使用的软件包主要是由 CentOS 组织提供的官方 RPM 包，另外，某些组织或个人也可以制作发布第三方的 RPM 包，但是在生产环境中为保证系统的可靠性，建议尽量不要使用第三方的 RPM 包。

为辨别软件包的来源并防止软件包被篡改，CentOS 会对发布的官方软件包提取消息摘要并用私钥进行数字签名，将公钥放置在已经安装好的 CentOS 系统以及系统安装光盘中，这样在安装 RPM 包时就可以先检查数字签名并验证消息摘要，然后只允许检查通过的 RPM 包继续安装。

公钥的文件名为`RPM-GPG-KEY-CentOS-7`，在 CentOS 系统中的存放路径是`/etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7`，在系统安装光盘中的存放路径是`/mnt/cdrom/RPM-GPG-KEY-CentOS-7`。

将 YUM 源定义文件中的 gpgcheck 项设为`1`表示检查 RPM 包的数字签名，设为`0`则表示不检查。如果将 gpgcheck 设为 1，那么在 YUM 源定义文件中必须再添加一个`gpgkey`行，以指定公钥的存放位置。

```text
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

如果将 gpgcheck 项设为 0，那么无须检查数字签名，`gpgkey`行也就不必设置。在学习或实验环境中，

在学习或实验环境中，可以将 gpgcheck 设为`0`，以简化操作。在生产环境中，为了保证安全性，建议将 gpgcheck 项设置为`1`。

### 检测 YUM 源

YUM 源设置好之后，可以执行`yum list`命令进行检测。该命令可以列出系统中已经安装的以及 YUM 源中尚未安装的所有软件包，其中名字前面带有`@`符号的是已经安装过的软件包。如果执行`yum list`命令后可以列出所有软件包，则证明 YUM 源配置正确。

`yum list`命令也可用于查询 YUM 源中是否存在指定的软件包以及软件包版本。

例如，查询 YUM 源中是否存在名为 vsftpd 的软件包。

```text
[root@localhost ~]# yum list vsftpd
上次元数据过期检查：3:51:29 前，执行于 2022 年 03 月 26 日 星期六 14 时 24 分 24 秒。
可安装的软件包
vsftpd.x86_64                       3.0.3-49.el9                       appstream
```

yum list 命令支持使用通配符，例如查询 YUM 源中所有名称中含有 ftp 的软件包。如果在安装软件时忘记了软件包的具体名称，就可以通过这种方式进行查询。

```shell
[root@localhost ~]# yum list *ftp*
上次元数据过期检查：3:53:48 前，执行于 2022 年 03 月 26 日 星期六 14 时 24 分 24 秒。
可安装的软件包
ftp.x86_64                          0.17-89.el9                             appstream
lftp.i686                           4.9.2-4.el9                             appstream
lftp.x86_64                                          4.9.2-4.el9                             appstream
python3-requests-ftp.noarch                          0.3.1-23.el9                            appstream
syslinux-tftpboot.noarch                             6.04-0.19.el9                           appstream
tftp.x86_64                                          5.2-35.el9                              appstream
tftp-server.x86_64                                   5.2-35.el9                              appstream
vsftpd.x86_64                                        3.0.3-49.el9                            appstream
```

也可以通过下面的方式进行查询。

```text
[root@localhost ~]# yum list | grep ftp
ftp.x86_64                                           0.17-89.el9                         appstream
lftp.i686                                            4.9.2-4.el9                         appstream
lftp.x86_64                                          4.9.2-4.el9                         appstream
python3-requests-ftp.noarch                          0.3.1-23.el9                        appstream
syslinux-tftpboot.noarch                             6.04-0.19.el9                       appstream
tftp.x86_64                                          5.2-35.el9                          appstream
tftp-server.x86_64                                   5.2-35.el9                          appstream
vsftpd.x86_64                                        3.0.3-49.el9                        appstream
```

除 yum list 之外，执行 yum repolist 命令可以列出系统中所有可用的 YUM 源，也可以将其作为一种检测 YUM 源是否配置正确的方法。

```shell
[root@localhost ~]# yum repolist
仓库 id                                               仓库名称
appstream                                             CentOS Stream 9 - AppStream
baseos                                                CentOS Stream 9 - BaseOS
extras-common                                         CentOS Stream 9 - Extras packages
```

### 修改 Epel 源为国内源

EPEL (Extra Packages for Enterprise Linux)是基于 Fedora 的一个项目，为`红帽系`的操作系统提供额外的软件包，适用于 RHEL、CentOS 和 Scientific Linux。

修改 Epel 源为国内源，可以参考：https://developer.aliyun.com/mirror/epel

## 常用的 YUM 命令

| 命令                      | 作用                         |
| ------------------------- | ---------------------------- |
| yum repolist all          | 列出所有仓库                 |
| yum list all              | 列出仓库中所有软件包         |
| yum info 软件包名称       | 查看软件包信息               |
| yum install 软件包名称    | 安装软件包                   |
| yum reinstall 软件包名称  | 重新安装软件包               |
| yum update 软件包名称     | 升级软件包                   |
| yum remove 软件包名称     | 移除软件包                   |
| yum clean all             | 清除所有仓库缓存             |
| yum check-update          | 检查可更新的软件包           |
| yum grouplist             | 查看系统中已经安装的软件包组 |
| yum groupinstall 软件包组 | 安装指定的软件包组           |
| yum groupremove 软件包组  | 移除指定的软件包组           |
| yum groupinfo 软件包组    | 查询指定的软件包组信息       |

1．yum info——查看软件包的信息

执行 yum info 命令可以查看指定软件包的简要信息，如果该软件包已经安装，那么命令执行后会显示`已安装的软件包`；如果软件包尚未安装，则会显示`可安装的软件包`。

例如，查看 vsftpd 软件包的信息。从中可以查看到软件包的版本、适用平台和软件描述等信息。可以通过该命令了解一些不熟悉的软件的基本功能。

```text
[root@localhost ~]# yum info vsftpd
上次元数据过期检查：3:58:18 前，执行于 2022 年 03 月 26 日 星期六 14 时 24 分 24 秒。
可安装的软件包
名称         : vsftpd
版本         : 3.0.3
发布         : 49.el9
架构         : x86_64
大小         : 169 k
源           : vsftpd-3.0.3-49.el9.src.rpm
仓库         : appstream
概况         : Very Secure Ftp Daemon
URL          : https://security.appspot.com/vsftpd.html
协议         : GPLv2 with exceptions
描述         : vsftpd is a Very Secure FTP daemon. It was written completely from
             : scratch.
```

2．yum install——安装软件

使用 YUM 方式安装软件时，无论当前处在哪个工作目录，都会自动从 YUM 源中查找所要安装的软件包。

使用`yum install`命令安装软件，例如`yum install vsftpd`。如果软件安装正确，那么在最后将出现`完毕!`或`Complete!`的提示。

```shell
[root@localhost ~]# yum install vsftpd
```

YUM 安装会自动检查软件包之间的依赖关系，例如安装一款名为 GCC 的软件，执行`yum install gcc`之后，可以发现还要安装很多依赖包。输入`y`，按回车键，就可以将 GCC 连同依赖包全部安装了。

yum install 也支持通配符，比如在安装 PHP 时忘记了软件包的具体名称，则可以执行`yum install php*`。该命令会将 YUM 源中所有以`php`开头的软件包全部列出，从中选择需要的软件包即可。

yum install 命令可以使用`-y`选项实现自动确认，这样就无须与用户交互了。

```text
[root@localhost ~]# yum install gcc -y
```

3．yum remove——卸载软件

卸载软件可以使用`yum remove`命令，例如卸载 vsftpd。

```shell
[root@localhost ~]# yum remove vsftpd
```

需要注意的是，yum remove 在卸载一个软件包的同时会将所有依赖于该软件的其他软件包也一同卸载。例如，执行命令`yum remove cpp`，CPP 是在安装 GCC 时作为依赖包被一同安装的，因而在卸载 CPP 时会提示要将 GCC 也一同卸载。因为如果 CPP 被卸载了，那么 GCC 肯定也无法正常使用。但是这又会导致新的问题出现，比如 GCC 又是别的软件的依赖包，那么将会导致这些软件也无法正常使用。因此，如果这些被一同卸载的软件正好是其他软件或系统本身运行所需要的，就容易造成问题甚至系统崩溃，因而在使用 yum remove 命令卸载软件时一定要慎重。

4．yum clean all——清除本地缓存

YUM 会自动创建本地缓存，用来存储一些 YUM 数据，以提高 YUM 的执行效率。YUM 默认优先使用 YUM 缓存来获得软件的相关信息，在大部分情况下我们无须费心管理这些数据，但如果发现 YUM 运行不太正常，也许就是由于 YUM 缓存错误造成的，此时就可以用`yum clean all`命令清除缓存以解决问题。

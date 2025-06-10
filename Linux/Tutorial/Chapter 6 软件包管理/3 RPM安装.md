# RPM安装

rpm命令目前在Linux系统中主要用作查询，如查询系统中是否已经安装了某个软件，查询某个软件包的信息等。

## 了解RPM软件包

RPM软件包是对程序源码进行编译和封装以后形成的包文件，在软件包中会封装软件的程序、配置文件和帮助手册等组件。

使用RPM机制封装的软件包文件拥有约定俗成的命名格式，一般使用`软件名称-版本号-发布号.硬件平台.rpm`的文件名形式，如下所示：

```
vsftpd-3.0.2-22.el7.x86_64.rpm
```

以`vsftpd-3.0.2-22.el7.x86_64.rpm`软件包为例，其名称中包含以下几个部分。

- 软件名称：vsftpd。
- 版本号：3.0.2。这是vsftpd的版本。
- 发布号：22.el7。发布号指的是RPM软件包的版本。需要注意的是，RPM包通常并不是由软件的开发者所制作的，我们所安装的这些RPM包，大多是由CentOS组织封装的，因而软件包的发布号与软件的版本号是两回事，一般更新发布号主要是对RPM包存在的错误或漏洞进行修补，在软件功能上则并没有增强。el7是针对RHEL 7系统发布的软件包，它同样也适用于CentOS 7系统。
- 硬件平台：x86_64，指软件包所适用的硬件平台。`x86_64`指64位的PC架构，`i386`或`i686`等都是指32位的PC架构，`noarch`代表不区分硬件架构。按照向下兼容的原则，一般32位的软件包也适用于64位平台，反之则不可。

在CentOS的系统光盘中，含有诸多Linux系统的常用软件。进入光盘挂载目录，在Packages子目录中存放了所有的RPM软件包。可以通过执行下面的命令统计光盘中软件包的数目。

```
# ll /mnt/cdrom/Packages/ | wc -l
```

## 安装/卸载软件包

利用RPM方式安装软件包所使用的命令是`rpm –ivh`，选项的含义如下：

- -i，安装软件包；
- -v，显示安装过程；
- -h，显示安装进度，安装每进行2%就会显示一个#号。

在利用RPM方式安装软件时，需要指明软件包的路径；或者先切换到软件包所在目录，然后安装。例如，利用RPM安装vsftpd程序（在输入软件包的名字时可以用<Tab>键补全）。

```shell
[root@localhost ~]# cd /mnt/cdrom/Packages/
[root@localhost Packages]# rpm -ivh vsftpd-3.0.2-22.el7.x86_64.rpm
警告：vsftpd-3.0.2-22.el7.x86_64.rpm: 头V3 RSA/SHA256 Signature, 密钥 ID f4a80eb5: NOKEY
准备中...                     ################################# [100%]
正在升级/安装...
   1:vsftpd-3.0.2-22.el7     ################################# [100%]
```

使用`rpm –e`命令可以删除一个已经安装过的软件，如将刚才安装的vsftpd删除。

```
[root@localhost ~]# rpm -e vsftpd     #当删除成功时，没有任何提示
[root@localhost ~]# rpm -e vsftpd     #当再次删除时，会提示软件包没有安装
错误：未安装软件包 vsftpd
```

## 查询软件包

rpm命令主要用来进行软件查询，用到的相关选项是`-q`（query）。

例如，查询系统中是否已经安装了openssh-server和httpd软件。

```shell
[root@localhost ~]# rpm -q openssh-server
openssh-server-8.7p1-8.el9.x86_64
[root@localhost ~]# rpm -q httpd
未安装软件包 httpd
```

在用`rpm –q`命令进行查询时，必须指定软件的完整名称，否则将无法查询出正确结果。例如，虽然系统中默认已经安装了openssh-server，但是只输入ssh无法查询到结果。

```shell
[root@localhost ~]# rpm -q ssh
未安装软件包 ssh
```

为了更加准确地查询到我们所需要的信息，通常将`-q`选项结合其他选项一起使用。

1．`-qa`选项——查询所有已安装的软件包

例如，统计系统中已经安装的RPM软件包的个数。

```shell
[root@localhost ~]# rpm -qa | wc -l
1153
```

如果不确定要查找的软件的准确名称，或者想知道系统中是否已经安装了某个软件包，那么可以使用`–qa`选项来查询所有已安装的软件包数据。

例如，查找系统中已经安装的所有与`ssh`有关的软件包。

```shell
[root@localhost ~]# rpm -qa | grep ssh
libssh-config-0.9.6-3.el9.noarch
openssh-8.7p1-8.el9.x86_64
libssh-0.9.6-3.el9.x86_64
openssh-clients-8.7p1-8.el9.x86_64
openssh-server-8.7p1-8.el9.x86_64
```

2．`-qi`选项——查询已安装软件包的信息

不同于yum info命令，如果软件包尚未安装，则不能用rpm -qi查看。

例如，查询openssh-server软件包的信息。

```shell
[root@localhost Packages]# rpm -qi openssh-server
Name        : openssh-server
Version     : 8.7p1
Release     : 8.el9
Architecture: x86_64
Install Date: 2022年03月25日 星期五 08时53分33秒
Group       : Unspecified
Size        : 1082158
License     : BSD
Signature   : RSA/SHA256, 2022年02月23日 星期三 17时15分57秒, Key ID 05b555b38483c65d
Source RPM  : openssh-8.7p1-8.el9.src.rpm
Build Date  : 2022年02月22日 星期二 20时31分29秒
Build Host  : x86-02.stream.rdu2.redhat.com
Packager    : builder@centos.org
Vendor      : CentOS
URL         : http://www.openssh.com/portable.html
Summary     : An open source SSH server daemon
Description :
OpenSSH is a free version of SSH (Secure SHell), a program for logging
into and executing commands on a remote machine. This package contains
the secure shell daemon (sshd). The sshd daemon allows SSH clients to
securely connect to your SSH server.
```

3．`-ql`选项——查询软件包所安装的文件

通过`–ql`选项可以查看某个软件包安装了哪些程序文件，以及这些文件的安装位置。

采用RPM机制安装软件不可以由用户指定软件安装目录，这是由于Linux默认的目录结构是固定的，每个默认目录都有专门的分工，因此安装软件时会自动分门别类地向相应的目录中复制对应的程序文件，并进行相关设置。

一个典型的Linux应用程序通常由以下几部分组成。

- 普通的可执行程序文件，一般保存在`/usr/bin`目录中，普通用户即可执行。
- 管理程序文件，一般保存在`/usr/sbin`目录中，需要管理员权限才能执行。
- 配置文件，一般保存在`/etc`目录中，配置文件较多时会建立相应的子目录。
- 日志文件，一般保存在`/var/log`目录中。
- 程序的参考文档，一般保存在`/usr/share/doc`目录中。
- 可执行文件及配置文件的man手册，一般保存在`/usr/share/man`目录中。

例如，查询openssh-server在系统的什么位置安装了程序文件。

```shell
[root@localhost Packages]# rpm -ql openssh-server
/etc/pam.d/sshd
/etc/ssh/sshd_config
/etc/ssh/sshd_config.d
/etc/ssh/sshd_config.d/50-redhat.conf
/etc/sysconfig/sshd
/usr/lib/.build-id
/usr/lib/.build-id/66
/usr/lib/.build-id/66/1ce29725f87e2faa8cdb5a0e6da257db67ac06
/usr/lib/.build-id/ef
/usr/lib/.build-id/ef/8851a4c51673086772e76e6a7be4f13b810ccd
/usr/lib/systemd/system/sshd-keygen.target
/usr/lib/systemd/system/sshd-keygen@.service
/usr/lib/systemd/system/sshd.service
/usr/lib/systemd/system/sshd.socket
/usr/lib/systemd/system/sshd@.service
/usr/libexec/openssh/sftp-server
/usr/libexec/openssh/sshd-keygen
/usr/sbin/sshd
/usr/share/empty.sshd
/usr/share/man/man5/moduli.5.gz
/usr/share/man/man5/sshd_config.5.gz
/usr/share/man/man8/sftp-server.8.gz
/usr/share/man/man8/sshd.8.gz
```

4．`-qc`选项——查询软件包所安装的配置文件

通常情况下，我们更关心的是软件包在系统中安装了哪些配置文件。通过`–qc`选项可以查询某个软件包所安装的配置文件。

例如，首先安装vsftpd，然后查询vsftpd在系统中所产生的配置文件。

```shell
[root@localhost ~]# yum install vsftpd -y
[root@localhost ~]# rpm -qc vsftpd
/etc/logrotate.d/vsftpd
/etc/pam.d/vsftpd
/etc/vsftpd/ftpusers
/etc/vsftpd/user_list
/etc/vsftpd/vsftpd.conf
```

5．`-qf`选项——查询某个文件所属的软件包

通过`-qf`选项，可以查询系统中的某个文件来自于哪个软件包。

例如，查询find命令文件来自于哪个软件包。这样如果误删了find命令文件，就可以通过安装该软件包进行修复。

```shell
[root@localhost ~]# which find                #查找命令文件路径
/usr/bin/find
[root@localhost ~]# rpm -qf /usr/bin/find     #查询文件的来源包
findutils-4.8.0-5.el9.x86_64
```

## 常用的RPM命令

| 命令                  | 作用                |
| --------------------- | ------------------- |
| rpm -ivh filename.rpm | 安装软件            |
| rpm -Uvh filename.rpm | 升级软件            |
| rpm -e filename.rpm   | 卸载软件            |
| rpm -qpi filename.rpm | 查询软件描述信息    |
| rpm -qpl filename.rpm | 列出软件文件信息    |
| rpm -qf filename      | 查询文件属于哪个RPM |

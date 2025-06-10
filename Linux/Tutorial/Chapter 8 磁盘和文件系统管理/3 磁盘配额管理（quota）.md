# 磁盘配额管理（quota）

磁盘配额管理，即限制用户的可用磁盘空间，如针对提供虚拟主机的Web服务器，需要对网站空间大小进行限制；针对邮件服务器，需要对用户邮箱大小进行限制；针对文件服务器，需要对每个用户可用的网络硬盘空间进行限制等。

在Linux系统中引入了quota磁盘配额功能，对用户可使用的磁盘空间和文件数量进行限制，目的就是将用户对磁盘容量的使用限制在一个合理的水平，防止存储资源耗尽。

## 什么是磁盘配额

quota设置的磁盘配额功能只在指定的文件系统（分区）内有效，用户使用未设置配额的文件系统时，将不会受到限制。quota主要针对系统中指定的用户账号和账号进行限制，没有被设置限额的用户或组将不受影响。为组账号设置配额后，组内所有用户使用的磁盘容量、文件数量的总和不能超过限制。

通过设置磁盘配额可以对用户或组进行两方面的限制：磁盘容量和文件数量。

- 磁盘容量：限制用户能够使用的磁盘数据块的大小，也就是限制磁盘空间大小，默认单位为KB。
- 文件数量：限制用户能够拥有的文件个数。在Linux系统中，每一个文件都有一个对应的数字标记，称为i节点（inode）编号，这个编号在文件系统内是唯一的，因此quota通过限制i节点的数量来实现对文件数量的限制。

磁盘配额的限制方法分为软限制和硬限制两种。

- 软限制是指设定一个软性的配额数值（如500MB磁盘空间、200个文件），在固定的宽限期（默认为7天）内允许暂时超过这个限制，但系统会给出警告信息。
- 硬限制是指设定一个硬性的配额数值（如1GB磁盘空间、500个文件），而且绝对禁止用户超过该限值。当达到硬限制值时，系统会给出警告并禁止继续写入数据。硬限制的配额值应大于相应的软限制值，否则软限制值将失效。

## 设置磁盘配额

下面以硬盘分区`/dev/sdb1`为例，先将其挂载到`/data`目录下，然后在该文件系统中配置实现磁盘配额功能。

1．启用磁盘配额

首先需要在指定的文件系统上启用磁盘配额功能。有两种方法可以启用磁盘配额：执行mount命令或修改/etc/fstab文件。

通过执行mount命令启用的磁盘配额，在下次分区重新挂载时将消失，因而建议采用第二种方法，即通过修改配置文件`/etc/fstab`的方式启用quota磁盘配额，通过这种方式启用的磁盘配额功能可以永久生效。

下面修改/etc/fstab文件，给需要设置配额的文件系统添加磁盘配额选项。在文件系统`/data`对应行的`defaults`字段后面添加`uquota`或者`usrquota`选项，启用用户配额功能，添加`gquota`或者`grpquota`选项，启用组配额功能。

```
[root@localhost var]# vim /etc/fstab
/dev/sdb1      /data       xfs     defaults,uquota,gquota        0   0
```

修改完`/etc/fstab`文件后需要将系统重启生效。为了不重启系统，我们也可以执行命令`umount /data`将文件系统卸载，然后执行`mount -a`命令按照/etc/fstab文件里的设置重新挂载硬盘分区，使设置生效。

```
[root@localhost ~]# umount /data      #卸载文件系统
[root@localhost ~]# mount -a          #按照fstab文件设置重新挂载文件系统
```

重新挂载后，可以执行mount命令查看已经挂载的文件系统，发现其已经启用了usrquota和grpquota功能。

```
[root@localhost ~]# mount | grep sdb1               #查看已经挂载的文件系统
/dev/sdb1 on /data type xfs (rw,relatime,seclabel,attr2,inode64,usrquota,grpquota)
```

2．编辑配额设置

配额设置是实现磁盘配额功能的重要环节，使用edquota命令结合``-u`` ``-g``选项可用于编辑用户或组的配额设置。

例如，针对用户jerry进行磁盘配额设置。

```
[root@localhost ~]# edquota -u jerry            #设置用户jerry的磁盘配额
```

正确执行edquota命令后，将进入文本编辑界面，可以设置磁盘容量和文件数目的软、硬限制数值。

```
Disk quotas for user jerry (uid 1001):
Filesystem   blocks   soft     hard      inodes        soft         hard
/dev/sdb1    0        0        100000    0             0            3
```

在edquota的编辑界面中，第1行提示了当前配额文件所对应的用户或组账号，第2行是配置标题栏，分别对应每行配置记录。配置记录中从左到右分为7个字段，各字段的含义如下所述。

- Filesystem：表示本行配置对应的文件系统（分区），即配额的作用范围。
- blocks：表示用户当前已经使用的磁盘容量，默认单位为KB，该值由edquota程序自动计算生成。
- soft：第3列中的soft对应磁盘容量的软限制数值，默认单位为`KB`；第6列中的soft对应文件数量的软限制数值，默认单位为`个`。
- hard：第4列中的hard对应磁盘容量的硬限制数值，默认单位为`KB`，第7列中的hard对应文件数量的硬限制数值，默认单位为`个`。
- inodes：表示用户当前已经拥有的文件数量，该数值也由edquota程序自动计算生成。

在进行配置设置时，只需要修改相应的soft、hard列下的数值。这里将用户jerry的磁盘容量硬限制额设置为100000（100MB），文件数量硬限制额设置为3个，设置完成后保存并退出。

也可以通过`–g`选项对用户组进行配额设置，例如执行命令`edquota –g fina`设置fina组的磁盘配额。需要注意的是，配额设置仅对基本组生效。如用户jerry所属的基本组是`fina`，所属的附加组是`tech`，那么只有针对`fina`组设置的配额才对jerry有效，而对`tech`组设置的配额对jerry没有限制。

## 验证并查看磁盘配额

1．验证磁盘配额

下面我们将使用受配额限制的用户账号jerry登录，并向应用了配额的文件系统进行复制文件等写入操作，测试所设置的磁盘配额项是否有效。为了方便测试，将用户jerry的基本组设为fina，对用户和组的磁盘配额功能一并进行测试。

在测试过程中，为了快速看到效果，可以使用dd命令进行调试。dd（diskdump）是一个设备转换和复制命令，分别使用`if=`选项指定输入设备（或文件）、使用`of=`选项指定输出设备（或文件），使用`bs=`选项指定读取数据块的大小，使用`count=`选项指定读取数据块的数量。

下面从设备文件/dev/zero中读取数据生成/tmp/test1文件，读取60个大小为1MB的数据块。/dev/zero是Linux系统中一个比较特殊的设备文件，类似于我们之前用过的`黑洞`文件/dev/null，/dev/zero文件可以提供无数的0，因而当我们需要生成一个指定大小的文件，但对文件内容并不关心时，就可以从/dev/zero文件来获取数据。

```
#生成一个大小为60MB的文件/tmp/test1
[root@localhost ~]# dd if=/dev/zero of=/tmp/test1 bs=1M count=60
#查看生成的文件大小
[root@localhost ~]# ll -h /tmp/test1
-rw-r--r--. 1 root root 60M 2月  28 22:38 /tmp/test1
```

下面再生成一个大小为10MB的文件/tmp/test2。

```
[root@localhost ~]# dd if=/dev/zero of=/tmp/test2 bs=1M count=10
```

开放/data目录的写入权限。

```
[root@localhost ~]# chmod 777 /data
```

切换到jerry用户的身份进行测试。磁盘配额功能验证成功。

```
[root@localhost ~]# su - jerry
#向/data目录中复制测试文件test1
[#jerry@localhost ~]$ cp /tmp/test1 /data/a
#再次复制测试文件test1，超出容量限制
[jerry@localhost ~]$ cp /tmp/test1 /data/b
cp: 写入"/data/b" 出错: 超出磁盘限额
cp: 扩展"/data/b" 失败: 超出磁盘限额
#向/data目录中复制测试文件test2
[jerry@localhost ~]$ cp /tmp/test2 /data/b
[jerry@localhost ~]$ cp /tmp/test2 /data/c
#再次复制测试文件test2，超出数量限制
[jerry@localhost ~]$ cp /tmp/test2 /data/d
cp: 无法创建普通文件"/data/d": 超出磁盘限额
```

2．查看用户或分区的配额使用情况

可以使用quota命令结合`-u` `-g`选项分别查看指定用户和组的配额使用情况。

例如，执行`quota -u jerry`命令查看用户jerry的磁盘配额使用情况。

```
[root@localhost ~]# quota -u jerry
Disk quotas for user jerry (uid 1001):
     Filesystem  blocks  quota  limit  grace  files  quota  limit  grace
      /dev/sdb1   81920      0  100000            3*     0     3
```

可以看到jerry已经使用的磁盘空间（blocks）为81920，最大限额（limit）为100000，已经使用的文件数量（files）为3，最大限额（limit）为3。

同样可以执行`quota -g fina`命令查看fina组的磁盘配额使用情况。

```
[root@localhost ~]# quota -g fina
Disk quotas for group fina (gid 1002):
     Filesystem  blocks  quota  limit  grace  files  quota  limit  grace
      /dev/sdb1   81920      0  200000            3      0      6
```

也可以使用repquota命令针对指定的文件系统输出配额情况报告。例如，执行`repquota /data`命令查看`/data`文件系统的配额使用情况报告。从中可以看到有哪些用户在`/data`中被设置了磁盘配额，以及用户磁盘空间的使用情况，如图4-17所示：

```
[root@localhost ~]# repquota /data
```

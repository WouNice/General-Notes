# 配置文件访问控制列表（FACL）

通过chmod、chown命令设置权限时，权限都是针对某一类用户设置的。当我们希望对某个指定的用户进行单独的权限控制，例如某个目录要开放给某个特定的使用者使用时，这些方法就无法满足要求了，这时就需要用文件访问控制列表来实现。

例如，`/home/project`目录的所有者是`root`用户，所属组是`root`组，预设权限是`770`。现在有个名为`manager`的用户，属于`manager`组，希望能够对`/home/project`目录具有读、写和执行权限；还有一个名为`test`的用户，属于`test`组，也希望能够对`/home/project`目录具有读取和执行权限。很明显，利用`chmod`或`chown`命令是无法完成这些要求的。因而，Linux系统提供了FACL（File Access Control List，文件访问控制列表）来专门完成这种单独的权限设置。

## 设置FACL

设置FACL使用的是setfacl命令，命令格式如下：

```
setfacl [选项] 设定值 文件名
```

常用选项如下：

- -m：m表示modify，用于设定或者修改一个FACL规则。
- -x：取消一个FACL规则。
- -b：取消所有的FACL规则。

例如，设置`manager`对`/home/project`目录具有rwx权限。这里采用的设置格式为“`u:manager:rwx`”，其中“`u`”代表用户设置权限，如果要为组设置权限，则采用“`g`”。

```
[root@localhost ~]# setfacl -m u:manager:rwx /home/project/
[root@localhost ~]# ll -d /home/project/
drwxrwxr-x+ 2 root root 6  3月 27 10:35 /home/project/
```

设置完FACL后，查看文件详细信息时会发现在权限部分多出一个“`+`”标识，代表文件启用了FACL权限。

例如，将`test`用户对于`/home/project`目录的规则从FACL中去除。

```
[root@localhost ~]# setfacl -x u:test /home/project/
```

通过“`setfacl -b`”命令去除所有的FACL规则：

```
[root@localhost ~]# setfacl -b /home/project/
```

需要注意的是，利用FACL为用户所设置权限的优先级要高于用户的基本权限。例如，对于/home/project目录，用户manager2是目录所属组manager的成员，该用户理应具有rwx权限。如果通过设置FACL不赋予其任何权限，那么manager2对`/home/project`目录最终将没有任何权限。

```
[root@localhost ~]# setfacl -m u:manager2:--- /home/project/     #指定manager2没有任何权限
[root@localhost ~]# su – manager2         #切换到jerry进行测试
[manager2@localhost ~]# touch /home/project/test
touch: 无法创建"/home/project/test": 权限不够
```

最后，解释一下FACL中`mask`的含义。为一个文件设置了FACL之后，在用`getfacl`命令查看时，规则列表中会有一行`mask`信息，代表当前所有用户的FACL最大权限。例如，我们之前设置的`manager`用户的FACL权限为`rwx`，`test`用户的FACL权限为`r-x`，那么mask的值就为所有FACL之和rwx。另外需要注意的是，为一个文件设置了FACL之后，在查看该文件的权限信息时，用户组所对应的权限位将被FACL的mask值所取代。

例如，我们执行下面的操作。

```
[root@localhost ~]# mkdir /tmp/test
[root@localhost ~]# ll -d /tmp/test    #目录的默认权限为755
drwxr-xr-x. 2 root root 6  3月 27 10:49 /tmp/test
[root@localhost ~]# setfacl -m u:manager:rwx /tmp/test
[root@localhost ~]# ll -d /tmp/test    #设置FACL之后，用户组的权限被mask取代
drwxrwxr-x+ 2 root root 6  3月 27 10:49 /tmp/test
```

因此，当我们查看某个文件的权限时，如果权限位的最后部分为“+”，就意味着该文件被设置了FACL，此时用户组所对应的权限位就不再是原先的含义，应通过执行getfacl命令来查看文件的具体权限设置。

当我们通过“`setfacl -x`”命令来取消某个用户的FACL规则时，即使目标文件已经没有了任何FACL规则，但在查看目标文件的属性信息时，权限位的最后部分仍然会有“+”，这就意味着第二组权限仍然表示mask值，而不是用户组的权限。因此，如果要取消所有FACL规则，那么建议采用“`setfacl -b`”方式，这样目标文件就彻底恢复到设置FACL之前的状态了。

## 查看FACL

通过`getfacl`命令可以查看FACL规则。

```
root@localhost ~]# getfacl /home/project/
getfacl: Removing leading '/' from absolute path names
# file: home/project/
# owner: root
# group: root
user::rwx
user:manager:rwx
user:test:r-x
group::r-x
mask::rwx
other::r-x
```

## 启用FACL支持

FACL既可以针对用户设置，也可以针对用户组设置。使用FACL必须要有文件系统的支持，Linux中标准的EXT系列和XFS文件系统都支持FACL功能。但是要注意，Linux中默认的文件系统支持FACL，但如果是新挂载的分区，则不支持FACL应用，可以在挂载文件系统时使用`-o acl`选项启动FACL支持。

例如，将`/dev/sdb1`分区挂载到`/home`目录，并启用FACL支持。文件系统挂载之后，通过`mount`命令确认FACL已启用。

```
[root@localhost ~]# mount -o acl /dev/sdb1 /home
[root@localhost ~]# mount | grep home
/dev/sdb1 on /home type xfs (rw,acl)
```

如果想要在系统启动时自动应用FACL功能，则需要修改`/etc/fstab`文件，在相应挂载设备的挂载选项“`defaults`”后面添加“`acl`”选项。

```
[root@localhost ~]# vim /etc/fstab
/dev/sdb1               /home      xfs    defaults,acl    0      0
```

## 配置FACL时应注意的问题

FACL用于提供额外权限，主要用来对权限进行微调。在系统中设置权限时，主要还是应该依靠chmod、chown这些传统的方法，而不能以FACL为主，否则维护起来会比较麻烦。

因而，在生产环境中设置权限时，建议先用chmod、chown设置总体权限，然后根据需要再用FACL设置细部权限。

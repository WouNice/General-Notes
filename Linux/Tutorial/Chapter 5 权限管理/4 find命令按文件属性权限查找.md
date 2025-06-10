# find命令按文件属性/权限查找

## 根据文件属性查找

文件属性主要是指文件的所有者和所属组这两种所属关系。按文件属性查找，主要有以下选项。

- -user 用户名：根据所有者查找。
- -group 组名：根据所属组查找。
- -uid UID：根据UID查找。
- -gid GID：根据GID查找。
- -nouser：查找没有所有者的文件。
- -nogroup：查找没有所属组的文件。

例如，在/home目录下查找所有属于用户student的文件或目录。

```
[root@localhost ~]# find /home -user student -ls
```

例如，在/var目录中查找所有者为root且所属组为mail的文件或目录。

```
[root@localhost ~]# find /var -user root -group mail -ls
```

有时可能会遇到这样的情况，比如文件/tmp/test属于zhangsan所有，如果将用户zhangsan删除，那么/tmp/test的所有者和所属组就变成了zhangsan原先的UID和GID。这时，我们也可以通过UID或GID去查找这类文件。

```
[root@localhost ~]# find /tmp -uid 502 -ls
```

对于/tmp/test这样的所有者和所属组变成了UID和GID的文件，就称为没有所有者和没有所属组的文件，这样的文件在系统中有一定危险性，因此，我们可以通过`-nouser`或`-nogroup`选项去查找这类文件。在找到这类文件之后，最好利用chown命令重新为其指定所有者和所属组。

```
[root@localhost ~]# find /tmp -nouser -ls
[root@localhost ~]# find /tmp -nogroup -ls
```

## 根据文件权限查找

相对于按照文件属性查找，按文件权限查找在实践中要应用得更多一些，用法也相对复杂。下面分类进行介绍。

1．精确匹配权限

按文件权限查找，需要用到`-perm`选项。`-perm`选项的基本用法很简单，格式为`-perm mode`，其中mode为所要匹配的权限，这种查找方式实现的是精确匹配。

例如，要在/boot目录中查找权限为755的普通文件，并显示详细信息。我们设置查找条件为`-perm 755`，可以发现共找到两个文件，这两个文件的权限都对查找条件进行了精确匹配。

```
[root@localhost ~]# find /boot -perm 755 -type f -ls
      133  10880 -rwxr-xr-x   1 root     root     11137712 3月  9 04:53 /boot/vmlinuz-5.14.0-71.el9.x86_64
      139  10880 -rwxr-xr-x   1 root     root     11137712 3月 25 08:54 /boot/vmlinuz-0-rescue-c546bc279746477590792eee11e17e70
```

2．模糊匹配权限

在更多情况下，我们希望能够对权限进行模糊匹配，比如查找所属组具有写权限的目录，或者查找其他用户具有写权限的文件等。在这些情况下，我们只关心所属组或其他用户是否有相应的权限，而不关心整体权限，因而这时使用精确匹配就无法满足要求了。

`-perm`选项提供了两种模糊匹配的方式：`-perm /mode`和`-perm -mode`。这两种模糊匹配方式不是很好理解。下面先举例说明它们之间的区别。

例如，我们要查找的权限为`220`。如果用字符的形式来表示权限的话，那么应该是`-w--w----`；如果用二进制的形式来表示的话，那么应该是`010010000`，如下所示：

```
用十进制数表示权限  2    2    0
用字符表示权限     -W-  -W-  ---
用二进制数表示权限 010   010  000
```

这里重点参考采用二进制形式表示的权限，其中数字0表示忽略相应位置的权限，数字1表示匹配相应位置的权限。因此，在采用`220`作为权限查找条件进行模糊匹配时，就表示要求所有者和所属组应具有写权限，而对其他的权限则予以忽略。

理解这点之后，`-perm /mode`和`-perm -mode`之间的区别就好理解了。`-perm /mode`要求所匹配的权限之间是`或`的关系，`-perm -mode`则要求所匹配的权限之间是`与`的关系。也就是说，`-perm/220`表示所有者或所属组中的任何一个具有写权限就可以，而`-perm -220`则表示所有者和所属组必须同时具有写权限。

下面通过实例进行验证，首先，我们准备一些测试文件。

```
[root@localhost ~]# mkdir /tmp/test
[root@localhost ~]# touch /tmp/test/test{1,2,3}
[root@localhost ~]# chmod 644 /tmp/test/test{1,2,3}
[root@localhost ~]# chmod 664 /tmp/test/test2
[root@localhost ~]# chmod 600 /tmp/test/test3
```

然后，我们分别通过两种不同的方式进行模糊匹配。

以`-perm /220`作为条件，查找所有者或所属组具有写权限的文件，可以看到3个测试文件均符合查找条件。

```
[root@localhost ~]# find /tmp/test -perm /220 -type f
/tmp/test/test1
/tmp/test/test2
/tmp/test/test3
```

以`-perm -220`作为条件，查找所有者和所属组都具有写权限的文件，只有/tmp/test/test2符合查找条件。

```
[root@localhost ~]# find /tmp/test -perm -220 -type f
/tmp/test/test2
```

因此，如果要在系统中查找所有人都有写权限的目录，则应该指定条件为`-perm -222`。如果以`-perm /222`为查找条件，则所有者、所属组或其他用户中任何一个具有写权限都会符合要求。

```
[root@localhost ~]# find / -perm -222 -type d -ls 2> /dev/null
  6810    0 drwxrwxrwt   2 root     root    100 12月 23 03:37 /dev/shm
654108    4 drwxrwxrwt   2 root     root    4096 12月 23 03:42 /var/tmp
784897    4 drwxrwxrwt  12 root     root    4096 12月 23 06:58 /tmp
……
```

3．查找特殊权限

除基本权限之外，find命令也支持查找特殊权限。对于特殊权限，SUID对应的数字是4，SGID对应的数字是2，粘滞位（SBIT）对应的数字是1。如果某个文件或目录被设置了特殊权限，那么它用数字形式表示的权限就成了4位数，特殊权限被放在左侧最高位。例如，/usr/bin/passwd文件的权限为`rwsr-xr-x`，用数字形式表示就是4755。又如，/tmp目录的权限为`rwxrwxrwt`，用数字形式表示就是1777。

因此，如果在系统中查找所有设置了SUID的文件，那么应将查找条件设置为`4000`。由于所要查找的权限位只有1个，因此无论使用`-perm -4000`还是`-perm /4000`，都可以达到相同的效果。在CentOS7系统中，这类文件的数量是27个。

```
[root@localhost ~]# find / -perm -4000 2> /dev/null | wc -l
27
[root@localhost ~]# find / -perm /4000 2> /dev/null | wc -l
27
```

同理，如果要查找所有设置了SGID的目录，那么应指定条件为`-perm -2000`或`-perm /2000`。查找设置了SBIT权限的目录，可以指定条件为`-perm -1000`或`-perm /1000`。

如果要查找所有设置了SGID或SBIT权限的目录，那么应该指定条件为`-perm /3000`。如果将条件指定为`-perm -3000`，则表示查找既设置了SGID又设置了SBIT的目录。

```
#查找同时设置了SGID和SBIT的目录
[root@localhost ~]# find / -perm -3000 -type d -ls 2> /dev/null
#查找设置了SGID或SBIT的目录
[root@localhost ~]# find / -perm /3000 -type d -ls 2> /dev/null
  8400    0 drwxrwxrwt   2 root     root     40 9月 11 11:17 /dev/mqueue
  8622    0 drwxrwxrwt   2 root     root     40 9月 11 11:17 /dev/shm
  69      4 drwxrwxrwt   7 root     root     4096 10月 19 11:16 /var/tmp
……
```

# tar 命令

gzip、bzip2 及 zip 命令无法对目录进行压缩，而在实际使用时不可避免地需要压缩目录，这就要用到 tar 命令。tar 命令对文件和目录都可以压缩，也可以将多个文件和目录同时打包成一个压缩文件，因此该命令的使用较为频繁。tar 命令的多短格式选项可以一起使用。

创建归档文件的命令格式如下：

```shell
tar 选项 归档文件名 源文件或目录
```

解压归档文件的命令格式如下：

```shell
tar 选项 归档文件名 [-C 目标路径]
```

tar 命令的常用选项如下：

- -c：创建.tar 格式的包文件。
- -x：解压.tar 格式的包文件。
- -v：输出详细信息。
- -f：表示使用归档文件。
- -p：打包时保留原始文件及目录的权限。
- -t：列表查看包内的文件。
- -C：解包时指定释放的目标文件夹。
- -z：调用 gzip 程序进行压缩或解压。
- -j：调用 bzip2 程序进行压缩或解压。

下面的示例是对以上选项用法的演示，注意查看显示结果。

```shell
# 调用 gzip 命令将/mnt 和/opt 目录打包备份到/home 目录下，压缩文件名为 testgz.tar.gz
[root@localhost /]# tar -czvf /home/testgz.tar.gz /opt/mnt
tar: 从成员名中删除开头的“/”
/opt/
/opt/rh/
/opt/2.txt
/opt/1.txt
/mnt/
/mnt/opt/
/mnt/opt/1.txt
/mnt/opt/2.txt
[root@localhost /]# ls /home
testgz.tar.gz  test.zip  zhangsan

# 在不解压的前提下查看压缩包中的文件
[root@localhost /]# tar -tvf /home/testgz.tar.gz
drwxr-xr-x root/root            0 2020-12-06 15:55 opt/
drwxr-xr-x root/root            0 2018-10-31 03:17 opt/rh/
-rw-r--r-- root/root            4 2020-12-06 15:24 opt/2.txt
-rw-r--r-- root/root            4 2020-12-06 15:23 opt/1.txt
drwxr-xr-x root/root            0 2020-12-06 16:04 mnt/
drwxr-xr-x root/root            0 2020-12-06 16:04 mnt/opt/
-rw-r--r-- root/root            4 2020-12-06 15:23 mnt/opt/1.txt
-rw-r--r-- root/root            4 2020-12-06 15:24 mnt/opt/2.txt

# 将 testgz.tar.gz 文件解压到/media 目录下
[root@localhost /]# tar -xzvf /home/testgz.tar.gz  -C /media
opt/
opt/rh/
opt/2.txt
opt/1.txt
mnt/
mnt/opt/
mnt/opt/1.txt
mnt/opt/2.txt
[root@localhost /]# ls -R /media
/media:
mnt  opt
/media/mnt:
opt
/media/mnt/opt:
1.txt  2.txt
/media/opt:
1.txt  2.txt  rh
/media/opt/rh:
```

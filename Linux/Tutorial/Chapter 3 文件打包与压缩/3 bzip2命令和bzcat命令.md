# bzip2命令和bzcat命令

bzip2命令用于解压缩后缀名为.bz2的归档文件，该命令无法将多个文件打包成一个压缩包，同样，目录是无法压缩的。bzcat命令可以对.bz2格式的压缩文件在不解压的前提下查看压缩包中的内容。

制作压缩文件时，选项-9表示高强度压缩，可以让制作的压缩文件体积更小。命令格式如下：

```
bzip2 [-9] 文件名
```

解压缩文件的命令格式如下：

```
bzip2 -d 归档文件名.bz2
```

或

```
bunzip2 归档文件名.bz2
```

不解压查看.bz2压缩包文件内容的命令格式如下：

```
bzcat 归档文件名.bz2
```

或

```
bunzip2 -c 归档文件名.bz2
```

下面的示例是对以上选项用法的演示，注意查看显示结果。

```shell
#还是在/opt目录下，使用bzip2命令压缩文件1.txt，并使用ls命令查看是否生成了.bz2格式的压缩文件，同样使用的也是相对路径
[root@localhost opt]# bzip2 1.txt
[root@localhost opt]# ls
1.txt.bz2  2.txt  rh

#在不解压的前提下查看.bz2压缩文件的内容
[root@localhost opt]# bzcat 1.txt.bz2
123
[root@localhost opt]# bunzip2 -c 1.txt.bz2
123

#使用bzip2命令解压.bz2格式的压缩文件
[root@localhost opt]# bzip2 -d 1.txt.bz2
[root@localhost opt]# ls
1.txt  2.txt  rh
[root@localhost opt]# cat 1.txt
123
```

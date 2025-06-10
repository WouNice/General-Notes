# gzip命令和zcat命令

gzip命令用于解压缩后缀名为.gz的归档文件，该命令无法将多个文件打包成一个压缩包，也无法压缩目录。zcat命令可以对.gz格式的压缩文件在不解压的前提下查看压缩包中的内容。

制作压缩文件时，选项-9表示高强度压缩，可以让制作的压缩文件体积更小。命令格式如下：

```
gzip [-9] 文件名
```

解压缩文件的命令格式如下：

```
gzip -d 归档文件名.gz
```

或

```
gunzip 归档文件名.gz
```

不解压查看.gz压缩包文件内容的命令格式如下：

```
zcat 归档文件名.gz
```

或

```
gunzip -c 归档文件名.gz
```

下面的示例是对以上选项用法的演示，注意查看显示结果。

```shell
#创建1.txt和2.txt文件，分别向其中添加一行数据，此处是在/opt目录下直接操作，使用的都是相对路径
[root@localhost opt]# echo "123" > 1.txt
[root@localhost opt]# echo "456" > 2.txt
[root@localhost opt]# cat 1.txt
123
[root@localhost opt]# cat 2.txt
456
[root@localhost opt]# ls
1.txt  2.txt  rh

#将1.txt文件压缩，压缩后1.txt原文件消失，使用ls命令查看是否生成压缩文件，可以看到生成的压缩文件名是在原有的文件名基础上添加后缀.gz成为压缩文件
[root@localhost opt]# gzip 1.txt
[root@localhost opt]# ls
1.txt.gz  2.txt  rh

#在不解压的前提下查看压缩文件中的数据内容，分别使用zcat和gunzip进行查看
[root@localhost opt]# zcat 1.txt.gz
123
[root@localhost opt]# gunzip -c 1.txt.gz
123

#使用gunzip解压并查看解压后的文件情况，当然也可以通过gzip -d来实现
[root@localhost opt]# gunzip 1.txt.gz
[root@localhost opt]# ls
1.txt  2.txt  rh

#使用通配符*压缩同类后缀名的文件，有多少个文件就生成多少个.gz格式的压缩包
[root@localhost opt]# gzip *.txt
[root@localhost opt]# ls
1.txt.gz  2.txt.gz  rh

#使用通配符*解压所有.gz格式的压缩包，为防止出现问题，可以查看解压后某个文件的内容进行确认
[root@localhost opt]# gzip -d *.gz
[root@localhost opt]# ls
1.txt  2.txt  rh
[root@localhost opt]# cat 1.txt
123
```

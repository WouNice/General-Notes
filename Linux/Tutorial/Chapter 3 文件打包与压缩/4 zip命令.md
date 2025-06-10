# zip命令

zip命令用于解压缩后缀名为.zip的归档文件。该命令可以将多个文件打包成一个压缩包，也可以在解压缩时指定路径名，而gzip和bzip2命令是无法指定路径的，对于目录也是无法压缩的。

制作压缩文件时，压缩文件名可以根据实际情况自定义。命令格式如下：

```
zip 归档文件名.zip 文件名
```

解压缩文件时，可以使用“-d”选项指定解压路径。命令格式如下：

```
unzip [-d] 归档文件名.zip
```

不解压查看zip压缩包中的文件信息（不是查看文件内容），格式如下：

```
unzip -l 归档文件名.zip
```

下面的示例是对以上选项用法的演示，注意查看显示结果。

```shell
#将/opt目录下的1.txt和2.txt文件打包压缩到/home目录下生成test.zip压缩文件，此处的操作目录可以不用固定在/opt下，都是根据绝对路径实现的
#第一步：查看/home和/opt目录下的文件信息，并查看文件1.txt的内容
[root@localhost /]# ls /home
zhangsan
[root@localhost /]# ls /opt
1.txt  2.txt  rh
[root@localhost /]# cat /opt/1.txt
123
#第二步：使用zip命令按照需求进行压缩，查看/opt目录，可以发现原文件是存在的
[root@localhost /]# zip /home/test.zip /opt/1.txt /opt/2.txt
  adding: opt/1.txt （stored 0%）
  adding: opt/2.txt （stored 0%）
[root@localhost /]# ls /opt
1.txt  2.txt  rh
[root@localhost /]# ls /home
test.zip  zhangsan

#在不解压的前提下查看压缩包中存在的文件，注意不是查看文件内容，因为不是单个文件所以无法直接查看文件内容
[root@localhost /]# unzip -l /home/test.zip
Archive: /home/test.zip
  Length    Date    Time    Name
---------       ----------      -----   ----
        4   12-06-2020      15:23   opt/1.txt
        4   12-06-2020      15:24   opt/2.txt
---------                       -------
        8                   2 files

#使用unzip命令将/home/test.zip文件通过“-d”选项解压到/mnt目录下，并查看/home目录下的zip文件是否存在，确认/mnt目录下的1.txt文件内容是否解压出错
[root@localhost /]# unzip -d /mnt /home/test.zip
Archive: /home/test.zip
 extracting:/mnt/opt/1.txt
 extracting:/mnt/opt/2.txt
[root@localhost /]# ls /home/
test.zip  zhangsan
[root@localhost /]# ls /mnt/opt
1.txt  2.txt
[root@localhost /]# cat /mnt/opt/1.txt
123
```

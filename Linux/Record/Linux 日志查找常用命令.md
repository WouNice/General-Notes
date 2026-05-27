# Linux 日志查找常用命令

## cat、zcat

-   cat -n app.log | grep "error"：查询日志中含有某个关键字error的信息，显示行号。
-   cat -n app.log | grep "error" --color：查询日志中含有某个关键字error的信息，显示行号，带颜色的。
-   cat -n test.log | tail -n +92 | head -n 20 ：表示查询92行之后的日志；表示在前面的查询结果里再查前20条记录。
-   cat -n app.log |grep "error" | more：使用more和less命令分页查看日志，空格键翻页。
-   cat -n app.log |grep "error" > temp.txt：把日志保存到文件。
-   zcat shopdiy-job01.jd__shopdiy-job.warn.log-2019-03-25-1.log.gz | gerp "hotelFriendCircle poi"：直接查找压缩包里的日志内容
-   gzip -dc app.log.2018-01-11.gz |grep "此次共实际刷数据" | more：不解压直接查看压缩包中的“关键词”信息

## grep

-   grep -E "123|abc" app.log（grep -E "123|abc" app.log --color）：查询日志文件app.log中包含 关键字 123 或 包含 关键字 abc 的行。（带颜色）
-   tail -f app.log | grep -E -C20 "123|abcd" --color：实时查看日志，且包含关键字 123 或者 abcd 的行。
-   grep -nE "123|abc" app.log：查询日志文件app.log中包含 关键字 123 或 包含 关键字 abc 的行，显示行号。
-   grep "此次共实际刷数据" app.log：查看日志中的关键字。
-   grep "此次共实际刷数据" app.log | more：查询关键字，并且按空格键翻页。
-   grep -i "magic" app.log ：不分大小写的搜索匹配magic。默认情况下是区分大小写的。
-   grep -w "magic" app.log ：匹配整个单词而不是字符串的一部分（如匹配“magic”，而不是“magical”）。
-   grep -E "123" app.log：显示app.log中含有关键字"123"的行内容。
-   grep -E -A5 "123" app.log：显示app.log中含有关键字"123"的行内容，以及之后的5行内容。
-   grep -E -B5 "123" app.log：显示app.log中含有关键字"123"的行内容，以及之前的5行内容。
-   grep -E -C5 "123" app.log：显示app.log中含有关键字"123"的行内容，以及之前之后的各5行内容。

## head【头正尾负】

-   head -n 10 app.log（head -n +10 app.log、head 10 app.log）：查看日志中的头10行日志。
-   head -n -10 app.log：查看日志中除了尾10行之外的所有日志。

## tail【头正尾负】

-   tail -n 10 app.log（tail -n -10 app.log、tail -10 app.log）：查看日志中的尾10行日志。
-   tail -n +10 app.log：查看日志中除了头10行之外的所有日志。
-   tail -f app.log：实时查看日志信息。
-   tail -f app.log |grep "关键词"：实时查看日志中含有某个关键词的信息。
-   tail -f app.log |grep -A3 "关键词"：实时查看日志中含有某个关键词的信息，该行的前面和后面各多显示3行。比如：tail -f app.log |grep -A3 "此次共实际刷数据"。
-   tail -f app.log |grep "mtUserScore" --color：实时查看app.log日志中的含有关键词“mtUserScore”的日志信息，并且彩色打印。
-   tail -n 20 app.log |grep --color "此次共实际刷数据"：查看日志的最后20行并且搜索关键字。
-   tail -fn 100 app.log：循环实时查看最后100行的记录。
-   tail -fn 100 app.log |grep --color "此次共实际刷数据"：循环查看最后100行的记录，并且查找关键字。
-   tail -500 /data/applogs/tomcat/catalina.out：查看Tomcat错误日志。
-   tail -f app.log | grep -A50 "randomFlow=" --color：查看后面50行的日志。tail -f app.log | grep -A50 "接口查询结果=" --color
-   tail -f app.log | egrep "repast launchId=|whtOuterCreativeService.generateCreative" --color：命中多个关键字中其中一个即可，注意：| 的前后不能有空格。
-   tail -f app.log | egrep "ERROR|WARN" --color：命中多个关键字中其中一个即可，注意：| 的前后不能有空格。
-   tail -f app.log | egrep "ERROR|WARN" -wc：统计数量，命中多个关键字中其中一个即可，注意：| 的前后不能有空格。
-   grep "modifyBudget" midas-shopdiy-web.access.log | grep "realAccountId=58754220" --color：同时满足两个条件。
-   tail -f nginx.access.log | grep -E "getPaginateList|queryAccounts" --color ：匹配多个关键字之一即可

## ps

-   ps -ef |grep java：查看Java进程。
-   ps -ef |grep tomcat：查看Tomcat进程。
-   curl www.baidu.com：查看url能不能使用。
-   ps -ef | grep 8080：查看端口号8080被哪个进程占用。
-   ps -ef | grep redis：查看已经启动所有进程中与字符“redis”相关的进程。
-   ps -ef | grep java：查看已经启动所有进程中与字符“java”相关的进程。
-   ping 192.168.1.122：查看本机能否和IP=192.168.1.122的主机连通。
-   scp -r redis-3.0.0/ 192.168.1.122:/usr/local：如果可以连通，则可以远程传输文件。
-   cp redis.conf /usr/local/redis-cluster/7001：复制文件到指定文件夹。

## Nginx日志查询

-   应用日志记录的位置：/data/applogs/shopdiy-web/logs
-   tail -f app.log：动态查看应用日志。
-   应用Nginx日志记录的位置，HTTP请求时长结果查看：/data/applogs/nginx
-   tail -f shopdiy-web.access.log：通过访问页面来动态查看Nginx日志。

## 压缩包的解压-压缩

-   gz压缩：app.log-2018-09-22-1.log 文件，执行：gzip app.log-2018-09-22-1.log，被压缩成 app.log-2018-09-22-1.log.gz
-   gz接压缩：app.log-2018-09-22-1.log.gz 文件，执行：gzip -d app.log-2018-09-22-1.log.gz，被解压成 app.log-2018-09-22-1.log
-   gzip -dc app.log.2018-01-11.gz |grep "此次共实际刷数据" | more：不解压直接查看压缩包中的“关键词”信息

## sed

-   这个命令可以查找日志文件特定的一段 , 也可以根据时间的一个范围查询。
-   sed -n "5,10p" app.log：按照行数——查看日志第5到第10行。
-   sed -n "/2018-04-08 09:40:53.374/,/2018-04-08 10:21:04.812/p" express.log | grep "此次共实际刷数据" ：按照时间段——查看两个时间之间的日志，并且显示关键字。其中，时间点一定要在日志中存在，可用：grep -E "2018-04-08 09:40:53.374" app.log --color：来查看时间点是不是存在日志中，带颜色。

## lsof （list open files）

-   lsof -i tcp：查看TCP连接的端口号使用情况
-   lsof -i udp：查看UDP连接的端口号使用情况
-   lsof -i tcp:58528 :（port替换成端口号，比如6379）可以查看该端口被什么程序占用，并显示PID，方便kill
-   lsof -i udp:58529 :（port替换成端口号，比如6379）可以查看该端口被什么程序占用，并显示PID，方便kill
-   kill -9 518：结束进程PID号为518的进程

## netstat

-   netstat -a：列出所有端口
-   netstat -at ：列出所有tcp端口
-   netstat -au：列出所有udp端口
-   netstat -l ：只显示监听端口
-   netstat -lt ：只列出所有监听tcp端口
-   netstat -lu ：只列出所有监听udp端口
-   netstat -st：显示 TCP 端口的统计信息
-   netstat -su：显示 UDP 端口的统计信息

## ifconfig

ifconfig：Mac查看本机IP地址。其中的ether是硬件mac地址，inet是IP地址。

```sh
[root@localhost ~]# ifconfig
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.117.128  netmask 255.255.255.0  broadcast 192.168.117.255
        inet6 fe80::20c:29ff:fec0:9e91  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:c0:9e:91  txqueuelen 1000  (Ethernet)
        RX packets 470  bytes 47097 (45.9 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 681  bytes 126643 (123.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

## 查看日志文件大小和查看指定关键词

（1）wc -c app.log：参数 -c 表示统计字符，因为一个字符是一个字节，所以这样得到字节数。

（2）du -sb app.log：参数 -b 表示以B计数。

（3）du -sm app.log：参数 -m 表示以M计数。

（4）du -sh app.log：参数 -h 表示以K/M/G计数，容易识别出文件大小 。

```java
cd /data/applogs/nginx
ls -al
du -sh dp-nginx.access.log
查看这个文件中这个关键词被调用地方：
cat dp-nginx.access.log | grep "queryAccountBalance"
查看这个文件中这个关键词被调用的数量：
cat dp-nginx.access.log | grep "queryAccountBalance" | wc -l
查看这些文件中这个关键词被调用的数量：
cat dp-nginx.access* | grep "queryAccountBalance" | wc -l
```

### 另：less app.log使用：

```java
(1) cd /data/applogs/shopdiy-cpc-job/logs
(2) ls -al
(3) 分页加载日志数据：less app.log
(4) 此时，shift+GG 命令进入文件尾部，然后输入 shfit+?，并且后面跟随你想要搜索的关键词，比如 ?NewCheckEntityStatusJob，回车，此时你想要看到的关键词都会加亮显示，供你查阅。
此时，N 是逐个向上翻阅；shift+N 是逐个向下翻阅。Q 是退出日志界面。
```

## 杂家笔记

-   less log.log：shift + GG 命令到文件尾部，然后输入 ？加上你要搜索的关键字例如 ？1213 shift+n 关键字之间进行切换。
-   tac：则是由最后一行到第一行反向在萤幕上显示出来，与cat反过来。
-   history：所有的历史记录
-   history | grep XXX：历史记录中包含某些指令的记录
-   history | more：分页查看记录
-   history -c：清空所有的历史记录
-   uname -a：查看内核/操作系统/CPU信息
-   cat /proc/cpuinfo：查看CPU详细信息
-   hostname：查看计算机名称
-   env：查看系统环境变量
-   free -m：查看内存使用量、查看交换区使用量
-   df -h：查看各分区使用量
-   du -sh 目录名：查看指定目录的大小
-   du -sh 文件名：查看指定文件的大小
-   grep MemTotal /proc/meminfo：查看计算机是多大内存，常用的是8G=8388608kB
-   grep MemFree /proc/meminfo：查看计算机还剩余多少内存
-   uptime：查看系统运行时间、用户数量、负载
-   ifconfig：查看所有网络接口的属性
-   netstat -lntp：查看所有监听的端口号
-   ps -ef：查看所有进程
-   top：实时显示进程状态
-   last：查看用户登录这台机器的日志
-   rpm -qa：查看系统上安装的所有的软件包
-   wc app.log：app.log文件的统计信息 3、92、598 app.log。app.log 文件的行数为3、单词数92、字节数598。

## 正则表达式

grep：grep详解

```java
# grep默认是区分大小写的，这里将匹配所有的小写字母： $ echo '1234\nabcd' | grep '[a-z]'
# 将匹配所有的数字： $ echo '1234\nabcd' | grep '[0-9]'
# 将匹配所有的数字 ：$ echo '1234\nabcd' | grep '[[:digit:]]'
# 将匹配所有的小写字母 ：$ echo '1234\nabcd' | grep '[[:lower:]]'
# 将匹配所有的大写字母 ：$ echo '1234\nabcd' | grep '[[:upper:]]'
# 将匹配所有的字母和数字，包括0-9,a-z,A-Z ：$ echo '1234\nabcd' | grep '[[:alnum:]]'
# 将匹配所有的字母 ：$ echo '1234\nabcd' | grep '[[:alpha:]]'
cat 1.txt | grep "[0-9]"
cat 1.txt | grep "[[:digit:]]"
cat 1.txt | grep "[[:alpha:]]"
cat 1.txt | grep "[[:upper:]]"
cat 1.txt | grep "[[:lower:]]"
cat 1.txt | grep "[[:alnum:]]"
```

### grep简介

grep：Gloabal Search Regular Expression and Print out the line，意为全局搜索正则表达式并打印文本行。所以

-   grep是一个强大的文本搜索工具。
-   grep与正则表达式联系紧密。

之后也会从这两个大的方面来详细介绍。grep命令的基本语法如下：grep [options] pattern [file...]

-   options表示选项，具体的命令选项见下表。
-   pattern表示要匹配的模式（包括目标字符串、变量或者正则表达式）。
-   file表示要查询的文件名，可以是一个或者多个。pattern后面所有的字符串参数都会被理解为文件名。

| 选项 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| -c   | 只打印匹配的文本行的行数，不显示匹配的内容                   |
| -i   | 匹配时忽略字母的大小写                                       |
| -h   | 当搜索多个文件时，不显示匹配文件名前缀                       |
| -n   | 列出所有的匹配的文本行，并显示行号                           |
| -l   | 只列出含有匹配的文本行的文件的文件名，而不显示具体的匹配内容 |
| -s   | 不显示关于不存在或者无法读取文件的错误信息                   |
| -v   | 只显示不匹配的文本行                                         |
| -w   | 匹配整个单词                                                 |
| -x   | 匹配整个文本行                                               |
| -r   | 递归搜索，搜索当前目录和子目录                               |
| -q   | 禁止输出任何匹配结果，而是以退出码的形式表示搜索是否成功，其中0表示找到了匹配的文本行 |
| -b   | 打印匹配的文本行到文件头的偏移量，以字节为单位               |
| -E   | 支持扩展正则表达式                                           |
| -P   | 支持Perl正则表达式                                           |
| -F   | 不支持正则表达式，将模式按照字面意思匹配                     |

## grep简单应用场景

**首先介绍一下我的实验环境，在/Users/cmm/test4MingLing下建立3个简单的txt文件：**

```java
#----------------------/Users/cmm/test4MingLing/text1.txt---------------------
  hello world
  mailx
  uuen
  letitia
  #----------------------/Users/cmm/test4MingLing/text2.txt---------------------
  hello world,this is for grep test
  #----------------------/Users/cmm/test4MingLing/text3.txt---------------------
  hello world
  tyrone
```

（1）多文件查询，file之间用空格隔开。

```java
grep -i "hello world" text1.txt text2.txt
#输出结果
text1.txt:hello world
text2.txt:hello world,this is for grep test
```

（2）多模式匹配，模式之间为“逻辑或”的关系，匹配任意一个即可返回。

```java
#方法1：使用-e选项
grep -e "hello world" -e "mailx" -r /Users/cmm/test4MingLing
#方法2：使用正则表达式，-E
grep -E "hello world|mailx" -r /Users/cmm/test4MingLing
#方法3：使用正则表达式，egrep，同grep -E等效
egrep "hello world|mailx" -r /Users/cmm/test4MingLing --color
#输出结果均相同：
/Users/cmm/test4MingLing/text1.txt:hello world
/Users/cmm/test4MingLing/text1.txt:mailx
/Users/cmm/test4MingLing/text2.txt:hello world,this is for grep test
/Users/cmm/test4MingLing/text3.txt:hello world
```

（3）多模式匹配，模式之间为“逻辑与”的关系，匹配所有模式。
基本思想大致相同：先匹配一个模式，然后grep下面一个模式，将前一次grep的结果作为要查询文件路径依次向后传递。

PS：注意：前一次grep的结果必须加上 -l 选项，否则会把匹配成功的文件内容作为要查询的文件名向后传递。

我希望做到的是能够显示出同时包含模式的文件，并且跟随显示匹配每个模式的内容。可惜我现在找到的方法都仅仅能够显示匹配最后一个模式的内容。

```java
#方法1：将其按一次的结果重定向到一个文件，然后 xargs grep 来依次读取。
grep -i "hello world" -rl /Users/cmm/test4MingLing >> temp.txt
cat temp.txt |xargs grep -i "mailx"
#输出结果
/Users/cmm/test4MingLing/text1.txt:mailx

#方法2：直接利用管道
grep -i "hello world" -rl /Users/cmm/test4MingLing | xargs grep -i "mailx"
#输出结果
/Users/cmm/test4MingLing/text1.txt:mailx
```

最后不得不感慨一下linux命令的组合方式，能像积木一样堆积出无穷的组合，是时候展现真正的想象力了。。。

（4）查找指定用户的进程
ps 是查看当前进程的指令，e 表示所有进程，f 表示全格式。

```java
ps -ef |grep "cmm"
ps -ef |grep "cmm"
#输出结果
  502   700     1   0  9:08上午 ??         0:00.03 /Applications/Google Chrome.app/Contents/Versions/70.0.3538.110/Google Chrome Framework.framework/Helpers/crashpad_handler --monitor-self-annotation=ptype=crashpad-handler --database=/Users/cmm/Library/Application Support/Google/Chrome/Crashpad --metrics-dir=/Users/cmm/Library/Application Support/Google/Chrome --url=https://clients2.google.com/cr/report --annotation=channel= --annotation=plat=OS X --annotation=prod=Chrome_Mac --annotation=ver=70.0.3538.110 --handshake-fd=9
    0   838   837   0  9:19上午 ttys001    0:00.02 login -pf cmm
    0  6133   837   0  2:28下午 ttys003    0:00.03 login -pf cmm
  502  7467  6134   0  3:54下午 ttys003    0:00.00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn cmm
```

分类: [Linux](https://www.cnblogs.com/nongzihong/category/1414301.html)

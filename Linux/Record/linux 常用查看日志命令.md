# linux 常用查看日志命令

1、tail 查看文本文件的尾部

```sh
tail -f alert_monitor.log //实时查看alert_monitor.log文件 日志
tail -n 100 alert_monitor.log //查询日志尾部最后100行的日志
tail -n +100 alert_monitor.log // 查询100行之后的所有日志
```

2、cat 查看文件

```sh
cat -n alert_monitor.log
cat -n alert_monitor.log | tail -n +100 | head -n 20 //查询100行之后的日志，且在100行之后里再查前20条日志
```

3、head 查看文本文件的开头部分

```sh
head -n 10 alert_monitor.log //查询日志文件中的头10行日志
head -n -10 alert_monitor.log //查询日志文件除了最后10行的其他所有日志
```

4、sed 管道命令

```sh
sed -n '/2020-04-17 16:17:20/,/2020-04-17 16:20:36/p' alert_monitor.log //查看时间段日志
sed -n '100,200p' alert_monitor.log //查看100行-200行日志
```

5、grep 文本搜索

```sh
grep fuju alert_monitor.log //搜索文件中 fuju
cat -n alert_monitor.log | grep "ERROR" //查询关键字ERROR的日志
```

6、more/less 日志特别多，分页查看

```sh
catc -n alert_monitor.log | grep "debug" | more 分页打印了,通过点击空格键翻页
（more与less区别 less更强大，可在多个终端使用，支持上下键前后翻阅）
```


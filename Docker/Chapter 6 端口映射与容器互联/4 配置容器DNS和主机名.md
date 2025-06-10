# 配置容器DNS和主机名

Docker服务启动后会默认启用一个内嵌的DNS服务，来自动解析同一个网络中的容器主机名和地址，如果无法解析，则通过容器内的DNS相关配置进行解析。

## 相关配置文件

容器中主机名和DNS配置信息可以通过三个系统配置文件来管理：`/etc/resolv.conf`、`/etc/hostname`和`/etc/hosts`。

启动一个容器，在容器中使用mount命令可以看到这三个文件挂载信息：

```
$ docker run -it ubuntu
root@75dbd6685305:/# mount
...
/dev/sde on /etc/resolv.conf type ext4 (rw,relatime)
/dev/sde on /etc/hostname type ext4 (rw,relatime)
/dev/sde on /etc/hosts type ext4 (rw,relatime)
...
```

Docker启动容器时，会从宿主机上复制 /etc/resolv.conf文件，并删除掉其中无法连接到的DNS服务器：

```
root@75dbd6685305:/# cat /etc/resolv.conf
nameserver 8.8.8.8
search my-docker-cloud.com
```

/etc/hosts文件中默认只记录了容器自身的地址和名称：

```
root@75dbd6685305:/# cat /etc/hosts
172.17.0.2   75dbd6685305
::1 localhost ip6-localhost ip6-loopback
127.0.0.1    localhost
```

/etc/hostname文件则记录了容器的主机名：

```
root@75dbd6685305:/# cat /etc/hostname
75dbd6685305
```

## 容器内修改配置文件

容器运行时，可以在运行中的容器里直接编辑/etc/hosts、/etc/hostname和/etc/resolve. conf文件。但是这些修改是临时的，只在运行的容器中保留，容器终止或重启后并不会被保存下来，也不会被docker commit提交。

## 通过参数指定

如果用户想要自定义容器的配置，可以在创建或启动容器时利用下面的参数指定，注意一般不推荐与`-net=host`一起使用，会破坏宿主机上的配置信息：

- 指定主机名`-h HOSTNAME`或者`--hostname=HOSTNAME`：设定容器的主机名。容器主机名会被写到容器内的`/etc/hostname`和`/etc/hosts`。但这个主机名只有容器内能中看到，在容器外部则看不到，既不会在docker ps中显示，也不会在其他容器的`/etc/hosts`中看到；
- `--link=CONTAINER_NAME:ALIAS`：记录其他容器主机名。在创建容器的时候，添加一个所连接容器的主机名到容器内/etc/hosts文件中。这样，新建容器可以直接使用主机名与所连接容器通信；
- `--dns=IP_ADDRESS`：指定DNS服务器。添加DNS服务器到容器的/etc/resolv.conf中，容器会用指定的服务器来解析所有不在/etc/hosts中的主机名；
- `--dns-option list`：指定DNS相关的选项；
- `--dns-search=DOMAIN`：指定DNS搜索域。设定容器的搜索域，当设定搜索域为.example.com时，在搜索一个名为host的主机时，DNS不仅搜索host，还会搜索host.example.com。

# Docker安装Ubuntu

## 获取ubuntu镜像

```sh
$ docker pull ubuntu
```

## 创建容器并运行

```sh
# ssh默认的端口为22,我们将docker中centos的22端口映射到宿主机的8888端口

$ docker run -it -d -p 8888:22 -v D:\sources:/sources/common --name myserver ubuntu /bin/bash
```

进入容器

```sh
$ docker exec -it myserver /bin/bash
```

## 修改用户密码

修改 root 用户密码：

```bash
$ passwd root
New password:
Retype new password:
passwd: password updated successfully
```

## 将Ubuntu的软件源更改为国内源

软件源：

```ini
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
```

将其写入 D:\sources 文件中，在容器中的终端的命令窗口中输入下面的命令：

```sh
cp /etc/apt/sources.list /etc/apt/sources.list.bcakup
```

先将需要配置的文件 sources.list 拷贝更名为 sources.list.bcakup

更改软件源：

```bash
cp /sources/common /etc/apt/sources.list
```

查看软件源是否更改：

```bash
cat /etc/apt/sources.list
```

## 开启ssh远程连接

### 执行更新

```bash
apt-get update
```

### 安装ssh-client、ssh-server

```bash
apt-get install openssh-client openssh-server
```

### 编辑sshd_config文件

将 sshd_config 文件内容转移到/sources/common 文件中：

```bash
cp /etc/ssh/sshd_config /sources/common
```

修改/sources/common 文件中的`#PermitRootLogin prohibit-password`的一行为`PermitRootLogin yes`

将修改后的/sources/common 文件复制到/etc/ssh/sshd_config 文件中：

```bash
cp /sources/common /etc/ssh/sshd_config
```

### 重启ssh服务

```bash
service ssh restart
```

### 安装net-tools工具包

```bash
apt-get install net-tools
```

### ssh连接

启动 ssh 服务

```text
service ssh start
```

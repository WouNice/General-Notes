# Docker安装centos8

## 获取centos8

```sh
docker pull centos
```

## 创建容器并运行

```sh
ssh默认的端口为22,我们将docker中centos的22端口映射到宿主机的8888端口
docker run -d -p 8888:22 --name centos8 --privileged=true centos /usr/sbin/init
```

进入容器

```sh
docker exec -it centos8 /bin/bash
```

## 安装常用工具

1. yum更新
   ```sh
   yum -y update
   ```
2. service安装
   ```sh
   yum install initscripts
   ```
3. ifconfig安装
   ```sh
   yum install net-tools
   ```
4. ssh安装
   ```sh
   sshd rpm -qa | grep ssh
   yum install openssh-server
   service sshd restart
   #查看是否启动22端口
   netstat -antp | grep sshd
   ```
5. 安装passwd
   ```sh
   yum install passwd
   ```
6. 安装防火墙
   ```sh
   yum install firewalld
   ```

## 开启ssh远程连接

1、查看宿主机ip：使用`ipconfig /all`命令，获取IPv4地址

2、防火墙设置：

- 启动防火墙：`systemctl start firewalld`
- 设置开机启动：`systemctl enable firewalld`

防火墙中加入ssh服务，添加到work zone：

   ```sh
   firewall-cmd --zone=work --add-service=ssh
   ```

3、修改`sshd_config`为密码登录：`vim /etc/ssh/sshd_config`，打开注释 `PermitRootLogin yes`，允许密码登录，保存退出

4、设置root用户密码

   ```sh
   passwd root
   ```

5、使用ssh工具远程登录

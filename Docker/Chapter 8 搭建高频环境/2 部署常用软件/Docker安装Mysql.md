# Docker安装Mysql

## 获取Mysql

拉取镜像

```sh
docker pull mysql
```

## 创建容器并运行

```sh
docker run -d -it --name mysql --restart=always -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -e TZ=Asia/Shanghai mysql
```

- MYSQL_ROOT_PASSWORD：设置密码
- TZ：设置容器的默认时区

## 连接Mysql服务器

```sh
docker exec -it mysql bash
```

## 进入Mysql命令行

```sh
$ docker exec -it mysql mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
......
```

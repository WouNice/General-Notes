# Docker安装Mongodb

## 获取Mongodb

```sh
docker pull mongo
```

## 创建容器并运行

```sh
docker run -it -d --name mongo -p 27017:27017 mongo --auth
```

- `–auth`：需要用户名密码访问容器服务

## 连接Mongodb服务器

进入容器：

```sh
docker exec -it mongo mongo admin
```

创建 roo 用户，配置用户名和密码数据库：

```bash
> db.createUser({ user:'root',pwd:'123456',roles:[ { role:'root', db: 'admin'}]});
```

认证身份：

```text
> db.auth('root', '123456')
1
#返回1则代表认证成功
```

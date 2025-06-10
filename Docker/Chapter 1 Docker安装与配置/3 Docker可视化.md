# Docker可视化

## Portainer

Portainer是一款Docker可视化管理工具，允许我们在网页中方便的查看和管理Docker容器。

官方网站：https://www.portainer.io/

下载镜像：

```shell
docker pull portainer/portainer-ce
```

创建数据卷：

```shell
docker volume create portainer_data
```

创建并运行portainer容器：

```shell
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```

访问 http://127.0.0.1:9000 对Docker进行管理。

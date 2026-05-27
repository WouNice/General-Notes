# Docker可视化

## Portainer

Portainer 是一款 Docker 可视化管理工具，允许我们在网页中方便的查看和管理 Docker 容器。

官方网站：https://www.portainer.io/

下载镜像：

```sh
docker pull portainer/portainer-ce
```

创建数据卷：

```sh
docker volume create portainer_data
```

创建并运行 portainer 容器：

```sh
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```

访问 http://127.0.0.1:9000 对 Docker 进行管理。

# Docker的配置

参考：[在 Windows 中配置 Docker ](https://docs.microsoft.com/zh-cn/virtualization/windowscontainers/manage-docker/configure-docker-daemon)

## 配置镜像加速器

> 由于镜像服务可能出现宕机，建议同时配置多个镜像。

```text
{
    "registry-mirrors": [
        "https://dockercf.jsdelivr.fyi",
        "https://docker.jsdelivr.fyi",
        "https://dockertest.jsdelivr.fyi",
        "https://dockerproxy.net",
        "https://mirror.iscas.ac.cn",
        "https://docker.melikeme.cn",
        "https://docker.zhai.cm",
        "https://docker.xiaogenban1993.com/",
        "https://docker.mybacc.com",
        "https://docker.xuanyuan.me/",
        "https://docker.yomansunter.com",
        "https://lispy.org/",
        "https://docker.hlmirror.com/",
        "https://a.ussh.net"
    ],
    "debug": true,
    "experimental": false
}
```

镜像备用：

-   [Docker Proxy 镜像加速](https://dockerproxy.link/)
-   [Docker 镜像加速站](https://docker.melikeme.cn/)
-   [CF-Workers-docker.io](https://github.com/cmliu/CF-Workers-docker.io)
-   [Dockerhub 镜像加速说明](https://docker.zhai.cm/)

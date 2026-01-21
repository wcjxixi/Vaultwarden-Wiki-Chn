# 1.构建您自己的镜像

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Building-your-own-docker-image)
{% endhint %}

克隆库，然后从库的根目录运行以使用默认的 SQLite 后端进行构建：

## 推荐的构建方法 <a href="#recommended-way-to-build" id="recommended-way-to-build"></a>

请阅读 docker 目录中提供的文档，了解有关如何在本地构建 Vaultwarden 的最新信息。

可以通过 docker 或 podman 来完成。请参阅：[https://github.com/dani-garcia/vaultwarden/tree/main/docker](https://github.com/dani-garcia/vaultwarden/tree/main/docker)。

## 简单的构建方法 <a href="#simple-ways-to-build" id="simple-ways-to-build"></a>

```bash
# 构建支持所有数据库的 docker 镜像：
docker buildx build -t vaultwarden .
```

要使用 SQLite 后端构建，只需运行：

```shell
# 构建 docker 镜像
docker buildx build -t vaultwarden --build-arg DB=sqlite .
```

要使用 MySQL 后端构建，只需运行：

```shell
# 构建 docker 镜像
docker buildx build -t vaultwarden --build-arg DB=mysql .
```

要使用 Postgresql 后端构建，只需运行：

```shell
# 构建 docker 镜像
docker buildx build -t vaultwarden --build-arg DB=postgresql .
```

在 docker-compose.yml 中它看起来像这样：

```yaml
  vaultwarden:
    image: vaultwarden
    build:
      context: vaultwarden
      args:
        DB: postgresql
```

# 1.构建您自己的镜像

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Building-your-own-docker-image)
{% endhint %}

克隆库，然后从库的根目录运行以使用默认的 SQLite 后端进行构建：

```shell
# 构建 docker 镜像
docker build -t vaultwarden .
```

要使用 MySQL 后端构建，运行：

```shell
# 构建 docker 镜像
docker build -t vaultwarden --build-arg DB=mysql .
```

要使用 Postgresql 后端构建，运行：

```shell
# 构建 docker 镜像
docker build -t vaultwarden --build-arg DB=postgresql .
```

在 docker-compose.yml 中它看起来像这样：

```javascript
  vaultwarden:
    # image: vaultwarden/server-postgresql:latest
    image: vaultwarden
    build: 
      context: vaultwarden
      args: 
        DB: postgresql
```

# \*使用非 root 用户运行 docker 容器

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Running-docker-container-with-non-root-user)
{% endhint %}

默认情况下，`vaultwarden/server` 使用 root 用户在容器内运行服务。如果您想以非 root 用户身份运行容器，则需要进行一些设置：

1、确保您在容器内挂载的目录可供用户写入。例如，如果您决定以 `nobody` 身份运行，那么此目录就必须是 id 为 `65534` 的用户可以写入的。有关在容器内指定用户的其他方法，请参阅 [docker 文档](https://docs.docker.com/engine/reference/run/#user)。这里的示例中，我们将使用 `nobody`。

```bash
# 在主机上创建目录，将其更改为您的首选路径
sudo mkdir /vw-data

# 使用用户 ID 设置所有者。
# 请注意，所有权必须与容器内的 /etc/passwd 中的用户一致，而不是主机上的用户
sudo chown 65534 /vw-data

# 授予所有者对该文件夹的全部权限
sudo chmod u+rwx /vw-data
```

2、使用合适的参数启动容器。定义用户并确保启动时端口设置为 `1024` 或更高。

```bash
docker run -d \
  --name vaultwarden \
  --user nobody \
  -e ROCKET_PORT=1024 \
  -v /vw-data/:/data/ \
  -p 80:1024 \
  vaultwarden/server:latest
```

请注意，端口映射 (`-p 80:1024`) 反映了 `ROCKET_PORT` 设置。

另一种方法可能是 `CAP_NET_BIND_SERVICE`，它允许以非 root 用户身份绑定到低于 `1024` 的端口。

```
cap_add:
  - CAP_NET_BIND_SERVICE
user: nobody
```

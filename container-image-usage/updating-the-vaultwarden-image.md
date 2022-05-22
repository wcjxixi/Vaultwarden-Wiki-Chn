# 3.更新 Vaultwarden 镜像

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Updating-the-vaultwarden-image)
{% endhint %}

更新非常简单，你只需确保保留了已挂载的卷。如果您使用[此处](starting-a-container.md)示例中的 bind-mounted 路径（绑定挂载路径）的方式，则只需使用 `pull` 拉取最新版的镜像，使用 `stop` 和 `rm` 停止和移除当前容器，然后与之前相同的方式启动一个新的容器即可：

```batch
# 拉取最新版本的镜像
docker pull vaultwarden/server:latest

# 停止并移除旧版本容器
docker stop vaultwarden
docker rm vaultwarden

# 使用已挂载的数据启动容器
docker run -d --name vaultwarden -v /vw-data/:/data/ -p 80:80 vaultwarden/server:latest
```

然后访问 [http://localhost:80](http://localhost/)

如果您没有为持久性数据绑定挂载卷，则多一个中间步骤，就是使用中间容器来保留数据：

```docker
# 拉取最新版本的镜像
docker pull vaultwarden/server:latest

# 创建中间容器以保留数据
docker run --volumes-from vaultwarden --name vaultwarden_data busybox true

# 停止并移除旧版本容器
docker stop vaultwarden
docker rm vaultwarden

# 使用已挂载的数据启动容器
docker run -d --volumes-from vaultwarden_data --name vaultwarden -p 80:80 vaultwarden/server:latest

# 移除中间容器（可选）
docker rm vaultwarden_data

# 您可以保留数据容器以用于将来的更新，这样的话，可以跳过最后的移除中间容器这一步。
```

你也可以使用 [Watchtower](https://containrrr.dev/watchtower/) 这样的工具来自动化更新过程。Watchtower 可以定期检查 Docker 镜像的更新，拉取更新后的镜像，并使用更新后的镜像重新创建容器。

## 使用 docker-compose 时的更新 <a href="#updating-when-using-docker-compose" id="updating-when-using-docker-compose"></a>

```docker
docker-compose down
docker-compose pull
docker-compose up -d
```

## 使用 systemd 服务时的更新（在本例中为 Debian/Raspbian） <a href="#updating-when-using-systemd-service-in-this-case-debian-raspbian" id="updating-when-using-systemd-service-in-this-case-debian-raspbian"></a>

```batch
sudo systemctl restart vaultwarden.service
sudo docker system prune -f
# 警告！这将删除已停止或未使用的容器，例如与 Vaultwarden 无关的容器
# 请仔细查看哪个容器是你需要的

docker ps -a
# 查看已停止的容器

#WARNING! This will remove:
#        - all stopped #containers
#        - all networks not used by at least one container
#        - all dangling images
#        - all dangling build cache
# 使用以下命令列出所有 Docker 镜像
docker images
# 这里你将看到所有未使用的镜像
#
```

`restart` 命令将会依次停止容器、提取最新镜像、然后运行容器。`prune` 命令将会移除当前较旧的容器（`-f` 表示不需要确认）。

如果需要，可以将它们放入 cronjob 中以计划任务自动运行（根据您的需要修改时间）：

```python
$ sudo crontab -e
0 2 * * * sudo systemctl restart vaultwarden.service

0 3 * * * sudo /usr/bin/docker system prune -f
```

如果 `/usr/bin/docker` 不是 docker 的正确路径，可以使用 `which docker` 命令查看它的实际路径。

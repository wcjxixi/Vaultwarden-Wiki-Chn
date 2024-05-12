# 3.更新 Vaultwarden 镜像

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Updating-the-vaultwarden-image)
{% endhint %}

更新非常简单，您只需确保保留了已挂载的卷。如果您使用[此处](starting-a-container.md)示例中的 bind-mounted 路径（绑定挂载路径）的方式，则只需使用 `pull` 拉取最新版本的镜像，再使用 `stop` 和 `rm` 来停止和移除当前容器，然后与之前相同的方式启动一个新的容器即可：

```shell
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

```shell
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

您也可以使用 [Watchtower](https://containrrr.dev/watchtower/) 这样的工具来自动化更新过程。Watchtower 可以定期检查 Docker 镜像的更新，拉取更新后的镜像，并使用更新后的镜像重新创建容器。

## 使用 Docker Compose 时的更新 <a href="#updating-when-using-docker-compose" id="updating-when-using-docker-compose"></a>

```shell
docker compose pull # 或者 `docker-compose pull` 如果使用后独立的 Docker Compose 的话
docker compose up -d # 或者 `docker-compose up -d` 如果使用后独立的 Docker Compose 的话
```

## 使用 systemd 服务时的更新（在本例中为 Debian/Raspbian） <a href="#updating-when-using-systemd-service-in-this-case-debian-raspbian" id="updating-when-using-systemd-service-in-this-case-debian-raspbian"></a>

```shell
sudo systemctl restart vaultwarden.service
sudo docker system prune -f
# 警告！这将删除已停止或未使用的容器，例如与 Vaultwarden 无关的容器
# 请仔细查看哪个容器是您需要的

docker ps -a
# 查看已停止的容器

#WARNING! This will remove:
#        - all stopped #containers
#        - all networks not used by at least one container
#        - all dangling images
#        - all dangling build cache
# 使用以下命令列出所有 Docker 镜像
docker images
# 这里您将看到所有未使用的镜像
#
```

`restart` 命令将会依次停止容器、提取最新镜像、然后运行容器。`prune` 命令将会移除当前较旧的容器（`-f` 表示不需要确认）。

如果需要，可以将它们放入 cronjob 中以计划任务自动运行（根据您的需要修改时间）：

```shell
$ sudo crontab -e
0 2 * * * sudo systemctl restart vaultwarden.service

0 3 * * * sudo /usr/bin/docker system prune -f
```

如果 `/usr/bin/docker` 不是 docker 的正确路径，可以使用 `which docker` 命令查看它的实际路径。

## 使用 DietPi 时的更新 <a href="#updating-when-using-dietpi" id="updating-when-using-dietpi"></a>

[DietPi](https://dietpi.com/) 是一个轻量级的基于 Debian 的发行版（镜像），适用于各种设备，例如 Raspberry Pi、Odroid、NanoPi 等。它提供了一个软件脚本，用于安装包括 Vaultwarden 在内的各种程序。这样可以让用户免去对安装命令的烦恼。

Vaultwarden 的更新必须由用户在 DietPi 上手动启动，没有自动安装方式，运行 `apt update && apt upgrade` 也不会执行更新。要更新之前使用 DietPi 的软件安装脚本安装的 Vautwarden 实例，请在 DietPi 的命令行中输入以下命令：

```batch
dietpi-software reinstall 183
```

建议使用 DietPi 8.7 或更新版本，因为与以前的版本相比，更新过程已大大加快。

如果您自定义了 Vaultwarden 的配置文件，它将被更新脚本保留。

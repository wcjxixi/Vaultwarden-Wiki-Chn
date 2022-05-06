# \*17.使用 systemd docker 运行

{% hint style="success" %}
对应的官方页面地址（Vaultwarden WiKi 已移除此页面）
{% endhint %}

这部分的内容允许您使用 systemd 来管理 Docker 容器的生命周期，若你喜欢的话。

首先，使用系统包管理器安装 `systemd-docker` 包。这是一个用于改进 docker 与 systemd 集成的封装器。

有关完整介绍和配置选项，请参阅 [Github 仓库](https://github.com/ibuildthecloud/systemd-docker)。

以 root 身份，使用你喜欢的编辑器用以下内容创建 `/etc/systemd/system/vaultwarden.service` 文件：

```python
[Unit]
Description=Vaultwarden
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=0
ExecStartPre=-/usr/bin/docker pull vaultwarden/server:latest
ExecStartPre=-/usr/bin/docker stop vaultwarden
ExecStartPre=-/usr/bin/docker rm vaultwarden
ExecStart=/usr/bin/docker run \
  -p 8080:80 \
  -p 8081:3012 \
  --env-file /opt/.vaultwarden.env \
  -v /opt/vw-data:/data/ \
  --rm --name vaultwardenvaultwarden/server:latest
ExecStopPost=-/usr/bin/docker rm vaultwarden
Restart=Always
RestartSec=30s
Type=notify
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

根据需要调整上述示例。特别要注意 `-p` 和 `-v` 选项，因为它们控制着容器和主机之间的端口和卷绑定。另外，请确保为您的配置提供一个 `--env-file`，或者直接通过 `-e KEY=VALUE` 输入您的所有配置。

对上述选项的解释：

* `TimeoutStartSec` 的值 `0`：等待默认启动时间后，认为服务已经失败，将停止 systemd。此为必选项，因为 `ExecStartPre` 中的 `docker pull` 命令需要一段时间来完成。
* `ExecStartPre`：在运行之前拉取 docker 标签。
* `ExecStopPost`：删除容器（以确保我们下次可以重新启动）。我们这样做的原因是 systemd 监控的是 docker 服务而不是单个容器。因此，我们使用 `unless-stopped` 告诉 docker 服务重启容器。这基本上就像 `--restart=Always`，但不包括 docker 服务停止的时候（或者容器被挂起）。当 docker 服务停止时，这允许我们使用 `Restart=Always` 让 systemd 仅重启服务。
* `Type` 的值 `notify`：告诉 systemd 从已准备就绪的服务中获取通知。
* `NotifyAccess` 的值 `all`：是由 `systemd-docker` 请求的。

## 设置环境变量 <a href="#setting-environment-variables" id="setting-environment-variables"></a>

可以通过两种方式在单元文件中直接指定环境变量：

* 在 `[Service]` 块中使用 `Environment` 指令。
* 使用 `docker` 的 `-e` 选项。此时，您可以省略上面示例中显示的 `--env` 选项。

要验证是否正确设置了环境变量，请检查 `systemctl show vaultwarden.service` 的输出中是否存在 `Environment` 行。

也可以在单元文件中使用 `EnvironmentFile` 指令将环境变量存储在单独的文件中。在这种情况下，请如上面示例中所示在 docker 命令行中设置 `--env` 选项，否则将不会处理环境文件。

systemd 可以获取以下格式的文件：

```python
Key="Value"
```

您可以在此[环境示例模版](https://github.com/dani-garcia/bitwarden\_rs/blob/21325b7523a68ab3ae8d435ab5b73176db6155ff/.env.template)中找到更多关于环境设置和语法的说明。

但是，systemd 项目并没有规定该文件的存储位置。有关此文件的最佳存储位置，请查阅发行版文档。例如，基于 RedHat 的发行版通常将这些文件放在 `/etc/sysconfig/` 中。

如果你不确定，只需使用 root 权限在 `/etc/` 中创建一个文件即可，比如 `/etc/vaultwarden.service.conf`。

在您的单元文件中的 `[Service]` 块中添加一个 `EnvironmentFile` 指令，其值是上面创建的文件的完整路径。例如：

```python
[Unit]
Description=Vaultwarden
After=docker.service
Requires=docker.service

[Service]
EnvironmentFile=/etc/vaultwarden.service.conf
TimeoutStartSec=0
-snip-
```

## 运行服务 <a href="#running-the-service" id="running-the-service"></a>

完成上述安装和配置后，使用 `sudo systemctl daemon-reload` 命令重新加载systemd 。然后，使用 `sudo systemctl start vaultwarden` 命令启动 Vaultwarden 服务。

要使服务跟随系统启动，请使用 `sudo systemctl enable vaultwarden`。

使用 `systemctl status vaultwarden` 来验证容器是否已经启动。

如果在启动服务时遇到 `json: cannot unmarshal object into Go value of type string` 错误，则应使用最新版本的 Go 来自己编译 systemd-docker 二进制，请参阅此[话题](https://github.com/ibuildthecloud/systemd-docker/issues/50)。

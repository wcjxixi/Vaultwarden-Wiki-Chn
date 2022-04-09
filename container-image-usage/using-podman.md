# 5.使用 Podman

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Using-Podman)
{% endhint %}

[Podman](https://podman.io) 是替代 Docker 的无守护程序，它与大部分 Docker 容器兼容。

## 创建一个 systemd 服务文件 <a href="#creating-a-systemd-service-file" id="creating-a-systemd-service-file"></a>

由于 Podman 的无守护程序架构，因此它比 Docker 更容易在 systemd 中运行。它带有一个便捷的 [generate syetemd 命令](http://docs.podman.io/en/latest/markdown/podman-generate-systemd.1.html)，该命令可以生成 systemd 文件，这里有[一篇不错的文章详细介绍了它](https://www.redhat.com/sysadmin/podman-shareable-systemd-services)，还有[这篇文章也详细介绍了一些最新的更新](https://www.redhat.com/sysadmin/improved-systemd-podman)。

```python
$ podman run -d --name vaultwarden -v /vw-data/:/data/:Z -e ROCKET_PORT=8080 -p 8080:8080 vaultwarden/server:latest
54502f309f3092d32b4c496ef3d099b270b2af7b5464e7cb4887bc16a4d38597
$ podman generate systemd --name vaultwarden
# container-foo.service
# 由 Podman 1.6.2 自动生成
# Tue Nov 19 15:49:15 CET 2019

[Unit]
Description=Podman container-foo.service
Documentation=man:podman-generate-systemd(1)

[Service]
Restart=on-failure
ExecStart=/usr/bin/podman start vaultwarden
ExecStop=/usr/bin/podman stop -t 10 vaultwarden
KillMode=none
Type=forking
PIDFile=/run/user/1000/overlay-containers/54502f309f3092d32b4c496ef3d099b270b2af7b5464e7cb4887bc16a4d38597/userdata/conmon.pid

[Install]
WantedBy=multi-user.target default.target
```

您可以提供一个 `--files` 标志来专用于特定文件，以将 systemd 服务文件输出到该文件。这样，我们就可以将容器作为任何常规服务文件来启用和启动。

```python
$ systemctl --user enable /etc/systemd/system/container-vaultwarden.service
$ systemctl --user start container-vaultwarden.service
```

### 每次重启时新建容器 <a href="#new-container-every-restart" id="new-container-every-restart"></a>

如果我们希望每次服务启动时都创建一个新的容器，我们可以编辑服务文件以包含以下内容：

```python
[Unit]
Description=Podman container-vaultwarden.service

[Service]
Restart=on-failure
ExecStartPre=/usr/bin/rm -f /%t/%n-pid /%t/%n-cid
ExecStart=/usr/bin/podman run --conmon-pidfile /%t/%n-pid --cidfile /%t/%n-cid --env-file=/home/spytec/Vaultwarden/vaultwarden.conf -d -p 8080:8080 -v /home/spytec/Vaultwarden/vw-data:/data/:Z vaultwarden/server:latest
ExecStop=/usr/bin/podman stop -t "15" --cidfile /%t/%n-cid
ExecStop=/usr/bin/podman rm -f --cidfile /%t/%n-cid
KillMode=none
Type=forking
PIDFile=/%t/%n-pid

[Install]
WantedBy=multi-user.target default.target
```

环境文件 `vaultwarden.conf` 可以包含你需要的容器的所有环境值，比如：

```python
ROCKET_PORT=8080
```

如果您希望容器拥有特定名称，则需要添加 `ExecStartPre=/usr/bin/podman rm -i -f vaultwarden`，如果进程未被正确清理的话。注意，此方式当前无法与具有 `User=` 选项的用户一起正常工作（见 [https://github.com/containers/podman/issues/5572](%20https:/github.com/containers/podman/issues/5572/)）。

## 故障排除 <a href="#troubleshooting" id="troubleshooting"></a>

### 调试 systemd 服务文件 <a href="#debugging-systemd-service-file" id="debugging-systemd-service-file"></a>

如果主机出现故障或容器崩溃，则 systemd 服务文件应自动停止现有容器并将其重新启动。可以通过 `journalctl --user -u container-vaultwarden -t 100` 来定位错误。

在大多数情况下，我们可以通过简单地增加服务文件中的 podman 命令的超时来解决我们看到的错误。

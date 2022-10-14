# 16.设置为 systemd 服务

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Setup-as-a-systemd-service)
{% endhint %}

这部分的内容要求您已经[编译了 Vaultwarden 二进制](../deployment/building-binary.md)。如果您已生成了 docker 镜像，则需要查看[使用 systemd-docker 运行](running-with-systemd-docker.md)。

## 设置 <a href="#setup" id="setup"></a>

要使 Vaultwarden 在系统启动的时候启动并使用 systemd 的其他功能（例如，隔离、日志记录等），则需要一个 `.service` 文件。以下是一个可行的起点：

```systemd
[Unit]
Description=Vaultwarden Server (Rust Edition)
Documentation=https://github.com/dani-garcia/vaultwarden
# 如果您使用 mariadb、mysql 或 postgresql 数据库， 
# 您必须像下面这样添加它们，并去掉前面的 # 以取消注释。
# 这将确保您的数据库服务器在 Vaultwarden 之前启动 ("After")，
# 并且在启动 Vaultwarden 之前成功启动 ("Requires")。

# 仅 sqlite
After=network.target

# MariaDB
# After=network.target mariadb.service
# Requires=mariadb.service

# Mysql
# After=network.target mysqld.service
# Requires=mysqld.service

# PostgreSQL
# After=network.target postgresql.service
# Requires=postgresql.service


[Service]
# 设置 Vaultwarden 用户/群组。此用户/群组对工作目录（见下文）允许有读写权限
User=vaultwarden
Group=vaultwarden
# 使用环境文件进行配置
EnvironmentFile=/etc/vaultwarden.env
# 已编译的二进制的位置
ExecStart=/usr/bin/vaultwarden
# 设置合理的连接和进程限制
LimitNOFILE=1048576
LimitNPROC=64
# 将 bitwarden_rs 与系统的其他部分隔离开
PrivateTmp=true
PrivateDevices=true
ProtectHome=true
ProtectSystem=strict
# 仅允许对以下目录进行写入，并将其设置为工作目录（用户和密码数据存储在这里）
WorkingDirectory=/var/lib/vaultwarden
ReadWriteDirectories=/var/lib/vaultwarden
# 允许 bitwarden_rs 绑定 0-1024 范围内的端口
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```

更改以上所有路径以匹配您的安装（`WorkingDirectory` 与 `ReadWriteDirectory` 应相同），将此文件命名为 `vaultwarden.service` 并将其放入 `/etc/systemd/system` 中。

如果必须更改现有（而不是像上面那样新建）的 systemd 文件（您安装的软件包提供给您的），可以使用下面的命令来添加更改：

```shell
$ sudo systemctl edit vaultwarden.service
```

请运行以下命令，以让 systemd 知道您的新文件或您所做的任何更改：

```shell
$ sudo systemctl daemon-reload
```

## 用法 <a href="#usage" id="usage"></a>

要启动此「服务」，请运行：

```shell
$ sudo systemctl start vaultwarden.service
```

要启用自动启动，请运行：

```shell
$ sudo systemctl enable vaultwarden.service
```

同理，您可以使用 `stop`、`restart` 和 `disable` 来停止、重启或禁用此服务。

### 更新 Vaultwarden <a href="#updating-bitwarden_rs" id="updating-bitwarden_rs"></a>

编译新版本的 Vaultwarden 之后，您可以复制已编译的（新的）二进制文件并替换现有的（旧的）二进制文件，然后重新启动服务：

```shell
$ sudo systemctl restart vaultwarden.service
```

### 卸载 Vaultwarden <a href="#uninstalling-bitwarden_rs" id="uninstalling-bitwarden_rs"></a>

在执行其他操作之前，应先停止并禁用该服务：

```shell
$ sudo systemctl disable --now vaultwarden.service
```

然后，您可以删除二进制、环境文件、web-vault 文件夹（如果已安装）以及用户数据（如果需要）。请记住，还要删除专门创建的用户、群组和防火墙规则（如果需要）和 systemd 文件。

删除 systemd 文件后，您应该通过下面的方式使 systemd 意识到这一点：

```shell
$ sudo systemctl daemon-reload
```

### 查看日志和状态 <a href="#logging-and-status-view" id="logging-and-status-view"></a>

如果要查看日志输出，请运行：

```shell
$ journalctl -u vaultwarden.service
```

或查看此服务的更简洁的状态，请运行：

```shell
$ systemctl status vaultwarden.service
```

## 故障排除 <a href="#troubleshooting" id="troubleshooting"></a>

### 旧版 systemd 的沙盒选项 <a href="#sandboxing-options-with-older-systemd-versions" id="sandboxing-options-with-older-systemd-versions"></a>

在 RHEL 7（以及 debian 8）中，使用的 systemd 不支持某些隔离选项 ([#445](https://github.com/dani-garcia/bitwarden\_rs/issues/445), [#363](https://github.com/dani-garcia/bitwarden\_rs/issues/363))。这可能导致出现如下错误：

```
Failed at step NAMESPACE spawning /home/vaultwarden/vaultwarden: Permission denied
```

或者：

```
Failed to parse protect system value
```

要解决这一点，您可以在包含有 `PrivateTmp`、`PrivateDevices`、`ProtectHome`、`ProtectSystem` 和 `ReadWriteDirectories` 的部分或全部行前面放置 `#` 符号来将其注释掉。尽管将所有这些行注释掉可能会起作用，但不建议这样做，因为这些都是很好的安全措施。要查看您的 systemd 支持哪些选项，请运行以下命令来查看其输出：

```shell
$ systemctl --version
```

检查您的 systemd 版本并与 [systemd/NEWS.md](https://github.com/systemd/systemd/blob/master/NEWS) 进行比较。

编辑 `.service` 文件后，请不要忘记在启动（或重启）服务之前运行如下命令：

```shell
$ sudo systemctl daemon-reload
```

### 服务无法启动 <a href="#service-fails-to-start" id="service-fails-to-start"></a>

systemd journal (`journalctl -eu vaultwarden.service`) 中显示以下错误：

```
Feb 18 05:29:10 staging-bitwarden systemd[1]: Started Vaultwarden Server (Rust Edition).
Feb 18 05:29:10 staging-bitwarden systemd[49506]: vaultwarden.service: Failed to execute command: Resource temporarily unavailable
Feb 18 05:29:10 staging-bitwarden systemd[49506]: vaultwarden.service: Failed at step EXEC spawning /usr/bin/vaultwarden: Resource temporarily unavailable
Feb 18 05:29:10 staging-bitwarden systemd[1]: vaultwarden.service: Main process exited, code=exited, status=203/EXEC
Feb 18 05:29:10 staging-bitwarden systemd[1]: vaultwarden.service: Failed with result 'exit-code'.
```

已知当 Vaultwarden 在容器（LXC 等）内部或本地运行时，会出现这种情况。服务文件中的参数 `LimitNPROC=64` 使服务无法启动。注释掉此参数后，服务可以正常启动。

**注意**：systemd 覆盖文件不起作用，必须注释/删除该行。最简单的方法是通过

```shell
# systemctl edit --full vaultwarden.service
```

然后重新加载守护程序并重新启动。

### 环境变量未被加载 <a href="#environment-variable-its-not-loaded" id="environment-variable-its-not-loaded"></a>

请注意，systemd 不支持 EnvironmentFile=/etc/vaultwarden.env 文件中的注释与变量在同一行中（参阅 [#1607](https://github.com/dani-garcia/vaultwarden/issues/1607)。比如下面这个 `.env` 文件示例中，变量 `WEBSOCKET_ENABLED` 将不会被加载：

```systemd
ROCKET_PORT=XXXX
WEBSOCKET_ENABLED=true # enable websocket
```

来源：[#1607](https://github.com/dani-garcia/vaultwarden/issues/1607)

## 更多信息 <a href="#more-information" id="more-information"></a>

有关 .service 文件的更多信息，请参阅 [systemd.service](https://www.freedesktop.org/software/systemd/man/systemd.service.html) 和 [systemd.exec](https://www.freedesktop.org/software/systemd/man/systemd.exec.html)（用于安全性配置）手册页。

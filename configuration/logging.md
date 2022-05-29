# 15.日志记录

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Logging)
{% endhint %}

默认情况下，Vaultwarden 仅记录到[标准输出](https://zh.wikipedia.org/wiki/%E6%A8%99%E6%BA%96%E4%B8%B2%E6%B5%81) (stdout)。您也可以将它配置为记录到文件。

## 记录到文件 <a href="#logging-to-a-file" id="logging-to-a-file"></a>

从 1.5.0 版本开始支持记录到文件。您可以使用 `LOG_FILE` 环境变量来指定日志文件的路径：

```shell
docker run -d --name vaultwarden \
...
  -e LOG_FILE=/data/vaultwarden.log \
...
```

当设置此环境变量时，日志消息将被记录到标准输出和日志文件中。如果您在 Docker 中运行，则需要使用从 Docker 主机挂载的文件路径（如 `data` 文件夹）；否则，如果容器被重新启动或移除，您的日志文件将丢失（或至少难以找回）。

## 更改日志级别 <a href="#change-the-log-level" id="change-the-log-level"></a>

为了减少日志消息的数量，您可以将日志级别设置为 warn（默认为 info）。[日志级别](https://docs.rs/log/0.4.7/log/enum.Level.html#variants)可使用 `LOG_LEVEL` 环境变量进行调整，同时还需要设置 `EXTENDED_LOGGING=true`。注意：使用日志级别 warn 或 error 仍然可以让 [Fail2Ban](security/fail2ban-setup.md) 正常工作。

`LOG_LEVEL` 选项包括：trace、debug、info、warn、error 以及 off。

```shell
docker run -d --name vaultwarden \
...
  -e LOG_LEVEL=warn -e EXTENDED_LOGGING=true \
...
```

## 查看日志记录 <a href="#viewing-logs" id="viewing-logs"></a>

如果在 Docker 中运行：`docker logs <container-name>`

如果使用 `systemd` 运行：`journalctl -u vaultwarden.service`（或你的服务名称）

否则，请检查标准输出被重定向到的位置，或设置 `LOG_FILE` 环境变量并查看该文件。

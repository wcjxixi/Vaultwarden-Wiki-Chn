# 6.启用 WebSocket 通知

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-WebSocket-notifications)
{% endhint %}

**重要提示**：这不适用于使用推送通知的移动客户端。

要启用 WebSockets 通知，必须使用外部反向代理，并且必须执行以下配置操作：

* 将 `/notifications/hub` 端点路由到 WebSocket 服务器，默认在 `3012` 端口，确保传递 `Connection` 和 `Upgrade` 头。（提示：可以使用 `WEBSOCKET_PORT` 变量来更改端口）
* 将所有其他（包括 `/notifications/hub/negotiate`）路由到标准 Rocket 服务器，默认在 `80` 端口上。
* 如果使用 Docker，则可能还需要使用 `-p` 标识来映射两个端口。

在[代理示例](../deployment/proxy-examples.md)页面查看配置示例。

然后，您需要通过将 `WEBSOCKET_ENABLED` 变量设置为 `true` 以在 Vaultwarden 端启用 WebSocket 协商：

```python
docker run -d --name vaultwarden \
  -e WEBSOCKET_ENABLED=true \
  -v /vw-data/:/data/ \
  -p 80:80 \
  -p 3012:3012 \
  vaultwarden/server:latest
```

提示：有此解决方案的原因是由于 Rocket 缺乏对 WebSockets 的支持（尽管[这是计划的功能](https://github.com/SergioBenitez/Rocket/issues/90)），这迫使我们在单独的端口上启动辅助服务器。

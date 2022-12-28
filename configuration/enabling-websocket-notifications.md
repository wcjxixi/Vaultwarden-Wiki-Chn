# 6.启用 WebSocket 通知

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-WebSocket-notifications)
{% endhint %}

WebSocket 通知用于将发生的一些相关事件通告给 Bitwarden 的浏览器和桌面客户端，例如密码数据库中有条目被修改了或被删除了。收到通知后，客户端可以采取适当的操作，例如重新获取修改的条目，或从其本地数据库副本中移除已删除的条目。在此通知方案中，Bitwarden 客户端与 Bitwarden 服务器（在本案例中为 Vaultwarden）建立持久的 WebSocket 连接。每当服务器有需要报告的事件时，它都会通过此持久连接将其发送给客户端。

请注意，WebSocket 通知不适用于移动 (Android/iOS) Bitwarden 客户端。这些客户端使用原生推送通知服务（Android 为 [FCM](https://firebase.google.com/docs/cloud-messaging)，iOS 为 [APN](https://developer.apple.com/go/?id=push-notifications)）。Vaultwarden 目前不支持向移动客户端推送通知。

要启用 WebSockets 通知，必须使用外部反向代理，并且必须执行以下配置操作：

* 将 `/notifications/hub` 端点路由到 WebSocket 服务器，默认在 `3012` 端口，确保传递 `Connection` 和 `Upgrade` 头。（提示：可以使用 `WEBSOCKET_PORT` 变量来更改端口）
* 将所有其他（包括 `/notifications/hub/negotiate`）路由到标准 Rocket 服务器，默认在 `80` 端口上。
* 如果使用 Docker，则可能还需要使用 `-p` 标识来映射两个端口。

在[代理示例](../deployment/proxy-examples.md)页面查看配置示例。

然后，您需要通过将 `WEBSOCKET_ENABLED` 变量设置为 `true` 以在 Vaultwarden 端启用 WebSocket 协商：

```shell
docker run -d --name vaultwarden \
  -e WEBSOCKET_ENABLED=true \
  -v /vw-data/:/data/ \
  -p 80:80 \
  -p 3012:3012 \
  vaultwarden/server:latest
```

注意：有此解决方案的原因是由于 Rocket 缺乏对 WebSockets 的支持（尽管[这是计划的功能](https://github.com/SergioBenitez/Rocket/issues/90)），这迫使我们在另外的端口上启动辅助服务器。

## 测试 WebSocket 连接 <a href="#test-the-websockets-connection" id="test-the-websockets-connection"></a>

有两种方式可以测试连接是否正常工作：

1. 打开浏览器的开发人员工具，转到网络选项卡然后筛选 `WS`/`WebSockets`。注销或刷新页面并再次登录，您应该会看到升级后的 WebSocket 连接的 101 响应。如果您单击该行，您应该能够看到消息。如果您没有在 `/notifications/hub` 上获得状态代码 101，则表示某些配置不正确。
2. 打开两个不同的浏览器或隐身/隐私窗口。在两个浏览器上登录您的帐户。创建一个新的条目，或者重命名一个条目，在另一个浏览器中应该会立即收到更改。

# 6.启用 WebSocket 通知

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-WebSocket-notifications)
{% endhint %}

WebSocket 通知用于将发生的一些相关事件通告给浏览器、Bitwarden 的桌面和浏览器扩展客户端，例如密码数据库中有条目被修改了或被删除了。收到通知后，客户端可以采取适当的操作，例如刷新已修改的条目，或从其本地缓存中移除已删除的条目。在此通知方案中，Bitwarden 客户端与 Bitwarden 服务器（在本案例中为 Vaultwarden）建立持久的 WebSocket 连接。每当服务器有需要报告的事件时，它都会通过此持久连接将其发送给客户端。

请注意，WebSocket 通知不适用于移动 (Android/iOS) Bitwarden 客户端。这些客户端使用原生推送通知服务（Android 为 [FCM](https://firebase.google.com/docs/cloud-messaging)，iOS 为 [APN](https://developer.apple.com/go/?id=push-notifications)）。这些必须使用 Bitwarden 云服务的推送证书单独配置，自 v1.29.0 起可用。

自 Vaultwarden v1.29.0 起，WebSocket 默认启用。以前的版本需要反向代理，因为 WebSocket 运行在与默认的 HTTPS 端口不同的端口上。

旧的实现在 v1.29.0 中仍然可用，暂时不会在更新期间中断。但将来这将被移除。

如果您确实使用像 nginx 或 Apache HTTPd 这样的反向代理，那么您需要确保正确配置它以传递 WebSocket `Upgrade` 和 `Connection` 标头。一些反向代理默认执行此操作，例如 Traefik。

自 Vaultwarden v1.29.0 版本起，旧的 `WEBSOCKET_ENABLED` 和 `WEBSOCKET_PORT` 已被弃用并将被忽略。在 v1.29.0 版本之后，您可以通过将 `ENABLE_WEBSOCKET` 设置为 `false` 值来禁用 Websocket 通知，这将减少 Vaultwarden 使用的资源（尽管不会太多）。自 v1.31.0 起，已移除对 3012 端口 WebSocket 流量的支持，因为它已集成至主 HTTP 端口。

示例配置包含在[代理示例](../reverse-proxy/proxy-examples.md)中。&#x20;

**请注意，某些示例尚未针对 v1.29.0 进行更新。**

## 测试 WebSocket 连接 <a href="#test-the-websockets-connection" id="test-the-websockets-connection"></a>

有两种方式可以测试连接是否正常工作：

1. 打开浏览器的开发人员工具，转到网络选项卡然后筛选 `WS`/`WebSockets`。注销或刷新页面并再次登录，您应该会看到升级后的 WebSocket 连接的 101 响应。如果您单击该行，您应该能够看到消息。如果您没有在 `/notifications/hub` 上获得状态代码 101，则表示某些配置不正确。消息将显示在浏览器的控制台窗口中：`[2023-12-01T00:00:00.000Z] Information: WebSocket connected to wss://HOST_NAME/notifications/hub?access_token=eyJ0eX......`
2. 打开两个不同的浏览器或隐身/隐私窗口。在两个浏览器上登录您的账户。创建一个新的条目，或者重命名一个条目，在另一个浏览器中应该会立即收到更改。

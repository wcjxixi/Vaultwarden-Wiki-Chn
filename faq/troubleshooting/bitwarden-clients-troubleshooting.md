# 3.Bitwarden 客户端故障排除

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Bitwarden-clients-troubleshooting)
{% endhint %}

## 通用 <a href="#general" id="general"></a>

如果您在使用任何官方 Bitwarden 客户端时遇到任何问题，尝试在任何 Bitwarden 存储库中报告该问题很可能会导致问题被关闭，并声明他们不支持第三方服务器 - 这确实是合理的做法。

然而有时问题确实源于客户端本身，无法通过服务器端修复，因此Vaultwarden也无能为力。为避免用户在Vaultwarden和Bitwarden之间反复推诿，建议遵循以下步骤定位问题根源，这有助于两个项目共同解决问题。这些步骤看似繁琐，但请记住：必须有人采取行动找出问题所在，而最适合执行测试的人，正是最初发现问题的人。

此外，在报告任何客户端（包括 web-vault）问题之前，请务必测试 Vaultwarden 的 `:testing` 标签镜像。因为 `:testing` 标签镜像可能已经包含了对新客户端的修复，但我们尚未将其视为稳定版本发布。

建议在运行 `:testing` 标签镜像之前备份您的数据库，因为它可能会更新或迁移一些数据，而这些数据与旧服务器版本不兼容！

## 如何排查问题 <a href="#how-to-troubleshoot-an-issue" id="how-to-troubleshoot-an-issue"></a>

最好的做法是创建一个免费的 Bitwarden Cloud 账户，然后尝试使用他们的在线云服务重现相同步骤。如果您可以通过 Bitwarden Cloud 环境重现此问题，请在 Bitwarden 上报告此问题，并说明您使用的是他们的在线云服务。

现在，如果由于某种原因在 Bitwarden Cloud 上无法重现此问题，它可能仍然是一个客户端问题，但与自托管系统相关，而 Bitwarden 也提供了自托管系统。不过，对于大多数用户来说，自托管系统的测试难度较大，因此不建议大多数用户尝试。

## 无法复现时，接下来该做什么？ <a href="#when-unable-to-reproduce-what-to-do-next" id="when-unable-to-reproduce-what-to-do-next"></a>

如果您无法使用 Bitwarden Cloud 重现此问题，并且 Vaultwarden 的 :testing 标签镜像也无效，那么您可能不知道该向哪里报告此问题。

为避免产生大量问题，最好先发起一个[讨论](https://github.com/dani-garcia/vaultwarden/discussions/categories/bitwarden-clients-q-a) ，一旦确定这可能是一个 Vaultwarden 的问题后，我们可以将此讨论转换为问题，或者可能要求您创建一个包含必要步骤和详细信息的新问题。这对用户和开发人员都有帮助。

## 测试客户端的技巧和窍门 <a href="#tips-and-tricks-to-test-clients" id="tips-and-tricks-to-test-clients"></a>

在这里添加一些关于如何排除客户端问题的技巧和窍门或许有所帮助。

我们已经有了一个 Android 客户端页面，其他客户端可能也将陆续跟进。

此外，最新版 Bitwarden 移动客户端有一个名为 Flight Recording（飞行记录仪）的功能，可辅助故障排查。该功能同样适用于Vaultwarden，它可能有助于显示客户端期望但没有接收到的内容。

要了解更多关于此功能的信息，请参阅 Bitwarden 文档：[https://bitwarden.com/help/flight-recorder/](https://bitwarden.com/help/flight-recorder/)

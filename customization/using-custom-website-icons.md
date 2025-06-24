# 4.使用自定义网站图标

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Using-custom-website-icons)
{% endhint %}

{% hint style="info" %}
本页面介绍的是显示在您的条目旁边的[网站图标](https://help.ppgg.in/security/data/website-icons)（当使用 `internal` 图标服务时）。如果您想自定义 web-vault 的外观，请参考[自定义 Vaultwarden CSS](customize-vaultwarden-css.md)。
{% endhint %}

{% hint style="warning" %}
客户端仅会为已配置了自动填充 URI 的条目请求图标。请注意，您也可以在客户端设置中关闭网站图标功能，此时客户端将不会向 Vaultwarden 请求图标。
{% endhint %}

如果您想为您的网站条目添加自定义图标，您可以将它们放置在 `ICON_CACHE_FOLDER` 位置（默认为 `data/icon_cache` ）。命名规则基于条目的指定 IP 或完全限定域名 (FQDN)，即 Bitwarden 在[此图示](https://help.ppgg.in/password-manager/autofill/troubleshoot-autofill/forming-uris-for-autofill#match-detection-options)中叫做 Hostname 的字段：

{% embed url="https://github.com/user-attachments/assets/47bdf0f1-46f9-41af-8030-d0f860e2a056" %}

这意味着在请求图标时将忽略方案和端口，因此您无法根据端口号提供不同的图标。

虽然 web-vault 支持多种图像类型，如 ICO、BMP、GIF、JPG、WEBP 和 PNG，但缓存的图标本身始终被命名为 `<fqdn>.png` 或 `<IP>.png` （例如 `data/icon_cache/en.wikipedia.org.png`）。因此，您应该相应地命名您的自定义图标。

## 图标缓存过期机制 <a href="#how-the-icon-cache-expiration-works" id="how-the-icon-cache-expiration-works"></a>

如果图标文件已存在，它将检查其最后修改时间是否已过期（可通过 `ICON_CACHE_TTL` 配置）。若已过期，则会尝试获取新图标而非直接使用现有图标。您可通过设置 `ICON_CACHE_TTL=0` 禁用过期功能，使 Vaultwarden 永久保留本地缓存的现有图标。

如果您无法使用 `ICON_CACHE_TTL=0` （因您希望为大多数网站获取新图标，仅提供少量自定义图标），您可以编写一个 cron 作业，对您自定义放置的图标定期调用 `touch`，从而保持其修改时间为最新状态以避免过期。

{% hint style="warning" %}
`ICON_CACHE_TTL` 默认设置为 2592000 秒（30 天），如果您未禁用过期或未定期更新修改时间，30 天后，任何手动放置的图标将被忽略并可能被覆盖。
{% endhint %}

如果获取图标失败（无论何种原因），Vaultwarden 将在 `ICON_CACHE_FOLDER` 设定的文件夹中为该域名创建一个空白的 `.miss` 文件（例如 `data/icon_cache/en.wikipedia.org.png.miss`），并在 `ICON_CACHE_NEGTTL` 设定的时间内不再尝试获取图标，而是使用备用图标。当 `.miss` 文件过期后，新的请求将自动移除该 `.miss` 文件（此处的「过期」指文件存在时间超过 `ICON_CACHE_NEGTTL` 中设定的秒数值，默认为 3 天）。

{% hint style="warning" %}
只要存在 `.miss` 文件（意味着未过期），即使存在有效的图标，Vaultwarden 仍会使用备用图标。因此对于已创建的自定义图标或已更新修改时间的图标，请务必移除对应的 `.miss` 文件。
{% endhint %}

## 网站图标故障排除 <a href="#website-icon-troubleshooting" id="website-icon-troubleshooting"></a>

如果您没有禁用图标下载（`DISABLE_ICON_DOWNLOAD`），Vaultwarden `internal` 图标服务将从指定的资源下载请求的图标。这是通过向指定的域名/IP（忽略端口）发起网络请求来完成的。如果您的 Vaultwarden 服务器无法发起出站请求（例如缺少互联网访问），则不会下载新图标。

默认情况下，出于安全考虑，Vaultwarden 会[屏蔽其认为非全局（即私有网络）的某些 IP 范围](https://github.com/dani-garcia/vaultwarden/blob/9059437c35e35ab8eb7d1d4716bf13eec0a4ee64/src/util.rs#L776-L819)。您还可以通过配置 `HTTP_REQUEST_BLOCK_REGEX` 来进一步配置 Vaultwarden 应额外屏蔽的主机。

如果您设置了 `ICON_CACHE_NEGTTL=0`，则会禁用 miss 指示器过期，这意味着 Vaultwarden 将始终为指定的域名使用默认的备用图标。

# 9.允许从内部服务获取图标

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Allow-icon-fetching-from-internal-services)
{% endhint %}

此配置适用于自托管环境，Vaultwarden 需要从托管在内部/私有网络上的服务中获取图标，例如：

* 托管多个自托管应用程序的 NAS 或服务器
* 通过本地网络访问的各种服务
* 仅通过 VPN（例如 Tailscale）才能访问的服务
* 使用内部 IP 或拆分 DNS 的反向代理设置

默认情况下，Vaultwarden 会出于安全考虑阻止对非全局/私有 IP 地址的请求。因此，解析到以下地址的服务图标可能无法加载：

* 局域网 IP 地址（`192.168.x.x`、`10.x.x.x` 等）
* Tailscale/CGNAT 范围（`100.x.x.x`）
* 其他仅供内部使用的地址

## 配置 <a href="#configuration" id="configuration"></a>

设置以下环境变量：

```systemd
HTTP_REQUEST_BLOCK_NON_GLOBAL_IPS=false
```

根据 Vaultwarden 版本，您可能还需要设置：

```systemd
ICON_BLACKLIST_NON_GLOBAL_IPS=false
```

但是，`ICON_BLACKLIST_NON_GLOBAL_IPS` 已弃用，新版本使用 `HTTP_REQUEST_BLOCK_NON_GLOBAL_IPS`。

然后重启/重新部署 Vaultwarden。

## TrueNAS SCALE 重要提示 <a href="#truenas-scale-important-note" id="truenas-scale-important-note"></a>

当将 Vaultwarden 作为 TrueNAS SCALE App 来运行时，仅设置环境变量可能还不够。

TrueNAS 可以通过应用程序配置界面来覆盖 Vaultwarden 的一些内部设置。

您还必须：

1. 打开 Vaultwarden 管理面板
2. 前往 `Advanced Settings`
3. 定位到 `Block non global IPs`
4. 将其设置为 `false` / 禁用
5. 保存然后重启 App

如果此设置保持启用状态，即使环境变量已存在，Vaultwarden 仍将继续阻止来自内部 IP 范围的图标下载。

## 安全考量 <a href="#security-considerations" id="security-considerations"></a>

禁用 `HTTP_REQUEST_BLOCK_NON_GLOBAL_IPS` 会降低对 SSRF（Server-Side Request Forgery - 服务器端请求伪造）攻击的防护能力。

禁用此设置后，Vaultwarden 可以向内部/私有 IP 地址范围发出 HTTP 请求。这对于仅通过内部网络、VPN 或私有 DNS 公开自托管服务的环境是必需的。

仅当满足以下条件时才禁用此设置：

* 您信任那些可以创建/编辑密码库条目的用户。
* 您的 Vaultwarden 实例是私有的，并且安全可靠。
* 您理解 Vaultwarden 将能够访问内部网络资源。

对于大多数自托管家庭实验室或内部基础设施设置而言，这种权衡是可以接受的，并且是实现正确图标获取功能的必要条件。

## 症状 <a href="#symptoms" id="symptoms"></a>

Vaultwarden 日志可能包含类似如下的警告：

```
IP 100.x.x.x for domain 'service.example.com' is not a global IP!
```

或者：

```
IP 192.168.x.x for domain 'service.example.com' is not a global IP!
```

禁用此限制后，内部/自托管服务的图标应该可以开始正常工作了。

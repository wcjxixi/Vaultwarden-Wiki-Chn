# 9.允许从内部服务获取图标

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Allow-icon-fetching-from-internal-services)
{% endhint %}

此配置适用于 Vaultwarden 需要从托管在内部/私有网络上的服务中获取图标的自托管环境，例如：

* 托管了多个自托管应用程序的 NAS 或服务器
* 通过本地网络访问的各种服务
* 仅通过 VPN（例如 Tailscale）才能访问的服务
* 使用内部 IP 或拆分式 DNS 的反向代理设置

默认情况下，出于安全考虑，Vaultwarden 会阻止对非全局/私有 IP 地址的请求。因此，解析到以下地址的服务图标可能无法加载：

* 局域网 IP 地址（`192.168.x.x`、`10.x.x.x` 等）
* Tailscale/CGNAT 范围（`100.x.x.x`）
* 其他仅供内部使用的地址

> **\[译者注]**：[CGNAT](https://zh.wikipedia.org/wiki/%E7%94%B5%E4%BF%A1%E7%BA%A7NAT) - 电信级 NAT 或运营商级 NAT（Carrier-grade NAT，缩写为 CGNAT 或 CGN），也称大规模 NAT（large-scale NAT，缩写 LSN），是运营商为了缓解 IPv4 地址枯竭问题，向客户分配私网 IPv4 地址而非公网地址，并通过自身的中间件完成的网络地址转换 (NAT) 操作。电信级 NAT 可以让更多的终端设备共享一个公网地址。

## 配置 <a href="#configuration" id="configuration"></a>

设置以下环境变量：

```systemd
HTTP_REQUEST_BLOCK_NON_GLOBAL_IPS=false
```

根据 Vaultwarden 版本，您可能还需要设置：

```systemd
ICON_BLACKLIST_NON_GLOBAL_IPS=false
```

但是，`ICON_BLACKLIST_NON_GLOBAL_IPS` 已弃用，新版本 Vaultwarden 使用 `HTTP_REQUEST_BLOCK_NON_GLOBAL_IPS`。

然后重启/重新部署 Vaultwarden。

## TrueNAS SCALE 重要提示 <a href="#truenas-scale-important-note" id="truenas-scale-important-note"></a>

当将 Vaultwarden 作为 TrueNAS SCALE App 来运行时，仅设置环境变量可能还不够。

> **\[译者注]**：[TrueNAS](https://www.truenas.com/) 是一个基于 ZFS 的开源 NAS（网络附加存储）系统，类似于群晖 DSM、威联通 QNAP。TrueNAS  由 [iXsystems](https://www.ixsystems.com/) 开发。
>
> TrueNAS 有两个主要版本：TrueNAS CORE（原 FreeNAS）和 TrueNAS SCALE。TrueNAS CORE 基于 FreeBSD，TrueNAS SCALE 基于 Linux。

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

> **\[译者注]**：SSRF 是一种由攻击者构造形成由服务端发起请求的安全漏洞。一般情况下，SSRF 攻击的目标是从外网无法访问的内部系统。详见[《深入理解 WEB 漏洞之 SSRF 漏洞》](https://github.com/ASTTeam/SSRF)。

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

# 2.使用 Let's Encrypt 证书运行私有 Vaultwarden 实例

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Running-a-private-vaultwarden-instance-with-Let's-Encrypt-certs)
{% endhint %}

假设您希望运行一个只能从本地网络访问的 Vaultwarden 实例，但您又希望此实例启用由一个被广泛接受的 CA 而不是你自己的[私有 CA](../../other-information/private-ca-and-self-signed-certs-that-work-with-chrome.md) 来签署的 HTTPS（以避免将专用 CA 证书加载到所有设备中的麻烦）。

本文将演示如何使用 [Caddy](https://caddyserver.com/) Web 服务器创建这样的设置，Caddy 内置了对诸多 DNS 提供商的 ACME 支持。我们将通过 ACME [DNS 验证方式](https://letsencrypt.org/docs/challenge-types/#dns-01-challenge)获取 Let's Encrypt 证书来配置 Caddy -- 在这里使用通常的 HTTP 验证方式的话会有问题，因为它依赖于 Let's Encrypt 服务器能够访问到您的内部 Web 服务器。

{% hint style="danger" %}
本文涵盖了更通用的 DNS 验证设置，但许多用户可能会发现使用 Docker Compose 来集成 Caddy 和 Vaultwarden 是最简单的。具体的例子请参见[使用 Docker Compose](../../container-image-usage/using-docker-compose.md#caddy-with-dns-challenge)。
{% endhint %}

涵盖了两个 DNS 提供商：

* [Duck DNS](https://www.duckdns.org/) -- 为你提供一个 `duckdns.org` 下的子域名（例如 `my-bwrs.duckdns.org`）。如果您没有自己的域名，此选项是最简单的。
* [Cloudflare](https://www.cloudflare.com/) -- 这可以让您把您的 Vaultwarden 实例放在您拥有或控制的域名下。请注意，Cloudflare 可以只作为一个 DNS 提供商使用（即不使用 Cloudflare 最著名的代理功能）。如果您目前没有自己的域名，您也许可以在 [Freenom](https://www.freenom.com/) 获得一个免费的域名。

当然也可以使用其他的网络服务器、[ACME 客户端](https://letsencrypt.org/docs/client-options/)和 DNS 提供商的组合来创建类似的设置，但您必须解决细节上的差异。

## 获取自定义 Caddy 构建 <a href="#getting-a-custom-caddy-build" id="getting-a-custom-caddy-build"></a>

由于大多数人不使用 DNS 验证方式，为每个 DNS 提供商自定义实现，因此 Caddy 默认没有内置此验证方式的支持。

最简单的方式是通过 [https://caddyserver.com/download](https://caddyserver.com/download) 获取带有 DNS 验证模块的 Caddy 版本。选择您的平台，选中 `github.com/caddy-dns/cloudflare`（用于 Cloudflare）和/或 `github.com/caddy-dns/duckdns`（用于 Duck DNS），然后点击下载。

如果您喜欢从源代码构建，可以使用 [`xcaddy`](https://caddyserver.com/docs/build#xcaddy)。例如，要创建一个包含 Cloudflare 和 Duck DNS 支持的构建：

```batch
xcaddy build --with github.com/caddy-dns/cloudflare --with github.com/caddy-dns/duckdns
```

将 `caddy` 二进制 移动到 `/usr/local/bin/caddy` 或其他合适的目录中。使文件可执行。（可选）运行语句 `sudo setcap cap_net_bind_service=+ep /usr/local/bin/caddy` 以允许 `caddy` 在特权端口 (< 1024) 上监听，而无须以 root 身份运行。

## Duck DNS 设置 <a href="#duck-dns-setup" id="duck-dns-setup"></a>

如果您还没有账户，请在 [https://www.duckdns.org/](https://www.duckdns.org/) 创建一个。给您的 Vaultwarden 实例创建一个子域名（例如，`my-vw.duckdns.org`），将其 IP 地址设置为您的 Vaultwarden 主机的私有 IP（例如，192.168.1.100）。记下您的账户的 token 值（[UUID](https://en.wikipedia.org/wiki/UUID) 格式的字符串）。Caddy 将需要此 token 来完成 DNS 验证。

在 caddy 可执行文件所在的同一目录中创建一个名为 `Caddyfile`（大写 C，无文件扩展名）的文件，其中包含以下内容，并将 `localhost:` 端口替换为 Vaultwarden 在其 `ROCKET_PORT=` 指令中使用的端口（Vaultwarden 的默认 Rocket\_port 为 8001）：

```nginx
{$DOMAIN}:443 {
    tls {
        dns duckdns {$DUCKDNS_TOKEN}
    }
    reverse_proxy localhost:8001
}
```

创建一个名为 `caddy.env` 的文件，内容如下（替换相应的值）：

```systemd
DOMAIN=my-vw.duckdns.org
DUCKDNS_TOKEN=00112233-4455-6677-8899-aabbccddeeff
```

切换到 caddy 所在目录然后运行以下命令以首次启动 `caddy`：

```batch
caddy run --envfile caddy.env
```

Duck DNS 域名（例如 my-vw.duckns.org）的 caddy 首次启动需要几秒钟的时间来解决 DNS 挑战和获取 HTTPS 证书。Caddy 通常将它们存储在 `/root/.local/share/caddy` 中，以及 caddy 的配置会自动保存到 `/root/.config/caddy`。

运行命令以启动 `vaultwarden`：

```batch
export ROCKET_PORT=8001

./vaultwarden
```

{% hint style="info" %}
在设置 Caddy 之前，Vaultwarden 是否已运行并不重要。
{% endhint %}

您现在应该可以通过 `https://my-vw.duckdns.org` 访问到您的 Vaultwarden 实例了。如果没有，请检查 caddy 的输出。

您可以使用 \[STRG]-\[C] 来停止 caddy。接下来通过以下命令在后台启动 caddy：

```batch
caddy start --envfile caddy.env
```

**重要提示：**如有必要，在某些路由器（例如 FritzBox）或 DNS 解析器（例如 unbound）中，由于 DNS 重新绑定保护，必须为域名（例如 `my-vw.example.com`）设置例外。

## Cloudflare 设置 <a href="#cloudflare-setup" id="cloudflare-setup"></a>

如果您还没有账户，请在 [https://www.cloudflare.com/](https://www.cloudflare.com/) 创建一个；您还需要到您的域名注册商那里将名称服务器设置为 Cloudflare 分配给您的值。为您的 Vaultwarden 实例创建一个子域名（例如，`vw.example.com`），将其 IP 地址设置为您的 Vaultwarden 主机的私有 IP（例如，`192.168.1.100`）。例如：

<figure><img src="https://camo.githubusercontent.com/17b5c9a41a4dfda12a3e60cdf054456392b5361a08082b1e9e2433d0c5354fa5/68747470733a2f2f692e696d6775722e636f6d2f4242767934596a2e706e67" alt=""><figcaption></figcaption></figure>

创建一个用于 DNS 验证的 API token（更多背景知识，请参阅 [https://github.com/libdns/cloudflare/blob/master/README.md](https://github.com/libdns/cloudflare/blob/master/README.md)）：

1. 点击右上角的个人图标并导航到 `My Profile`，然后选择 `API Tokens` 选项卡。
2. 点击 `Create Token` 按钮，然后点击`Edit zone DNS` 右边的 `Use template`。
3. 编辑 `Token name` 字段（如果您希望使用更具描述性的名称）。
4. 在 `Permissions` 下应出现一个权限：`Zone / DNS / Edit`。添加其他权限：`Zone / Zone / Read`。
5. 在 `Zone Resources` 下，设置 `Include / Specific zone / example.com`（使用您自己的域名替换 `example.com`）。
6. 在 `TTL` 下，为您的 tokan 设置一个变为非活动状态的 End Date。您也可以在以后设置。
7. 创建 token 并复制 token 值。

您的 token 列表看起来应该像这样：

<figure><img src="https://camo.githubusercontent.com/6f44c7e4797be79e533787884ecc880c5d1797c266a4550ae2d61dbbf885932a/68747470733a2f2f692e696d6775722e636f6d2f466f4f763957772e706e67" alt=""><figcaption></figcaption></figure>

创建一个名为 `Caddyfile` 的文件，内容如下：

```nginx
{$DOMAIN}:443 {
    tls {
        dns cloudflare {$CLOUDFLARE_API_TOKEN}
    }
    reverse_proxy localhost:8080
}
```

创建一个名为 `caddy.env` 的文件，内容如下（替换相应的值）：

```systemd
DOMAIN=vw.example.com
CLOUDFLARE_API_TOKEN=<your-api-token>
```

运行命令以启动 `caddy`：

```batch
caddy run --envfile caddy.env
```

运行命令以启动 `vaultwarden`：

```batch
export ROCKET_PORT=8080

./vaultwarden
```

您现在应该可以通过 `https://vw.example.com` 访问到您的实例了。

**重要提示：**如有必要，在某些路由器（例如 FritzBox）中，由于 DNS 重新绑定保护，必须为域名（例如 `vw.example.com`）设置例外。

## 使用 `lego` CLI 获取证书 <a href="#getting-certs-using-the-lego-cli" id="getting-certs-using-the-lego-cli"></a>

在上面的 DuckDNS 例子中，Caddy 使用 `lego` 库通过 DNS 验证获取证书。`lego` 也有一个 CLI，你可以直接使用它来获取证书，例如，如果你想使用 Caddy 以外的反向代理。 (注意：这个例子使用 `lego`，但也有其他独立的 ACME 客户端支持 DNS 验证方式（参阅 [DNS 验证](running-a-private-vaultwarden-instance-with-lets-encrypt-certs.md#dns-challenge)部分）。

下面是一个如何做到这一点的例子。

1. 从 [https://github.com/go-acme/lego/releases](https://github.com/go-acme/lego/releases) 下载预建的 `lego` 二进制文件到您的系统中。将其解压到某个目录，比如 `/usr/local/lego`。
2. 从那个目录中，运行 `DUCKDNS_TOKEN=<token> ./lego -a --dns duckdns -d my-vm.duckdns.org -m me@example.com run`（用合适的值替换令牌、域名和电子邮件地址）。这将使您在 Let's Encrypt 注册，并为您的域名获取一个证书。
3. 设置一个每周的 cron 作业来运行 `DUCKDNS_TOKEN=<token> ./lego --dns duckdns -d my-vw.duckdns.org -m me@example.com renew`。这将在你的证书即将到期时更新它。

{% hint style="info" %}
`lego` 默认请求 ECC/ECDSA 证书。如果您使用 Vaultwarden 中内置的 [Rocket HTTPS 服务器](enabling-https.md#via-rocket)，您需要请求 RSA 证书。在上面的 `lego` 命令中，添加选项 `--key-type rsa2048`。
{% endhint %}

在这个例子中，您需要用生成的输出来配置您的反向代理：

* `/usr/local/lego/.lego/certificates/my-vw.duckdns.org.crt` （证书）
* `/usr/local/lego/.lego/certificates/my-vw.duckdns.org.key` （私钥）

## 故障排除 <a href="#troubleshooting" id="troubleshooting"></a>

### DNS 问题 <a href="#dns-issues" id="dns-issues"></a>

如果您的子域名出现 DNS 解析错误（例如，`DNS_PROBE_FINISHED_NXDOMAIN` 或 `ERR_NAME_NOT_RESOLVED`），可能是您的 DNS 解析器阻止了解析，有以下原因：

1. 出于安全原因，它会阻止动态 DNS 服务。
2. 为防止 [DNS 重新绑定](https://en.wikipedia.org/wiki/DNS\_rebinding)攻击，或出于其他一些原因，它会阻止域名解析到私有 (RFC 1918) IP 地址。

无论哪种情况，您都可以尝试使用其他 DNS 解析器，例如 Google 的 `8.8.8.8` 或 Cloudflare 的 `1.1.1.1`。对于第二种情况，如果您在 dnsmasq 或 Unbound 等本地 DNS 服务器后面运行，则可以将其配置为完全禁用 DNS 重新绑定保护，或允许某些域名返回私有地址。关于 Unbound，您可以通过将以下指令添加到其配置文件中来实现（使用您自己的 Duck DNS 域名替换该域名）：

```yaml
private-domain: "my-vw.duckdns.org"
```

然后通过 `unbound-control reload` 或 `systemctl restart unbound` 重新启动 unbound 以使其加载新配置。

此外，请确保关闭您之前为 Vaultwarden 设置的 HTTPS 设置，特别是通过 Rocket TLS 使用您自己的（自签名）证书的私有 CA，因为这会干扰您新的 Let's Encrypt 受保护的域名。只需在 Vaultwarden 的环境文件中注释掉（# 符号）`ROCKET_TLS` 指令即可：

```systemd
# ROCKET_TLS={certs="./cert.pem",key="./privkey.pem"}
```

### Vaultwarden 登录问题 <a href="#vaultwarden-login-issues" id="vaultwarden-login-issues"></a>

更改域名后不要忘记更新 Vaultwarden 的环境文件：

```systemd
DOMAIN=https://my-vw.duckdns.org
```

## 参考 <a href="#references" id="references"></a>

### DNS 验证 <a href="#dns-challenge" id="dns-challenge"></a>

* [https://caddy.community/t/how-to-use-dns-provider-modules-in-caddy-2/8148](https://caddy.community/t/how-to-use-dns-provider-modules-in-caddy-2/8148)
* [https://community.letsencrypt.org/t/dns-providers-who-easily-integrate-with-lets-encrypt-dns-validation/86438](https://community.letsencrypt.org/t/dns-providers-who-easily-integrate-with-lets-encrypt-dns-validation/86438)

### Caddy Cloudflare 组件 <a href="#caddy-cloudflare-module" id="caddy-cloudflare-module"></a>

* [https://github.com/caddy-dns/cloudflare](https://github.com/caddy-dns/cloudflare)
* [https://go-acme.github.io/lego/dns/cloudflare/](https://go-acme.github.io/lego/dns/cloudflare/)

### Caddy Duck DNS 组件 <a href="#caddy-duck-dns-module" id="caddy-duck-dns-module"></a>

* [https://github.com/caddy-dns/duckdns](https://github.com/caddy-dns/duckdns)
* [https://go-acme.github.io/lego/dns/duckdns/](https://go-acme.github.io/lego/dns/duckdns/)

# 1.启用 HTTPS

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-HTTPS)
{% endhint %}

现在几乎都需要启用 [HTTPS](https://en.wikipedia.org/wiki/HTTPS)，才能满足 Vaultwarden 的正常操作，这是因为 Bitwarden 网络密码库使用的 [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto)，大多数浏览器只有在 HTTPS 环境下才能使用。

启用 HTTPS 的几种方式：

* （推荐）把 Vaultwarden 放在一个[反向代理](https://en.wikipedia.org/wiki/Reverse\_proxy)后面，代表 Vaultwarden 处理 HTTPS 连接。
* （不推荐）启用 Vaultwarden 内置的 HTTPS 功能（通过 [Rocket](https://rocket.rs) 网络框架）。Rocket 的 HTTPS 实现相对不成熟且有限。此方式也不支持 [WebSocket 通知](../../configuration/enabling-websocket-notifications.md)。

有关这些选项的更多细节，请参考[启用 HTTPS](enabling-https.md#enabling-https) 部分。

要使 HTTPS 服务器工作，它还需要 SSL/TLS 证书，因此您需要决定如何获得该证书。同样，有几个选项：

* （推荐）使用 [ACME 客户端](https://letsencrypt.org/docs/client-options/)获取 [Let's Encrypt](https://letsencrypt.org) 证书。一些反向代理（例如 [Caddy](https://caddyserver.com)）也内置支持使用 ACME 协议获取证书。
* （推荐）如果你信任 [Cloudflare](https://www.cloudflare.com) 来代理你的流量，你可以让他们处理你的 SSL/TLS 证书的发放。请注意，上游的 Bitwarden 网络密码库（[https://vault.bitwarden.com/](https://vault.bitwarden.com)）运行在 Cloudflare 后面。
* （不推荐）[建立一个私人 CA](../../other-information/private-ca-and-self-signed-certs-that-work-with-chrome.md)，并发行你自己的（自签名）证书。这有各种陷阱和不便，所以请自行考虑是否使用此选项。

参考[获取 SSL/TLS 证书](enabling-https.md#getting-ssl-tls-certificates)部分，以了解这些选项的更多细节。要使移动应用程序正常运行，必须设置正确的 [OCSP 装订](https://en.wikipedia.org/wiki/OCSP\_stapling)设置。

## 启用 HTTPS <a href="#enabling-https" id="enabling-https"></a>

### 通过反向代理 <a href="#via-a-reverse-proxy" id="via-a-reverse-proxy"></a>

有很多常用的反向代理， 在[代理示例](../proxy-examples.md)中可以找到一些配置的示例。 如果您不熟悉反向代理并且没有偏好，请首先考虑 [Caddy](https://caddyserver.com)，因为它内置了对获取 Let's Encrypt 证书的支持。[使用 Docker Compose](../../container-image-usage/using-docker-compose.md) 文章中有一个使用 Caddy 的很好的例子。

### 通过 Rocket <a href="#via-rocket" id="via-rocket"></a>

{% hint style="warning" %}
不建议使用此方法。
{% endhint %}

要对 `vaultwarden` 本身启用 HTTPS，请设置如下格式的 `ROCKET_TLS` 环境变量：

```python
ROCKET_TLS={certs="/path/to/certs.pem",key="/path/to/key.pem"}
```

位置：

* `certs`：PEM 格式的 SSL/TLS 证书链的路径
* `key`：PEM 格式的 SSL/TLS 证书对应的私钥文件的路径

说明：

* `ROCKET_TLS` 行中使用的文件\_扩展名\_不一定非要像示例中那样是 `.PEM`。某些地方可能会使用其他扩展名，例如，`.crt` 作为证书，`.key` 作为私钥。这些文件的\_格式\_必须是 PEM，即 base64 编码。而 PEM 只是 openssl 的默认格式，因此您可以将 .cert、.cer、.crt 和 .key 文件重命名为 .pem，反之亦然，使用 .crt 或 .key 作为 `ROCKET_TLS` 行中的文件扩展名。
*   使用 RSA 证书/密钥。Rocket 似乎无法处理 ECC 证书/密钥，否则会输出类似下面的误导性错误消息：

    > `[ERROR] environment variable ROCKET_TLS={certs="/ssl/ecdsa.crt",key="/ssl/ecdsa.key"} could not be parsed`

    （环境变量本身的格式没有错误；只是因为 Rocket 无法解析证书/密钥内容。）
* 如果在 Docker 下运行，请记住，Vaultwarden 在容器内部运行时将解析 `ROCKET_TLS` 值 ，所以请确保 `certs` 和 `key` 路径是容器内部呈现的样子（可能与 Docker 主机系统上的路径不同）。

```python
docker run -d --name vaultwarden \
  -e ROCKET_TLS='{certs="/ssl/certs.pem",key="/ssl/key.pem"}' \
  -v /ssl/keys/:/ssl/ \
  -v /vw-data/:/data/ \
  -p 443:80 \
  vaultwarden/server:latest
```

您需要挂载 ssl 文件夹（使用 -v 参数），同时需要转发适当的端口（使用 -p 参数），通常是用于 HTTPS 连接的 443 端口。如果您选择的端口号不是 443，例如 3456，请记住在连接到服务时明确提供该端口号，例如：`https://vaultwarden.local:3456`。

有关如何在本地系统上设置和使用私有 CA 的更多信息，请参阅[此页面](../../other-information/private-ca-and-self-signed-certs-that-work-with-chrome.md)。如果遵循该指南，您的 ROCKET\_TLS 行看起来应该像这样：`-e ROCKET_TLS='{certs="/ssl/vaultwarden.crt",key="/ssl/vaultwarden.key"}' \`

{% hint style="warning" %}
确保你的证书文件包含完整的信任链。对于 certbot，这意味着应使用 `fullchain.pem` 而不是 `cert.pem`。完整的信任链应该包括两个证书：叶证书（与 `cert.pem` 中的内容相同），后面跟随 R3 或 E1 [中间证书](https://letsencrypt.org/certificates/#intermediate-certificates)。例如，Android 默认不在其系统信任存储中包含任何 Let's Encrypt 中间证书，所以如果你不提供完整的证书链，Android 客户端很可能无法连接。
{% endhint %}

用于获取证书的软件通常使用符号链接。如果是这样的话，需要确保这两个位置能被 docker 容器访问到。

例如：[certbot](https://certbot.eff.org) 会在 `/etc/letsencrypt/live/mydomain/` 下创建一个包含所需要的 `fullchain.pem` 和 `privkey.pem` 文件的文件夹。

这些文件链接到 `../../archive/mydomain/privkey.pem`。

因此，从 Vaultwarden 容器中使用，应像这样：

```python
docker run -d --name vaultwarden \
  -e ROCKET_TLS='{certs="/ssl/live/mydomain/fullchain.pem",key="/ssl/live/mydomain/privkey.pem"}' \
  -v /etc/letsencrypt/:/ssl/ \
  -v /vw-data/:/data/ \
  -p 443:80 \
  vaultwarden/server:latest
```

#### 检查证书是否有效 <a href="#check-if-certificate-is-valid" id="check-if-certificate-is-valid"></a>

当您的 Vaultwarden 服务器对外界可用时，您可以使用 [Comodo SSL Checker](https://comodosslstore.com/ssltools/ssl-checker.php)，[Qualys' SSL Labs](https://www.ssllabs.com/ssltest/) 或 [Digicert SSL Certficate Checker](https://www.digicert.com/help/) 来检查您的 SSL 证书（包括证书链）是否有效。缺少证书链，Android 设备将连接失败。

您可以使用 [Qualys' SSL Labs](https://www.ssllabs.com/ssltest/analyze.html) 检查，但它不支持自定义端口。另外，请记住选中「Do not show the results on the boards」复选框，否则您的系统将在「Recently Seen」列表中可见。

如果您运行的是没有与公共 Internet 连接的本地服务器，则可以使用 `openssl` 命令，[testssl.sh](https://testssl.sh) 或 [SSLScan](https://github.com/rbsec/sslscan/) 来验证证书的有效性。

执行以下操作以验证证书是否随链安装（注意将 vault.domain.com 改为您自己的域名）：

```python
openssl s_client -showcerts -connect vault.domain.com:443 -servername vault.domain.com

# 或者不同的端口，比如 7070
openssl s_client -showcerts -connect vault.domain.com:7070 -servername vault.domain.com
```

输出的开头应类似于以下内容（使用 Let's Encrypt 证书）：

```python
CONNECTED(00000003)
depth=2 O = Digital Signature Trust Co., CN = DST Root CA X3
verify return:1
depth=1 C = US, O = Let's Encrypt, CN = Let's Encrypt Authority X3
verify return:1
depth=0 CN = vault.domain.com
verify return:1
```

有 3 个不同深度（请注意，它是从 0 开始的）级别的验证。在接下来的输出中，您应该看到来自 Let's Encryptbase 的使用 base64 编码的证书信息。

#### 检查 OSCP 有效性 <a href="#check-oscp-validity" id="check-oscp-validity"></a>

> \[**译者注**]：OCSP是在线证书状态协议（Online Certificate Status Protocol）的缩写 ，是一个用于获取 X.509 数字证书撤销状态的网际协议，用于检验证书合法性。OCSP 查询需要建立一次完整的 HTTP 查询请求，期间的 DNS 查询、建立 TCP 连接、服务端响应和数据传输都是额外开销，使得建立 TLS 连接花费更多时长。后来出现了OCSP Stapling ，将原本需要客户端发起的 OCSP 请求转嫁给服务端，并随证书一起发送给客户端，因此能提高 TLS 握手效率。
>
> OCSP Stapling 一般翻译为 OCSP 装订或 OCSP 封套。

如果 OCSP Stapling 无法正常工作，则连接移动应用程序将失败，并显 `Chain validation failed` 消息。

正确设置 OCSP Stapling 后，[Digicert SSL Checker](https://www.digicert.com/help/) 的吊销检查部分将包含「OCSP Staple: Good」。您的网络服务器必须能够连接到证书的 X509v3 扩展中的「Authority Information Access」URL，才能使 OCSP Stapling 正常工作。

您还可以使用如下命令行检查 OCSP 的状态：

```
openssl s_client -showcerts -connect vault.domain.com:443 -servername vault.domain.com -status
```

在其输出中必须包含：

```
OCSP Response Status: successful (0x0)
```

## 获取 SSL/TLS 证书 <a href="#getting-ssl-tls-certificates" id="getting-ssl-tls-certificates"></a>

### 通过 Let's Encrypt <a href="#via-lets-encrypt" id="via-lets-encrypt"></a>

[Let's Encrypt](https://letsencrypt.org) 免费发放 SSL/TLS 证书。

为了使之工作，你的 Vaultwarden 实例必须拥有一个 DNS 名称（即你不能简单地使用 IP 地址）。如果你的 vaultwarden 可以在公共互联网上访问，那么设置 Let's Encrypt 就比较容易，但即使你的实例是私有的（即只能在你的局域网上访问），也可以通过 [DNS 挑战](running-a-private-vaultwarden-instance-with-lets-encrypt-certs.md)获取 Let's Encrypt 证书。

如果你已经拥有或控制了一个域名，那么只需为你的 Vaultwarden 实例的 IP 地址添加一个 DNS 名称即可。如果你没有，你可以购买一个域名，尝试在 [Freenom](https://www.freenom.com) 免费获得一个，或者使用像 [Duck DNS](https://www.duckdns.org) 这样的服务来获取一个现有域名下的名称（例如，`my-bitwarden.duckdns.org`）。

拥有了实例的 DNS 名称后，您就可以使用 [ACME 客户端](https://letsencrypt.org/docs/client-options/)为你的 DNS 名称获取证书。[Certbot](https://certbot.eff.org) 和 [acme.sh](https://github.com/acmesh-official/acme.sh) 是两个最流行的独立客户端。一些反向代理（例如 [Caddy](https://caddyserver.com)）也内置了 ACME 客户端。

### 通过 Cloudflare <a href="#via-cloudflare" id="via-cloudflare"></a>

[Cloudflare](https://www.cloudflare.com) 为个人提供免费服务。如果你信任他们代理你的流量，并作为你的 DNS 供应商，你也可以让他们处理你的 SSL/TLS 证书的发放。

注册您的域名并为您的 Vaultwarden 实例添加了 DNS 记录后，登录 Cloudflare 仪表板并选择 `SSL/TLS`，然后选择 `Origin Server`。生成一个原始证书（你可以选择最长 15 年的有效期），并配置 Vaultwarden 来使用它。如果你选择了 15 年有效期，那么在可预见的未来，无需续签此原始证书。

请注意，原始证书仅用于确保 Cloudflare 和 Vaultwarden 之间的通信。Cloudflare 将自动处理用于客户端和 Cloudflare 之间通信的证书的发放和更新。

另外，如果你使用的是 Vaultwarden 内置的 Rocket HTTPS 服务器，请确保选择 `RSA` 作为原始证书的私钥类型，因为 Rocket 目前不支持 ECC/ECDSA 证书。

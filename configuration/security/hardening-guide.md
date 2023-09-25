# 1.强化指南

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Hardening-Guide)
{% endhint %}

## 应用程序配置 <a href="#application-configuration" id="application-configuration"></a>

下面的小节涵盖了 Vaultwarden 本身相关的强化。

### 禁用注册和（可选）邀请 <a href="#disable-registration-and-optionally-invitations" id="disable-registration-and-optionally-invitations"></a>

默认情况下，Vaultwarden 允许任何匿名用户在未被邀请的情况下在服务器上注册新账户。这是在服务器上创建第一个用户所必需的，但建议您在管理面板中（如果启用了管理面板的话）或[使用环境变量](../disable-registration-of-new-users.md)将其禁用，以防止攻击者在 Vaultwarden 服务器上创建账户。

Vaultwarden 还允许注册用户邀请其他新用户在服务器上创建账户并加入其组织。只要您信任您的用户，这不会带来直接风险，但是可以在管理面板或[使用环境变量](../disable-registration-of-new-users.md)将其禁用。

### 禁用显示密码提示 <a href="#disable-password-hint-display" id="disable-password-hint-display"></a>

Vaultwarden 在登录页面上显示密码提示，以适应没有配置 SMTP 的小型/本地部署，这可能被攻击者滥用，以方便对服务器上的用户进行密码猜测攻击。可以在管理面板中通过取消勾选 `Show password hints` 选项或[使用环境变量](../disable-registration-of-new-users.md)来禁用它。

## HTTPS / TLS 配置 <a href="#https-tls-configuration" id="https-tls-configuration"></a>

下面的小节涵盖了 HTTPS/TLS 相关的强化。

### 严格 SNI <a href="#strict-sni" id="strict-sni"></a>

[SNI](https://zh.wikipedia.org/wiki/%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%8D%E7%A7%B0%E6%8C%87%E7%A4%BA) 是网络浏览器请求 HTTPS 服务器为特定网站（如 `vaultwarden.example.com`）提供 SSL/TLS 证书的方式。假设`vaultwarden.example.com` 的 IP 地址是 `1.2.3.4`。理想情况下，您希望您的实例只能通过 https://vaultwarden.example.com 访问，并且不能通过 https://1.2.3.4 进行访问。这是因为 IP 地址会因为各种原因被不断扫描，如果能通过这种方式检测到您的 Vaultwarden 实例，就会成为一个更明显的目标。例如，一个简单的 [Shodan 搜索](https://www.shodan.io/search?query=bitwarden)就会发现一些通过 IP 地址访问的 Bitwarden 实例。

### 反向代理 <a href="#reverse-proxying" id="reverse-proxying"></a>

一般来说，您应该避免通过 Vaultwarden 内置的 [Rocket TLS 支持](../../deployment/https/enabling-https.md)启用 HTTPS，特别是当您的实例是公开访问的时候。Rocket 本身列出了如下[警告](https://rocket.rs/v0.4/guide/configuration/#configuring-tls)：

> Rocket's built-in TLS is not considered ready for production use. It is intended for development use only.（Rocket 内置的 TLS 还不能用于生产。它只用于开发用途。）

比如，Rocket TLS 不支持严格 SNI 或 ECC 证书（仅 RSA）。

请参看[代理示例](../../deployment/proxy-examples.md)，以了解反向代理配置的示例。

## Docker 配置 <a href="#docker-configuration" id="docker-configuration"></a>

下面的小节涵盖了 Docker 相关的强化。

### 以非 root 用户运行 <a href="#run-as-a-non-root-user" id="run-as-a-non-root-user"></a>

Vaultwarden Docker 镜像被配置为默认以 root 用户的身份运行容器进程。这允许 Vaultwarden 读取/写入 [bind-mounted](https://docs.docker.com/storage/bind-mounts/) 到容器中的任何数据，而无需权限问题，即使这些数据是由另一个用户（例如，你在 Docker 主机上的用户账户）拥有的。

默认配置在安全性和可用性之间取得了很好的平衡--在一个非特权 Docker 容器中以 root 身份运行，本身就提供了合理的隔离级别，同时也让那些不是非常精通如何在 Linux 上管理所有权/权限的用户更容易进行设置。然而，作为通用策略，从安全的角度来说，以所需的最低权限运行进程是更好的；对于用 Rust 等内存安全语言编写的程序来说，这一点就不那么重要了，但请注意，Vaultwarden 也使用了一些用 C 语言编写的库代码（例如 SQLite、OpenSSL、MySQL、PostgreSQL 等）。

要在 Docker 中以非 root 用户 (uid/gid 1000) 的身份运行容器进程 (vaultwarden)：

```shell
docker run -u 1000:1000 [...other args...] vaultwarden/server:latest
```

在 `docker-compose` 中类似操作：

```batch
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: bitwarden
    user: 1000:1000
    ... other configuration ...c
```

在许多 Linux 发行版中，默认用户的 uid/gid 为 1000（运行 `id` 命令进行验证），所以如果您想在不换成其他用户的情况下轻松地访问您的 Vaultwarden 数据，这是一个很好的值，但你可以根据需要调整 uid/gid。请注意，您很可能需要指定一个数字 uid/gid，因为 Vaultwarden 容器不共享用户/组名到 uid/gid 的相同映射（例如，将容器中的 `/etc/passwd` 和 `/etc/group` 文件与 Docker 主机上的文件对比）。

Vaultwarden Docker 镜像的设置使得 `vaultwarden` 可执行文件绑定到端口 80，这工作正常，因为它默认以 root 身份运行。但是，非 root 进程通常无法绑定到[特权端口](https://www.w3.org/Daemon/User/Installation/PrivilegedPorts.html)（即低于 1024 的端口）。从版本 20.10.0 开始（参见 [moby/moby#41030](https://github.com/moby/moby/pull/41030)），Docker 专门配置其容器，以便默认情况下允许非 root 进程绑定到特权端口。对于早期版本的 Docker 或其他没有这种特殊行为的容器运行时，Vaultwarden Docker 镜像还在 `vaultwarden` 可执行文件上设置 [`cap_net_bind_service`](https://man7.org/linux/man-pages/man7/capabilities.7.html) 功能，这是另一种允许可执行文件在以非 root 用户身份运行时绑定到特权端口的方法。

### 挂载数据到容器中 <a href="#mounting-data-into-the-container" id="mounting-data-into-the-container"></a>

一般来说，只有 Vaultwarden 正常运行所需要的数据才应该被挂载到 Vaultwarden 容器中（通常情况下，这只是您的数据目录，也许还有一个包含 SSL/TLS 证书和私钥的目录）。不要挂载您的整个主目录，例如，`/var/run/docker.sock` 等，除非您有特定的原因，并且知道您在做什么。

另外，如果您不希望 Vaultwarden 修改您挂载的数据（例如，certs），可以通过在卷规范中添加 `:ro` 来[只读挂载它](https://docs.docker.com/storage/bind-mounts/#use-a-read-only-bind-mount)（例如，`docker run -v /home/username/vaultwarden-ssl:/ssl:ro`）。

## 杂项 <a href="#miscellaneous" id="miscellaneous"></a>

### 暴力破解 <a href="#brute-force-mitigation" id="brute-force-mitigation"></a>

当不使用双重身份验证时，（理论上）有可能对用户的密码进行暴力破解，从而获得对其账户的访问权限。缓解此问题的一种相对简单的方法是设置 fail2ban，设置后，在过多的失败登录尝试后将阻止访问者的 IP 地址。但是在许多反向代理（例如 cloudflare）后面使用此功能时，应格外注意。参阅：[Fail2Ban 设置](fail2ban-setup.md)。

### 隐藏在子目录下 <a href="#hiding-under-a-subdir" id="hiding-under-a-subdir"></a>

通常，Bitwarden 实例驻留在子域的根目录下（即 `bitwarden.example.com`，而不是 `bitwarden.example.com/some/path`）。上游的 Bitwarden 服务器目前只支持子域根目录，而 Vaultwarden 则增加了对[备用基本目录](../using-an-alternate-base-dir-subdir-subpath.md)的支持。对于某些用户来说，这很有用，因为他们只能访问一个子域，并希望在不同的目录下运行多个服务。在这种情况下，他们通常可以做一些显而易见的选择，比如使用 `mysubdomain.example.com/bitwarden`。然而，您也可以通过把 Vaultwarden 放在类似 `mysubdomain.example.com/vaultwarden/<mysecretstring>` 这样的目录下来提供额外的保护，其中 `<mysecretstring>` 有效地充当一个密码。也许有人会说这是[通过隐藏实现安全](https://en.wikipedia.org/wiki/Security\_through\_obscurity)，但实际上这是深度防御 -- 子目录的隐蔽性只是额外的一层安全保护，而不是为了成为主要的安全手段（用户主密码的强度仍然是主要的安全手段）。

有关安全性子路径托管的一般性讨论，请参阅：[https://github.com/debops/debops/issues/1233](https://github.com/debops/debops/issues/1233)

如果你想让 Caddy 断开除 vaultwarden 之外的所有连接：

```nginx
mysubdomain.example.com {
	route {
		reverse_proxy /my-custom-path/* 10.0.0.150:8083 {
			header_up X-Real-IP {remote_host}
		}
		handle /* {
			abort
		}
	}
}
```

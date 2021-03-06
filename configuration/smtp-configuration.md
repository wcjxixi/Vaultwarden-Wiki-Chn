# 12.SMTP 配置

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/SMTP-Configuration)
{% endhint %}

{% hint style="warning" %}
**注意**：v1.25.0 版本之前的 Vaultwarden 有一个关于 SSL 和 TLS 的漏洞/误导性的配置设置项。这已在测试版和新发布的版本中得到修复。

旧设置项是 `SMTP_SSL` 和 `SMTP_EXPLICIT_TLS`。

新设置项是 `SMTP_SECURITY`，它具有以下可用选项：`starttls`、`force_tls` 以及 `off`。

* `SMTP_SECURITY=starttls` 等同于 `SMTP_SSL=true`
* `SMTP_SECURITY=force_tls` 等同于 `SMTP_EXPLICIT_TLS=true`
{% endhint %}

您可以配置 Vaultwarden 通过 SMTP 代理来发送电子邮件：

```shell
docker run -d --name vaultwarden \
  -e SMTP_HOST=<smtp.domain.tld> \
  -e SMTP_FROM=<vaultwarden@domain.tld> \
  -e SMTP_PORT=587 \
  -e SMTP_SECURITY=starttls \
  -e SMTP_USERNAME=<username> \
  -e SMTP_PASSWORD=<password> \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

从 v1.25.0 开始，用于 SMTP SSL/TLS 配置的环境变量已更新为 `SMTP_SECURITY`（之前的有误导性，参阅[错误 #851](https://github.com/dani-garcia/vaultwarden/issues/851)）。当 `SMTP_SECURIT` 设置为 `starttls` 时（这是默认值），将仅接受 TLSv1.1 和 TLSv1.2 协议，并且 `SMTP_PORT` 默认为`587`。如果设置为 `off`，`SMTP_PORT` 则默认设置为 `25` 并将尝试加密（2020 年 3 月 12 日之前的代码不会尝试加密）。这是非常不安全的，仅在您知道您在做什么时才使用此设置。要以隐式模式（强制 TLS）运行 SMTP，请将 `SMTP_SECURITY` 设置为 `force_tls`。如果您不登录也可以发送电子邮件，简单地将 `SMTP_USERNAME` 和 `SMTP_PASSWORD` 设置为空即可。

请注意，如果启用了 SMTP 和邀请，邀请将通过电子邮件发送给新用户。您必须使用 Vaultwarden 实例的基础 URL 来设置 `DOMAIN` 配置项，以生成正确的邀请链接：

```shell
docker run -d --name vaultwarden \
...
-e DOMAIN=https://vault.example.com \
...
```

用户邀请链接有效期为 5 天，过期后需要重新发送邀请。

## SMTP 服务器 <a href="#smtp-servers" id="smtp-servers"></a>

正确配置 SMTP 服务器/中继并不是一件容易的事。Vaultwarden 使用的邮件程序库也不是最容易排除故障的。所以，除非您对自己设置这个特别感兴趣，否则使用外部服务可能更简单。

这里有几个对于大部分使用场景来说已经足够的免费服务：

* [SendGrid](https://sendgrid.com)（每天 100 封电子邮件）
* [MailJet](https://www.mailjet.com)（每天 200 封电子邮件）
* [SMTP2GO](https://www.smtp2go.com/)（每月 1000 封电子邮件）

## 一些知名服务的默认设置 <a href="#here-some-sane-defaults-for-well-known-services" id="here-some-sane-defaults-for-well-known-services"></a>

### 通用 <a href="#general" id="general"></a>

邮件服务器侦听端口 25 主要只是为了接受来自其他邮件服务器的邮件，并且仅用于它们是最终位置的邮件。此外，许多互联网提供商会阻止传出端口 25 以防止垃圾邮件。大多数需要登录的邮件服务器使用端口 587 或端口 465。端口 587 称为提交端口，大多数时候只能在使用用户名和密码时使用。在客户端和服务器之间的通信期间，端口 587 开始时未加密然后升级为 TLS 加密连接。端口 465 从一开始就是 SSL 加密的，该端口根本没有纯文本通信。

针对每一种端口的常见设置：

* 对于使用端口 465 的邮件服务器

```systemd
SMTP_PORT=465
SMTP_SECURITY=force_tls
```

* 对于使用端口 587（有时候是 25）的邮件服务器

```systemd
SMTP_PORT=587
SMTP_SECURITY=starttls
```

* 对于根本不支持加密的邮件服务器

```systemd
SMTP_PORT=25
SMTP_SECURITY=off
```

### HELO 主机名 <a href="#helo-hostname" id="helo-hostname"></a>

默认情况下，机器的主机名被用来作为 HELO 命令中的主机名。要覆盖它，您可以在配置中设置 `HELO_NAME`。

### Google/Gmail

您需要为 Vaultwarden 生成应用专用密码才能使用 Gmail。按照此处的步骤操作：[使用应用专用密码登录](https://support.google.com/accounts/answer/185833?hl=zh-Hans\&ref\_topic=7189145)。最后您会得到一个密码（为了方便输入，中间没有空格），使用这个密码。

严格 SSL：

```systemd
 # Domains: gmail.com, googlemail.com
  SMTP_HOST=smtp.gmail.com
  SMTP_PORT=465
  SMTP_SECURITY=force_tls
  SMTP_USERNAME=<mail-address>
  SMTP_PASSWORD=<less-secure-app-password>
```

StartTLS：

```systemd
  # Domains: gmail.com, googlemail.com
  SMTP_HOST=smtp.gmail.com
  SMTP_PORT=587
  SMTP_SECURITY=starttls
  SMTP_USERNAME=<mail-address>
  SMTP_PASSWORD=<less-secure-app-password>
```

另外参考：[Using Lettre With Gmail](https://web.archive.org/web/20210925161633/https://webewizard.com/2019/09/17/Using-Lettre-With-Gmail/)

### Hotmail/Outlook/Office365

```systemd
  # Domains: hotmail.com, outlook.com, office365.com
  SMTP_HOST=smtp-mail.outlook.com
  SMTP_PORT=587
  SMTP_SECURITY=starttls
  SMTP_USERNAME=<mail-address>
  SMTP_PASSWORD=<password>
  SMTP_AUTH_MECHANISM="Login"
```

### SendGrid

将 `<full-api-key>` 替换为从 SendGrid 生成的以 `SG` 开头的 API-Key。还要确保 API-Key 具有完整的 `Mail Send` 权限，否则您无法使用此密钥登录。

StartTLS：

```systemd
  SMTP_HOST=smtp.sendgrid.net
  SMTP_PORT=587
  SMTP_SECURITY=starttls
  SMTP_USERNAME=apikey
  SMTP_PASSWORD=<full-api-key>
  SMTP_AUTH_MECHANISM="Login"
```

严格 SSL：

```systemd
  SMTP_HOST=smtp.sendgrid.net
  SMTP_PORT=465
  SMTP_SECURITY=force_tls
  SMTP_USERNAME=apikey
  SMTP_PASSWORD=<full-api-key>
  SMTP_AUTH_MECHANISM="Login"
```

## 带特殊字符的密码 <a href="#passwords-with-special-characters" id="passwords-with-special-characters"></a>

如果您想在密码中使用一些特殊字符，可能需要对其中的一些字符进行转义，以免混淆环境变量解析器。

例如，可以使用 `\` 或 `'` 或 `"`，但在实际使用的时候，需要对它们进行转义。如果您使用特殊字符，最好总是使用单引号将密码括起来。

我们以下面这个密码为例：`~^",a.%\,'}b&@|/c!1(#}`

它包含了一些可能会破坏环境变量解析的字符，比如 `\`、`'` 和 `"`。单个 `\` 通常用于转义其他字符，因此如果您想使用单个 `\`，则需要键入 `\\`。 此外，引号 `'` 和 `"` 可能会引起一些问题，因此让我们将此密码括在单引号中并转义特殊字符。为了让上面的密码起作用，我们需要输入 `'~^",a.%\\,\'}b&@|/c!1(#}'`，在这里你看到我们转义了 `\` 和 `'` 字符并使用单引号将整个密码括了起来。所以：`~^",a.%\,'}b&@|/c!1(#}` 变成了 `'~^",a.%\\,\'}b&@|/c!1(#}'`。

## 使用已弃用的 `SMTP_SSL` 和 `SMTP_EXPLICIT_TLS` SMTP 环境变量（适用于 v1.24.0 及更低版本） <a href="#using-deprecated-smtp-environment-variable-smtp_ssl-and-smtp_explicit_tls-for-v1.24.0-and-lower" id="using-deprecated-smtp-environment-variable-smtp_ssl-and-smtp_explicit_tls-for-v1.24.0-and-lower"></a>

您可以配置 Vaultwarden 通过 SMTP 代理来发送电子邮件：

```shell
docker run -d --name vaultwarden \
  -e SMTP_HOST=<smtp.domain.tld> \
  -e SMTP_FROM=<vaultwarden@domain.tld> \
  -e SMTP_PORT=587 \
  -e SMTP_SSL=true \
  -e SMTP_USERNAME=<username> \
  -e SMTP_PASSWORD=<password> \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

当 `SMTP_SSL` 设置为 `true` 时（这是默认值），将仅接受 TLSv1.1 和 TLSv1.2 协议，并且 `SMTP_PORT` 默认为`587`（等同于 `SMTP_SECURITY=starttls`）。如果设置为 `false`，`SMTP_PORT` 则默认设置为 `25` 并将尝试加密（2020 年 3 月 12 日之前的代码不会尝试加密）（等同于 `SMTP_SECURITY=off`）。这是非常不安全的，仅在您知道您在做什么时才使用此设置。要以隐式模式（强制 TLS）运行 SMTP，请将 `SMTP_EXPLICIT_TLS` 设置为 `true`（等同于 `SMTP_SECURITY=force_tls`）。如果您不登录也可以发送电子邮件，简单地将 `SMTP_USERNAME` 和 `SMTP_PASSWORD` 设置为空即可。

**注意**：如果您在 v1.25.0 及更高版本上使用 `SMTP_SSL` 和 `SMTP_EXPLICIT_TLS` 设置，Vaultwarden 会对已弃用的设置产生的错误进行忽略。

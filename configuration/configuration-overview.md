# =1.配置概述

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Configuration-overview)
{% endhint %}

> \[**译者注**]：
>
> 1. 某些设置只能通过环境变量配置，如管理页面中 `Read-Only Config` 部分的配置（如日志记录）
> 2. 通过管理页面修改配置后马上生效
> 3. 若直接修改 `config.json` 文件，则需要重启 Vaultwarden 才能生效（因为只有启动时才会读取 `config.json` 文件）

## 如何配置 Vaultwarden <a href="#how-to-configure-vaultwarden" id="how-to-configure-vaultwarden"></a>

基本上有三种不同的方式配置 Vaultwarden：

1. 设置环境变量，
2. 使用 `ENV_FILE` 以及
3. 通过 `config.json`（这可以通过[管理页面](enabling-admin-page.md)生成和管理）

您可以在 [`.env.template`](https://github.com/dani-garcia/vaultwarden/blob/main/.env.template) 文件中找到大多数配置选项的文档列表。通常，注释内容中的值表示默认值，但这并不一定。如果不是，事实来源将是 [`src/config.rs`](https://github.com/dani-garcia/vaultwarden/blob/main/src/config.rs)。

如果启用了[管理页面](enabling-admin-page.md)，您还可以看到带有配置值的配置选项（如果您使用 `config.json`，则指示该值是否已从初始值更改）。

{% hint style="warning" %}
**注意：**`config.json` 文件_**不是**_配置您的设置的推荐方式！要么使用环境变量，您可以通过多种方式为您的容器环境（Docker、Docker-Compose、K8s 等）进行配置；或者，如果使用独立的二进制文件（其不是由 Vaultwarden 本身分发的），请使用位于当前工作目录下的 `.env` 文件。在管理界面中保存设置时会创建并覆盖 `config.json` 文件！
{% endhint %}

如果您依赖[第三方软件包](../deployment/third-party-packages.md)，则必须检查提供的文档（例如 Arch Linux 的 `vaultwarden` 软件包的安装通知），因为下游维护者通常会对他们的软件包做出一些假设。

### 使用环境变量 <a href="#using-environment-variables" id="using-environment-variables"></a>

配置 Vaultwarden 的推荐方法是通过环境变量。根据您运行 Vaultwarden 的方式（例如直接运行、在容器化环境中运行、通过 systemd 运行等），设置环境变量的方法有多种，因此请熟悉您的平台和安装方法。

大多数可以设置的环境变量都可以在 `.env.template` 文件中找到。您还可以使用该文件作为容器环境的环境文件的基础（例如，通过 [`env_file`](https://docs.docker.com/compose/environment-variables/set-environment-variables/#use-the-env\_file-attribute) 属性）或与 systemd 服务一起使用（参见 [`EnvironmentFile=`](https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html#EnvironmentFile=)） - 只是不要将此文件与下面的 [`ENV_FILE` 方式](configuration-overview.md#using-an-env\_file)混淆！

{% hint style="info" %}
请注意，不同平台之间的环境文件解释方式可能存在一些细微的差异（关于变量扩展或是否可以或应该在值周围使用引号等）。
{% endhint %}

您还需要确保在正确的环境中设置变量。如果您使用容器化环境，`vaultwarden` 进程将与主机平台隔离运行。如果您使用可以设置环境变量的容器管理平台（例如使用 `docker-compose` 时），这一点尤其重要。因为通常这些环境变量可以在创建容器时使用，但它们不会被传递到正在运行的容器中。

{% hint style="danger" %}
如果更改值，则需要重新创建使用环境变量配置的容器，因为这些值绑定到容器。因此，除非[从（已更改的）文件中读取](https://github.com/dani-garcia/vaultwarden/wiki/Configuration-overview#loading-individual-values-from-files)该值，否则重新启动不会执行任何操作。
{% endhint %}

### 使用 `ENV_FILE` <a href="#using-an-env_file" id="using-an-env_file"></a>

Vaultwarden 还可以直接从环境文件本身读取配置选项，这在开发 Vaultwarden 时特别有用。

默认情况下，Vaultwarden 将尝试从当前工作目录读取一个名为 `.env` 的文件（例如，如果您从签出存储库的根目录运行 `cargo run`，它应该位于同一根目录中）。

{% hint style="info" %}
使用环境文件设置进程的运行时环境（无论是使用 docker 还是 systemd）与在 Vaultwarden 中使用 env 文件之间存在差异。例如。您还可以在使用[容器镜像](../container-image-usage/which-container-image-to-use.md)时将环境文件挂载到 `/.env` ，而不是[通过 `--env-file`](https://docs.docker.com/compose/environment-variables/set-environment-variables/#substitute-with---env-file) （创建容器时读取）传递环境文件。参见上面的解释。
{% endhint %}

直接在环境中设置的值将优先于此方法。这意味着可以在不更改 `ENV_FILE` 中的值的情况下覆盖这些值（这对于调试目的可能很有用，例如，当您临时设置 `LOG_LEVEL=debug` 时）。

### 从文件加载单个值 <a href="#loading-individual-values-from-files" id="loading-individual-values-from-files"></a>

Vaultwarden 支持从磁盘加载配置选项的值（通过环境变量或在 `ENV_FILE` 中设置）。您可以通过将 `_FILE` 添加到相关配置选项并将值设置为包含该值的文件的路径来实现此目的。

如果您想使用 `docker secrets` 这样的功能，这非常有用。例如，通过设置 `SMTP_PASSWORD_FILE=/run/secrets/smtp_password` ，它将从文件加载 SMTP 密码，而不将其作为环境变量提供给进程或容器）。

### 使用 `/admin` 页面 <a href="#using-the-admin-page" id="using-the-admin-page"></a>

此外，Vaultwarden 还可以使用 `config.json` 文件进行配置，该文件可以通过 `/admin` 面板生成和编辑，并保存在数据文件夹中。

{% hint style="info" %}
虽然从技术上讲可以手动创建和编辑 `config.json` 文件，**但我们强烈建议不要这样做**。[JSON](https://www.json.org/) 具有相当严格的语法，如果您不知道自己在做什么，这可能会成为调试的噩梦。
{% endhint %}

`config.json` 中的设置将覆盖任何其他配置方法，并且您将在启动时收到警告哪些设置会被覆盖。

由于生成的 `config.json` 在保存时将包含**所有**可编辑选项，因此请注意，一旦通过 `/admin` 页面生成配置文件，您就无法通过任何其他方法修改这些选项（至少在不修改或删除配置的情况下无法修改 `config.json` 文件）。

{% hint style="danger" %}
只读配置部分中的选项**无法**通过 `/admin` 页面修改，因为它们需要重新启动服务器，并且如果您手动将它们添加到 `config.json` 并单击“保存”，**它们将被移除**。请使用上述其他方法来修改它们。在大多数情况下，这意味着您还需要重新创建容器！
{% endhint %}

## 设置域名 URL <a href="#setting-the-domain-url" id="setting-the-domain-url"></a>

确保将 `DOMAIN` 环境变量（或配置文件中的 `domain`）设置为您的 Vaultwarden 实例的基础 URL。如果不这样做，可能会出现莫名其妙的功能性问题。一些示例：

* `https://vaultwarden.example.com`
* `https://vaultwarden.example.com:8443`（非默认端口）
* `https://host.example.com/vaultwarden`（[子目录托管](using-an-alternate-base-dir-subdir-subpath.md) - 尽可能避免 URL 重写）

## 有关不同配置选项的更多信息 <a href="#further-information-about-different-configuration-options" id="further-information-about-different-configuration-options"></a>

* 邀请和注册设置
  * [禁用邀请](disable-invitations.md)
  * [禁用新用户注册](disable-registration-of-new-users.md)
* 管理后台
  * [启用管理页面](enabling-admin-page.md)
  * [禁用管理令牌](disable-the-admin-token.md)
* [SMTP 配置](smtp-configuration.md)
* 通知
  * [启用 WebSocket 通知](enabling-websocket-notifications.md)
  * [启用移动客户端推送通知](enabling-mobile-client-push-notification.md)
* 2FA 设置
  * [启用 U2F 和 FIDO2 WebAuthn 身份验证](enabling-u2f-and-fido2-webauthn-authentication.md)
  * [启用 YubiKey OTP 身份验证](enabling-yubikey-otp-authentication.md)
* [日志记录](logging.md)
* [其他配置](other-configuration.md)
  * [更改持久数据位置](changing-persistent-data-location.md)
  * [更改 API 请求大小限制](changing-the-api-request-size-limit.md)
  * [更改 worker 数量](changing-the-number-of-workers.md)
* [翻译电子邮件模板](translating-the-email-templates.md)

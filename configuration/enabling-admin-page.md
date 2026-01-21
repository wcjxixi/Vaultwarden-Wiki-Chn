# 2.启用管理页面

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-admin-page)
{% endhint %}

{% hint style="warning" %}
强烈建议在启用此功能之前激活 HTTPS，以避免潜在的 [MITM](https://zh.wikipedia.org/wiki/%E4%B8%AD%E9%97%B4%E4%BA%BA%E6%94%BB%E5%87%BB) 攻击。
{% endhint %}

Vaultwarden 管理面板允许服务器管理员配置 Vaultwarden，查看所有已注册的用户和组织，以及删除它们。它也允许邀请新用户，即使禁用了注册功能。它还提供了一个诊断页面，您可以在其中生成支持字符串。

<figure><img src="https://github.com/user-attachments/assets/7deeb859-b84a-45d6-97ab-50932ce8a6a0" alt=""><figcaption></figcaption></figure>

## 如何启用管理页面 <a href="#how-to-enable-the-admin-page" id="how-to-enable-the-admin-page"></a>

要启用管理页面，您需要配置一个身份验证令牌。该令牌可以是任何内容，但建议使用一个长且随机生成的字符串，例如，通过运行 `openssl rand -base64 48` 来生成。

**请保管好这个令牌。如果您将其配置为 `ADMIN_TOKEN` ，它将用作访问服务器管理区域的密码！**&#x7531;于配置通常以明文形式存储，建议[保护管理令牌](enabling-admin-page.md#secure-the-admin_token)。

您也可以通过[禁用管理令牌](../alternative-deployments/disable-the-admin-token.md)来启用管理员面板。由于这会给予管理面板无限制的访问权限，因此您只有在清楚自己在做什么的情况下才应该这样做。

### 会话管理 <a href="#session-management" id="session-management"></a>

如果您输入 `ADMIN_TOKEN` 的密码，您将获得一个授权您使用 `/admin` 面板的 JSON Web Token (JWT)。管理会话默认长度[设置为 20 分钟](https://github.com/dani-garcia/vaultwarden/blob/0c6817cb4e24964deaf765fd676da6c49e47d099/src/config.rs#L776-L777)。您可以通过更改 `ADMIN_SESSION_LIFETIME` 来配置会话长度。

由于 JWT 的特性以及管理面板没有额外的会话处理，任何拥有有效 JWT 的人都可以使用存储的令牌访问 Vaultwarden 管理页面。更改会话有效期甚至管理令牌本身都不会影响当前已登录的用户，因此您应避免不必要地增加管理会话长度。

要使任何会话失效，可以从 `DATA_FOLDER` 中移除 [`rsa_key.pem`](../backup/backing-up-your-vault.md#the-rsa_key-files) 然后重启 Vaultwarden 以重新创建 RSA 密钥。

## 禁用管理页面 <a href="#disabling-the-admin-page" id="disabling-the-admin-page"></a>

要禁用管理页面，请确保没有设置 `ADMIN_TOKEN` 或 `DISABLE_ADMIN_TOKEN` 环境变量，并且 `config.json`（如果该文件存在）中不存在 `"admin_token"` 键。之后重新创建容器并重启 Vaultwarden 以使更改生效。

## 保护 ADMIN\_TOKEN <a href="#secure-the-admin_token" id="secure-the-admin_token"></a>

> **\[译者注]**：此功能自 [1.28.0+](https://github.com/dani-garcia/vaultwarden/releases/tag/1.28.0) 后可用。

您可以通过使用 Argon2 生成 [PHC 字符串](https://github.com/P-H-C/phc-string-format/blob/master/phc-sf-spec.md)来对 `ADMIN_TOKEN` 进行哈希处理。

PHC 字符串可以通过[使用内置的 `hash` 命令](enabling-admin-page.md#using-vaultwarden-hash)或[使用 `argon2` CLI 工具](enabling-admin-page.md#using-argon2)生成。

### 使用 `vaultwarden hash` <a href="#using-vaultwarden-hash" id="using-vaultwarden-hash"></a>

Vaultwarden 内置了一个 PHC 生成器，您可以通过 CLI 调用 `vaultwarden hash` 来运行它。默认情况下，此命令使用 [Bitwarden 的默认设置](https://github.com/bitwarden/clients/blob/04d1fbb716bc7676c60a009906e183bb3cbb6047/libs/common/src/enums/kdfType.ts#L8-L10)（m=64 MiB，t=3 迭代，p=4 线程）。您可以通过传递 `--preset owasp` 来使用 [OWASP 最低的推荐设置](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html#argon2id)（m=19MiB，t=2，p=1）。

Vaultwarden hash 命令会要求输入两次密码，如果两次输入的密码相同，则会输出生成的 PHC 字符串。

运行该命令的一些示例：

```shell
# 直接使用 vaultwarden 二进制文件
./vaultwarden hash

# 通过 docker 并创建一个临时容器
docker run --rm -it vaultwarden/server /vaultwarden hash

# 通过正在运行的容器上的 docker（相应地替换 vwcontainer）
docker exec -it vwcontainer /vaultwarden hash
```

### 使用 `argon2` <a href="#using-argon2" id="using-argon2"></a>

您还可以使用大多数 Linux 发行版上提供的 `argon2` CLI。

```sh
# 使用 Bitwarden 默认
echo -n "MySecretPassword" | argon2 "$(openssl rand -base64 32)" -e -id -k 65540 -t 3 -p 4
# 输出：$argon2id$v=19$m=65540,t=3,p=4$bXBGMENBZUVzT3VUSFErTzQzK25Jck1BN2Z0amFuWjdSdVlIQVZqYzAzYz0$T9m73OdD2mz9+aJKLuOAdbvoARdaKxtOZ+jZcSL9/N0

# 使用 OWASP 最低的推荐设置
echo -n "MySecretPassword" | argon2 "$(openssl rand -base64 32)" -e -id -k 19456 -t 2 -p 1
# 输出：$argon2id$v=19$m=19456,t=2,p=1$cXpKdUxHSWhlaUs1QVVsSStkbTRPQVFPSmdpamFCMHdvYjVkWTVKaDdpYz0$E1UgBKjUCD2Roy0jdHAJvXihugpG+N9WcAaR8P6Qn/8
```

### 使用已生成的 PHC 字符串 <a href="#using-the-generated-phc-string" id="using-the-generated-phc-string"></a>

在环境变量中使用已生成的 PHC 字符串作为管理令牌，或者将 PHC 字符串传递给 docker/podman CLI 命令。对于 `docker-compose.yml` 文件，请按照以下说明操作。

如果您通过 `/admin` 页面配置了 Vaultwarden，您应该将字符串粘贴到 `Admin token/Argon2 PHC` 字段（在 General settings 中）：

<figure><img src="https://github.com/user-attachments/assets/52bf60df-1880-41b2-aab7-eac9982f7505" alt=""><figcaption></figcaption></figure>

设置 PHC 字符串后，您可以使用生成 PHC 字符串时使用的密码登录，例如上述示例中的 `MySecretPassword` 来登录。

{% hint style="info" %}
如果您可以将整个 `$argon2id$…` PHC 字符串作为管理密码输入，那么您可能正在使用一个过时的 Vaultwarden 版本，该版本尚未支持 argon2id。请确保您至少在使用[最新版本](https://github.com/dani-garcia/vaultwarden/releases/latest)。
{% endhint %}

### 如何防止 `docker-compose.yml` 中的变量插值 <a href="#how-to-prevent-variable-interpolation-in-docker-compose.yml" id="how-to-prevent-variable-interpolation-in-docker-compose.yml"></a>

当[使用 Docker Compose](../container-image-usage/using-docker-compose.md) 并且您通过 `environment` 指令配置 `ADMIN_TOKEN` 时，您需要使用两个美元符号 `$$` 来转义已生成的 argon2 PHC 字符串中出现的所有五个美元符号 `$` 以防止[变量插值](https://docs.docker.com/compose/compose-file/#interpolation)，例如：

```yaml
  environment:
    ADMIN_TOKEN: $$argon2id$$v=19$$m=19456,t=2,p=1$$UUZxK1FZMkZoRHFQRlVrTXZvS0E3bHpNQW55c2dBN2NORzdsa0Nxd1JhND0$$cUoId+JBUsJutlG4rfDZayExfjq4TCt48aBc9qsc3UI
```

这可以自动完成，例如通过在上面的 `argon2` 命令行的末尾添加 `| sed 's#\$#\$\$#g'` 并使用 sed 来完成。

否则您将收到警告消息，变量将无法正确设置：

```
WARNING: The argon2id variable is not set. Defaulting to a blank string.
WARNING: The v variable is not set. Defaulting to a blank string.
WARNING: The m variable is not set. Defaulting to a blank string.
...
```

{% hint style="info" %}
当为 `docker-compose.yaml` 使用 `.env` 文件时，不需要变量插值。如下面的示例所示。在这种情况下，只需使用单个 `$` 变体。与使用 docker/podman CLI 时使用 `-e ADMIN_TOKEN` ，或者在[配置为 Vaultwarden 使用 `ENV_FILE`](configuration-overview.md#using-an-env_file) 的方式相同。
{% endhint %}

```
/docker-data
├── .env
├── docker-compose.yaml
├── vaultwarden/data
```

**.env：**

_确保在 docker-compose 所使用的 `env` 文件中使用单引号。_

```systemd
VAULTWARDEN_ADMIN_TOKEN='$argon2id$v=19$m=65540,t=3,p=4$MmeK.....'
```

**docker-compose.yaml：**

```yaml
services:
  vaultwarden:
    image: ghcr.io/dani-garcia/vaultwarden
    container_name: vaultwarden
    restart: unless-stopped
    volumes:
      - /path/to/vaultwarden/data/:/data/
    environment:
      - ADMIN_TOKEN=${VAULTWARDEN_ADMIN_TOKEN}
```

您可以通过调用 `docker compose config` 来检查您的配置，您应该会看到 `$` 符号已转义成了两个 `$$`。

### 故障排除提示 <a href="#troubleshooting-tips" id="troubleshooting-tips"></a>

如果您一直收到 `You are using a plain text ADMIN_TOKEN which is insecure.` 消息，则说明您要么已经通过管理界面保存了设置，环境变量将不会被使用（请参阅配置优先级）。或者您需要验证是否使用了正确的格式。

您需要确保配置的 PHC 字符串被正确传递给 Vaultwarden，以避免实际值被加上不必要的引号（如 `'` 或 `"` ）包围，以及避免美元符号 `$` 被重复转义为 `$$`，比如把\
`$argon2id$v=19$m=65540…` 变成 `$$argon2id$$v=19$$m=65540…` 。

如果您使用环境变量传递了该配置，可以通过调用 `printenv ADMIN_TOKEN` （或者如果您使用 Docker，则运行 `docker exec vwcontainer printenv ADMIN_TOKEN` ）来检查输出结果是否仅返回配置的 PHC 字符串，例如：

```yaml
$argon2id$v=19$m=65540,t=3,p=4$bXBGMENBZUVzT3VUSFErTzQzK25Jck1BN2Z0amFuWjdSdVlIQVZqYzAzYz0$T9m73OdD2mz9+aJKLuOAdbvoARdaKxtOZ+jZcSL9/N0
```

或者，如果您使用管理页面配置 Vaultwarden，可以通过运行 `grep admin_token data/config.json` 来检查是否返回预期的 PHC 字符串，如下所示：

```yaml
  "admin_token": "$argon2id$v=19$m=65540,t=3,p=4$bXBGMENBZUVzT3VUSFErTzQzK25Jck1BN2Z0amFuWjdSdVlIQVZqYzAzYz0$T9m73OdD2mz9+aJKLuOAdbvoARdaKxtOZ+jZcSL9/N0",
```

## 使用 Vaultwarden 管理面板 <a href="#using-the-vaultwarden-admin-panel" id="using-the-vaultwarden-admin-panel"></a>

### 设置 <a href="#settings" id="settings"></a>

您在管理页面首次保存配置时，将在您的 `DATA_FOLDER` 中生成一个名为 `config.json` 的文件。该文件中的值将优先于相应的环境变量。

{% hint style="warning" %}
创建 `config.json` 会为当前配置中的大多数值设置默认值，因此您将来需要使用管理面板来配置您的实例。唯一的例外是只读部分中的配置选项以及更高级的配置选项。
{% endhint %}

在您点击 `Save` 按钮之前，管理页面中的配置更改是不会生效的。例如，如果您正在测试 SMTP 设置，您更改了 `SMTP Auth mechanism` 设置，然后点击 `Send test email` 来测试更改，这将不会像预期的那样工作 -- 因为您没有点击 `Save`，`SMTP Auth mechanism` 的更改不会生效。

### 用户 <a href="#users" id="users"></a>

用户概览允许您管理所有用户账户，并检查他们是否已完成注册，他们加入了哪些组织以及他们的用户角色是什么。组织的颜色表示用户的当前角色：<mark style="color:blue;">蓝色</mark>表示普通用户，<mark style="color:green;">绿色</mark>表示经理/自定义角色，<mark style="color:purple;">紫色</mark>表示管理员，<mark style="color:orange;">橙色</mark>表示所有者。

{% embed url="https://github.com/user-attachments/assets/ffb94abc-9ce4-4be5-ac87-d89e51e5b7b1" %}

通过右侧的操作，您可以移除 2FA 提供程序，为用户取消授权任何现有会话，以及禁用或删除任何用户。

如果您点击组织按钮，也可以更改指定成员的角色。

{% embed url="https://github.com/user-attachments/assets/1910822f-a297-431a-9309-8262c6563b5e" %}

由于组织至少需要一位所有者，因此您无法移除最后一位所有者的所有者角色。

您也无法通过管理面板将用户添加到组织中。您只能将组织中的现有成员晋升为其他角色。

### 组织 <a href="#organizations" id="organizations"></a>

在组织概览中，您可以删除任何组织。由于您无法删除组织的最后一位所有者，您可能必须先删除所有者的组织。

{% embed url="https://github.com/user-attachments/assets/88444e11-04ca-430c-a2e4-8fba2f126ad9" %}

### 诊断 <a href="#diagnostics" id="diagnostics"></a>

诊断页面会收集一些基本信息，有助于定位某些配置错误，同时也会检查是否有可用更新。这也是您可以生成支持字符串的页面，该字符串会自动收集您系统中的最重要信息，并使其易于分享到我们的问题追踪系统（以及我们的支持论坛）。

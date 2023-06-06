# 4.启用管理页面

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-admin-page)
{% endhint %}

**重要**：强烈建议在启用此功能之前激活 HTTPS，以避免潜在的 [MITM](https://zh.wikipedia.org/wiki/%E4%B8%AD%E9%97%B4%E4%BA%BA%E6%94%BB%E5%87%BB) 攻击。

该页面允许服务器管理员查看并删除所有已注册的用户。它也允许邀请新用户，即使禁用了注册功能。

要启用管理页面，您需要设置一组身份验证令牌。该令牌可以是任何字符，但建议使用随机生成的长字符串，比如运行 `openssl rand -base64 48` 命令生成。

**此令牌是您访问服务器管理区域的密码！请确保其安全性。**您该如何[确保管理令牌的安全](enabling-admin-page.md#secure-the-admin\_token)。

要设置令牌，请使用 `ADMIN_TOKEN` 变量：

```shell
docker run -d --name vaultwarden \
  -e ADMIN_TOKEN=some_random_token_as_per_above_explanation \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

此后，管理页面将在 `/admin` 子目录中可用。

在管理页面首次保存设置时，将自动在 `DATA_FOLDER` 文件夹中生成 `config.json` 文件。该文件中的值优先于环境变量值。

需要注意的是，在您点击 `Save` 按钮之前，管理页面中的配置更改是不会生效的。例如，如果您正在测试 SMTP 设置，您更改了 `SMTP Auth mechanism` 设置，然后点击 `Send test email` 来测试更改，这将不会像预期的那样工作 -- 因为您没有点击 `Save`，`SMTP Auth mechanism` 的更改不会生效。

**注意**：更改 `ADMIN_TOKEN` 后，当前已登录的任何管理员仍可以使用他们现有的登录会话直到到期。管理会话生命周期是[可配置的](https://github.com/dani-garcia/vaultwarden/blob/a13a5bd1d8c3fea3fce80eba6e8c3aa8880855dd/.env.template#L342-L343)，默认为 20 分钟。

## 禁用管理页面 <a href="#disabling-the-admin-page" id="disabling-the-admin-page"></a>

要禁用管理页面，您必须取消设置 `ADMIN_TOKEN` 并重新启动 Vaultwarden。

**注意**：如果环境变量 `ADMIN_TOKEN` 的值持续保留在上述的 `config.json` 文件中，则移除环境变量 `ADMIN_TOKEN` 并不会禁用管理页面。**要禁用管理页面**，请确保没有设置 `ADMIN_TOKEN` 环境变量，并且 `config.json`（如果该文件存在）中不存在 `"admin_token"` 键。

## 保护 ADMIN\_TOKEN <a href="#secure-the-admin_token" id="secure-the-admin_token"></a>

{% hint style="warning" %}
此功能自 [1.28.0+](https://github.com/dani-garcia/vaultwarden/releases/tag/1.28.0) 后可用。
{% endhint %}

{% hint style="danger" %}
优先使用环境变量。

但是，如果您通过管理界面更新了设置，则需要通过相同的 Web 界面更新管理令牌！

请**不要**手动编辑 `config.json` 文件，因为如果操作不当可能会引起故障！
{% endhint %}

{% hint style="info" %}
要在保护令牌后登录管理页面，您需要使用令牌创建期间提供的密码（而不是令牌本身）。
{% endhint %}

以前 `ADMIN_TOKEN` 只能是纯文本格式。您现在可以通过生成 [PHC 字符串](https://github.com/P-H-C/phc-string-format/blob/master/phc-sf-spec.md)来使用 Argon2 对 `ADMIN_TOKEN` 进行哈希处理。这可以通过使用 Vaultwarden 中的内置 `hash` 命令或使用 `argon2` CLI 工具生成。在 Vaultwarden 应用程序中，有两个预设，一个使用 [Bitwarden 默认](https://github.com/bitwarden/clients/blob/04d1fbb716bc7676c60a009906e183bb3cbb6047/libs/common/src/enums/kdfType.ts#L8-L10)的，一个使用 [OWASP 推荐](https://cheatsheetseries.owasp.org/cheatsheets/Password\_Storage\_Cheat\_Sheet.html#argon2id)。

有关如何生成 Argon2id PHC 哈希的一些示例。

### 使用 `vaultwarden hash` <a href="#using-vaultwarden-hash" id="using-vaultwarden-hash"></a>

Vaultwarden 中内置了一个 PHC 生成器，您可以通过 CLI `vaultwarden hash` 运行它。这可以通过已经运行的实例上的 `docker exec` 来完成，或者通过在您自己的系统上的 docker 在本地运行它。

下面使用 `vwcontainer` 作为容器名称，请将其替换为您的实例的正确容器名称。Vaultwarden CLI 会要求输入两次密码，如果两次相同，它将输出已生成的 PHC 字符串。

示例：

```sh
# 使用 Bitwarden 默认（默认的预设值）
# Via docker on a running container
docker exec -it vwcontainer /vaultwarden hash

# 通过 docker 并创建一个临时容器
docker run --rm -it vaultwarden/server /vaultwarden hash

# 直接使用 vaultwarden 二进制文件
./vaultwarden hash

# 使用 OWASP 最低的推荐设置
# Via docker on a running container
docker exec -it vwcontainer /vaultwarden hash --preset owasp

# 通过 docker 并创建一个临时容器
docker run --rm -it vaultwarden/server /vaultwarden hash --preset owasp

# 直接使用 vaultwarden 二进制文件
./vaultwarden hash --preset owasp
```

### 使用 `argon2` <a href="#using-argon2" id="using-argon2"></a>

您还可以使用大多数 Linux 发行版上提供的 `argon2` CLI。

```bash
# 使用 Bitwarden 默认
echo -n "MySecretPassword" | argon2 "$(openssl rand -base64 32)" -e -id -k 65540 -t 3 -p 4
# 输出：$argon2id$v=19$m=65540,t=3,p=4$bXBGMENBZUVzT3VUSFErTzQzK25Jck1BN2Z0amFuWjdSdVlIQVZqYzAzYz0$T9m73OdD2mz9+aJKLuOAdbvoARdaKxtOZ+jZcSL9/N0

# 使用 OWASP 最低的推荐设置
echo -n "MySecretPassword" | argon2 "$(openssl rand -base64 32)" -e -id -k 19456 -t 2 -p 1
# 输出：$argon2id$v=19$m=19456,t=2,p=1$cXpKdUxHSWhlaUs1QVVsSStkbTRPQVFPSmdpamFCMHdvYjVkWTVKaDdpYz0$E1UgBKjUCD2Roy0jdHAJvXihugpG+N9WcAaR8P6Qn/8
```

请在您的 docker/podman CLI 命令中使用这些字符串。对于 `docker-compose.yml` 文件，请按照以下说明操作。如果您使用的是已有的设置，请不要忘记通过管理界面更新您的密码/令牌。

### 如何防止 `docker-compose.yml` 中的变量插值 <a href="#how-to-prevent-variable-interpolation-in-docker-compose.yml" id="how-to-prevent-variable-interpolation-in-docker-compose.yml"></a>

当[使用 Docker Compose](../container-image-usage/using-docker-compose.md) 并通过 `environment` 指令配置 `ADMIN_TOKEN` 时，您需要使用两个美元符号 `$$` 来转义已生成的 argon2 PHC 字符串中出现的所有五个美元符号 `$` 以防止[变量插值](https://docs.docker.com/compose/compose-file/#interpolation)，例如：

```
  environment:
    ADMIN_TOKEN: $$argon2id$$v=19$$m=19456,t=2,p=1$$UUZxK1FZMkZoRHFQRlVrTXZvS0E3bHpNQW55c2dBN2NORzdsa0Nxd1JhND0$$cUoId+JBUsJutlG4rfDZayExfjq4TCt48aBc9qsc3UI
```

这可以自动完成，例如通过添加 `| sed 's#$#$$#g'` 到上面的 `argon2` 命令行的末尾。

否则您将收到警告消息并且变量将无法正确设置：

```
WARNING: The argon2id variable is not set. Defaulting to a blank string.
WARNING: The v variable is not set. Defaulting to a blank string.
WARNING: The m variable is not set. Defaulting to a blank string.
...
```

**注意：**为 `docker-compose.yaml` 使用 `.env` 文件时情况并非如此。如下所示。在这种情况下，只需使用单个 `$` 变量。与使用 `-e ADMIN_TOKEN` 的 docker/podman cli 相同。

```
/docker-data
├── .env
├── docker-compose.yaml
├── vaultwarden/data
```

**.env：**

```
VAULTWARDEN_ADMIN_TOKEN=$argon2id$v=19$m=65540,t=3,p=4$MmeK.....
```

{% hint style="warning" %}
Compose 按字面解释 `.env` 文件中等号后的每个字符。所以这里需要省略单引号。
{% endhint %}

**docker-compose.yaml：**

```docker
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

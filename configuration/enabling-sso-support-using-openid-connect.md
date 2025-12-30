# 8.使用 OpenId Connect 启用 SSO 支持

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-SSO-support-using-OpenId-Connect)
{% endhint %}

## 使用 OpenId Connect 的 SSO <a href="#sso-using-openid-connect" id="sso-using-openid-connect"></a>

要使用外部身份验证源，您的 SSO 需要支持 OpenID Connect：

* OpenID Connect 发现端点需要可用
* 客户端认证将使用 ID 和 Secret 完成

仍然需要主密码，且不由 SSO 控制（根据您的观点，这可能是一个功能）。这引入了另一种控制谁可以使用密码库的方式，而无需使用邀请或使用 LDAP。

## 配置 <a href="#configuration" id="configuration"></a>

以下配置可用：

* `SSO_ENABLED`：启用 SSO
* `SSO_ONLY` ：禁用电子邮箱 + 主密码认证
* `SSO_SIGNUPS_MATCH_EMAIL` ：在 SSO 注册时，如果存在具有相同电子邮件地址的用户，则进行关联（默认 `true` ）
* `SSO_ALLOW_UNKNOWN_EMAIL_VERIFICATION` ：允许未知电子邮箱验证状态（默认 `false` ）。允许此功能与 `SSO_SIGNUPS_MATCH_EMAIL` 结合使用可能会带来账户接管的风险。
* `SSO_AUTHORITY` ：您的 SSO 的 OpenID Connect Discovery 端点
  * 不应包含 `/.well-known/openid-configuration` 部分，且无 `/` 尾随
  * $SSO\_AUTHORITY/.well-known/openid-configuration 应返回一个 JSON 文档：[https://openid.net/specs/openid-connect-discovery-1\_0.html#ProviderConfigurationResponse](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfigurationResponse)
* `SSO_SCOPES` ：可选，允许在需要时覆盖范围（默认 `"email profile"` ）
* `SSO_AUTHORIZE_EXTRA_PARAMS` ：可选，允许在授权重定向时添加额外参数（默认 `""` ）
* `SSO_PKCE` ：为授权代码流程激活 PKCE（默认 `true` ）。
* `SSO_AUDIENCE_TRUSTED` ：可选，用于信任 ID Token 的额外受众的正则表达式（ `client_id` 始终受信任）。编写正则表达式时使用单引号： `'^$'` 。
* `SSO_CLIENT_ID` ：客户端 ID
* `SSO_CLIENT_SECRET` ：客户端机密
* `SSO_MASTER_PASSWORD_POLICY` ：可选的主密码策略（不支持 `enforceOnLogin` ）。
* `SSO_AUTH_ONLY_NOT_SESSION` ：启用的话表示仅用于身份验证，不用于会话生命周期
* `SSO_CLIENT_CACHE_EXPIRATION` ：缓存对发现端点的调用，持续时间（秒）， `0` 禁用（默认 `0` ）;
* `SSO_DEBUG_TOKENS` ：记录所有令牌以便于调试（默认 `false` ，需要设置 `LOG_LEVEL=debug` 或 `LOG_LEVEL=info,vaultwarden::sso=debug` ）

回调 URL 为： `https://your.domain/identity/connect/oidc-signin`

## 账户和电子邮箱处理 <a href="#account-and-email-handling" id="account-and-email-handling"></a>

使用 SSO 登录时，一个标识符（来自 IdToken 的 `{iss}/{sub}` 声明）会保存在一个单独的表（`sso_users`）中。该标识符用于链接到 SSO 提供程序标识符，而无需更改默认用户 `uuid`。之所以需要这样做，是因为：

* 存储 SSO 标识对于防止因更改电子邮箱而导致账户被接管非常重要。
* 我们不能使用标识符作为用户 uuid，因为它太长了（`sub` 部分最多 255 个字符，参见[规范](https://openid.net/specs/openid-connect-core-1_0.html#CodeIDToken)）。
* 我们希望能根据电子邮箱关联现有账户，但仅限于用户首次登录时（由 `SSO_SIGNUPS_MATCH_EMAIL` 控制）。
* 我们需要能够关联现有的存根账户，例如在邀请用户加入组织时创建的账户（只有在用户没有私钥的情况下才能关联）。

此外：

* 如果提供程序报告电子邮箱 `unverified`，注册将被阻止。
* 更改电子邮箱需要用户自己完成，因为这需要更新 `key`。登录时，如果提供程序返回的电子邮箱不是已保存的电子邮箱，系统将向用户发送一封电子邮件，要求他更新电子邮箱。
* 如果设置了 `SIGNUPS_DOMAINS_WHITELIST`，则会在 SSO 注册和尝试更改电子邮箱时应用。

这意味着，如果需要更改提供程序网址或提供程序本身，必须先删除关联，然后确保 `SSO_SIGNUPS_MATCH_EMAIL` 已激活，以允许新的关联。

要删除关联（这对 `Vaultwarden` 用户没有影响）：

```sql
TRUNCATE TABLE sso_users;
```

### 对于 `SSO_ALLOW_UNKNOWN_EMAIL_VERIFICATION`  <a href="#on-sso_allow_unknown_email_verification" id="on-sso_allow_unknown_email_verification"></a>

如果提供程序不发送电子邮箱的验证状态（`email_verified` [claim](https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims)），则需要激活此设置。

如果设置为 `SSO_SIGNUPS_MATCH_EMAIL=true`（默认值），那么即使用户不控制电子邮箱地址，也可以与现有的非 SSO 账户关联。这样，用户就可以访问敏感信息，但仍需要主密码才能读取密码。

因此，在使用 `SSO_ALLOW_UNKNOWN_EMAIL_VERIFICATION` 时，建议禁用 `SSO_SIGNUPS_MATCH_EMAIL`。如果需要关联非 SSO 用户，请尽量在最短时间内同时激活这两个设置。

## 客户端缓存 <a href="#client-cache" id="client-cache"></a>

默认情况下，客户端缓存是禁用的，因为它会导致签名密钥出现问题。

这意味着每次我们需要与提供程序交互（生成 authorize\_url、交换授权代码、刷新令牌）时，都要再次调用发现端点。这种情况并不理想，因此 `SSO_CLIENT_CACHE_EXPIRATION` 允许您配置一个适合您的提供程序的过期时间。

如果 `IdToken` 验证失败，客户端缓存就会失效（但您会定期遇到一个倒霉的用户 ^^），这是防止过期时间配置错误的一种保护措施。

### Google 示例（滚动密钥） <a href="#google-example-rolling-keys" id="google-example-rolling-keys"></a>

以 Goole 为例，在检查发现[端点](https://accounts.google.com/.well-known/openid-configuration)响应报头时，我们可以看到缓存控制的 `max-age` 被设置为 `3600` 秒。而 [jwk\_uri](https://www.googleapis.com/oauth2/v3/certs) 响应头通常包含一个更大值的 `max-age`。结合用户[反馈](https://github.com/ramosbugs/openidconnect-rs/issues/152)，我们可以得出结论：Goole 每周都会更新签名密钥。

缓存过期时间设置过高会导致回报率降低，但使用 `600`（10 分钟）这样的过期时间应该会带来很多好处。

### 手动滚动密钥 <a href="#rolling-keys-manually" id="rolling-keys-manually"></a>

如果要滚动已使用的密钥，请先添加一个新的密钥，但不要立即开始使用它签名。等待在 `SSO_CLIENT_CACHE_EXPIRATION` 中配置的延迟时间，然后就可以开始使用它签名了。

正如 Google 示例中提到的，即使不打算滚动密钥，设置过高的值也会导致回报率降低。

## Keycloak

默认访问令牌生命周期可能只有 `5min`，请设置更大的值，否则会与同样设置为 `5min`的 Bitwarden 前端过期检测冲突。

在领域级别：

* `Realm settings / Tokens / Access Token Lifespan` 至少设置为 `10min`（使用 `kcadm.sh` 时的 `accessTokenLifespan` 设置）。
* `Realm settings / Sessions / SSO Session Idle/Max` 用于刷新令牌的生命周期

或者对于在 `Clients / Client details / Advanced / Advanced settings` 中的特定客户端，可以找到 `Access Token Lifespan` 和 `Client Session Idle/Max`。

服务器配置，无特别的设置：

* `SSO_AUTHORITY=https://${domain}/realms/${realm_name}`
* `SSO_CLIENT_ID`
* `SSO_CLIENT_SECRET`

### 测试 <a href="#testing" id="testing"></a>

如果您想运行 Keycloak 的测试实例，可以使用 Playwright [docker-compose](https://github.com/dani-garcia/vaultwarden/blob/main/playwright/docker-compose.yml)。具体使用方法请参阅 [README.md](https://github.com/dani-garcia/vaultwarden/blob/main/playwright/README.md#openid-connect-test-setup)。

## Auth0

由于以下问题 [https://github.com/ramosbugs/openidconnect-rs/issues/23](https://github.com/ramosbugs/openidconnect-rs/issues/23)（它们似乎没有遵循规范），其无法正常工作。目前提供了一个功能标志 (`oidc-accept-rfc3339-timestamps`) 可用于绕过这个问题，但您需要用它来编译服务器。目前还没有计划始终激活该功能或为 Auth0 制作专门的发行版本。

## Authelia

要获取 `refresh_token` 以扩展会话，您需要添加 `Offline_Access` 范围。

配置看起来如下：

* `SSO_SCOPES="email profile offline_access"`

## Authentik

默认访问令牌生命周期可能只有 `5min`，请设置更大的值，否则会与同样设置为 `5min` 的 Bitwarden 前端过期检测产生冲突。

要更改令牌生命周期，请转到 `Applications / Providers / Edit / Advanced protocol settings`。

从 2024.2 版本开始，您需要添加 `Offline_Access` 范围，并确保在 `Applications / Providers / Edit / Advanced protocol settings / Scopes` 中选中它（[文档](https://docs.goauthentik.io/docs/providers/oauth2/#authorization_code)）。

服务器配置看起来应如下：

* `SSO_AUTHORITY=https://${domain}/application/o/${application_name}/`：尾部的 `/` 很重要
* `SSO_SCOPES="email profile offline_access"`
* `SSO_CLIENT_ID`
* `SSO_CLIENT_SECRET`

### 疑难解答 <a href="#troubleshooting" id="troubleshooting"></a>

* `Failed to discover OpenID provider`  / `Failed to parse server response`（ 检测 OpenID 提供程序失败 / 解析服务器响应失败）：
  * 首先确保可以访问添加了 `/.well-known/openid-configuration` 的 Authority 端点。
  * 然后检查文件是否返回了 `id_token_signing_alg_values_supported: ["RS256"]`。如果返回 `HS256`，那么再次选择默认签名密钥应该能解决该问题（[步骤](https://github.com/Timshel/vaultwarden/issues/107#issuecomment-3200007338)）。
* `Failed to contact token endpoint: Parse(Error ... Invalid JSON web token: found 5 parts`：该错误可能是由加密令牌 (JWE) 导致的，请确保未使用加密密钥（[步骤](https://github.com/dani-garcia/vaultwarden/issues/6230#issuecomment-3245196399)）。

## Casdoor

自版本 [v1.639.0](https://github.com/casdoor/casdoor/releases/tag/v1.639.0) 起应该可以正常工作（已使用版本 [v1.686.0](https://github.com/casdoor/casdoor/releases/tag/v1.686.0) 进行测试）。创建应用程序时，您需要选择 `Token format -> JWT-Standard`。

然后使用以下内容配置您的服务器：

* `SSO_AUTHORITY=https://${provider_host}`
* `SSO_CLIENT_ID`
* `SSO_CLIENT_SECRET`

## GitLab

使用以下内容在 Gitlab 设置中创建一个应用程序：

* `redirectURI`：[https://your.domain/identity/connect/oidc-signin](https://your.domain/identity/connect/oidc-signin)
* `Confidential`：`true`
* `scopes`：`openid`, `profile`, `email`

然后使用以下内容配置您的服务器：

* `SSO_AUTHORITY=https://gitlab.com`
* `SSO_CLIENT_ID`
* `SSO_CLIENT_SECRET`

## Google Auth

Google [文档](https://developers.google.com/identity/openid-connect/openid-connect?hl=zh-cn)。默认情况下，如果没有额外[配置](https://developers.google.com/identity/protocols/oauth2/web-server#creatingclient)，您将没有 `refresh_token`，会话时间将限制为 1 小时。

使用以下内容配置您的服务器：

* `SSO_AUTHORITY=https://accounts.google.com`
* `SSO_AUTHORIZE_EXTRA_PARAMS="access_type=offline&prompt=consent"`
* `SSO_CLIENT_ID`
* `SSO_CLIENT_SECRET`

## Kanidm

不需要特殊配置。

使用以下内容配置您的服务器：

* `SSO_AUTHORITY=https://your.domain/oauth2/openid/${SSO_CLIENT_ID}`
* `SSO_CLIENT_ID`
* `SSO_CLIENT_SECRET`

## Microsoft Entra ID

1. 在 [Entra ID](https://entra.microsoft.com/) 中按照 [Identity | Applications | App registrations](https://entra.microsoft.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade/quickStartType//sourceType/Microsoft_AAD_IAM) 创建「应用程序注册」。
2. 在「应用程序注册」的「概述」中，您需要「目录（租户）ID」以用于 `SSO_AUTHORITY` 变量，「应用程序（客户端）ID」以用于 `SSO_CLIENT_ID` 值。
3. 在「证书和秘钥」中创建「应用程序秘钥」，您需要「秘钥值」以用于 `SSO_CLIENT_SECRET`。
4. 在「身份验证」中添加 https://warden.example.org/identity/connect/oidc-signin 作为「网页重定向 URI」。
5. 在「API 权限」中，确保在「API / 权限名称」下列出 `profile`、`email` 和 `offline_access`（`offline_access` 是必需的，否则不会返回 `refresh_token`，请参阅 [https://github.com/MicrosoftDocs/azure-docs/issues/17134](https://github.com/MicrosoftDocs/azure-docs/issues/17134)）。

只有 v2 端点符合 OpenID 规范，请参阅 [https://github.com/MicrosoftDocs/azure-docs/issues/38427](https://github.com/MicrosoftDocs/azure-docs/issues/38427) 和 [https://github.com/ramosbugs/openidconnect-rs/issues/122](https://github.com/ramosbugs/openidconnect-rs/issues/122)。

您的服务器配置看起来应如下：

* `SSO_AUTHORITY=https://login.microsoftonline.com/${Directory_ID}/v2.0` #租户
* `SSO_SCOPES=openid profile offline_access User.Read`
* `SSO_CLIENT_ID=${Application_ID}` #客户端
* `SSO_CLIENT_SECRET=${Secret_Value}`

## Rauthy

要使用提供程序控制的会话，您需要在运行 `Rauthy` 时使用 `DISABLE_REFRESH_TOKEN_NBF=true`，否则服务器在尝试读取尚未生效的 `refresh_token` 时会失败（即使 `access_token` 有效，Bitwarden 客户端也会触发刷新。详情参阅 [rauthy](https://github.com/sebadob/rauthy/issues/651)）。另一种方法是使用 `SSO_AUTH_ONLY_NOT_SESSION=true` 的默认会话处理方式。

创建客户端时不需要特殊配置。

您的配置看起来应如下：

* `SSO_AUTHORITY=http://${provider_host}/auth/v1`
* `SSO_CLIENT_ID=${Client ID}`
* `SSO_CLIENT_SECRET=${Client Secret}`
* `SSO_AUTH_ONLY_NOT_SESSION=true`：仅在不使用 `DISABLE_REFRESH_TOKEN_NBF=true` 运行 `Rauthy` 时才需要

## Slack

您需要在 [https://api.slack.com/apps/](https://api.slack.com/apps/) 中创建一个 App。

看起来返回的 `access_token` 不是 JWT 格式，并且没有使用它发送到期日期。因此，您需要使用默认的会话生命周期。

您的配置看起来应如下：

* `SSO_AUTHORITY=https://slack.com`
* `SSO_CLIENT_ID=${Application Client ID}`
* `SSO_CLIENT_SECRET=${Application Client Secret}`
* `SSO_AUTH_ONLY_NOT_SESSION=true`

## Zitadel

要获取 `refresh_token` 以扩展会话，您需要添加 `Offline_Access` 范围。

此外，Zitadel 还在 Id Token 的受众中加入了 `Project id` 和客户 `Client Id`。为使验证生效，您需要将 `Resource Id` 添加为受信任的受众（默认情况下 `Client Id` 是受信任的）。您可以使用 `SSO_AUDIENCE_TRUSTED` 配置来控制受信任的受众。

根据 [Zitadel#9200](https://github.com/zitadel/zitadel/issues/9200)，`id_token` 会传递一个受信任受众列表，其中包括 `Project Id`。如果最终有许多受信任的 `aud` 字符串，`SSO_AUDIENCE_TRUSTED` 可能会变得难以管理。在这种情况下，`SSO_AUDIENCE_TRUSTED: '^\d{18}$'`（18 是 `aud` 列表中每个字符串的大小，可能因 Zitadel 实现而异）会对您有所帮助，但单独添加所有审核字符串也是安全的，如 `SSO_AUDIENCE_TRUSTED:'^abcd|def|xyz$'`。

自 [zitadel#721](https://github.com/zitadel/oidc/pull/721) 以来，PKCE 应该可以使用客户端机密。但旧版本可能需要禁用它（`SSO_PKCE=false`）。

配置看起来应如下：

* `SSO_AUTHORITY=https://${provider_host}`
* `SSO_SCOPES="email profile offline_access"`
* `SSO_CLIENT_ID`
* `SSO_CLIENT_SECRET`
* `SSO_AUDIENCE_TRUSTED='^${Project Id}$'`

## 会话生命周期 <a href="#session-lifetime" id="session-lifetime"></a>

会话生命周期取决于刷新令牌和调用 SSO 令牌端点（授权类型：`authorization_code`）后返回的访问令牌。如果没有返回刷新令牌，会话将仅限于访问令牌的生命周期。

令牌不会持久保存在服务器中，而是封装在 JWT 令牌中并返回给应用程序（VW `identity/connect/token` 端点返回的 `refresh_token` 和 `access_token` 值）。请注意，出于与 Web 前端兼容的原因，服务器将始终返回一个 `refresh_token`，它的存在并不表明 SSO 返回了刷新令牌（但您可以使用 [https://jwt.io](https://jwt.io/) 对其值进行解码，然后检查 `token` 字段是否包含任何内容）。

有了刷新令牌，应用程序中的活动就会在访问令牌快过期时（网页客户端为 [5 分钟](https://github.com/bitwarden/clients/blob/0bcb45ed5caa990abaff735553a5046e85250f24/libs/common/src/auth/services/token.service.ts#L126)）触发刷新。

此外，在执行某些操作时还会进行令牌检查，如果有刷新令牌，就会执行刷新，否则就会调用用户信息端点来检查访问令牌的有效性。

### 禁用 SSO 会话处理 <a href="#disabling-sso-session-handling" id="disabling-sso-session-handling"></a>

如果无法获取 `efresh_token` 或出于其他原因，可以禁用 SSO 会话处理，以恢复到默认的处理方式。您需要启用 `SSO_AUTH_ONLY_NOT_SESSION=true`，然后访问令牌的有效期将为 2 小时，刷新令牌的空闲时间为 7 天（可无限延长）。

### 调试信息 <a href="#debug-information" id="debug-information"></a>

使用 `LOG_LEVEL=debug` 运行，您就能看到令牌过期的信息。

## 桌面客户端 <a href="#desktop-client" id="desktop-client"></a>

在处理从浏览器（用于 SSO 登录）到应用程序的重定向方面存在一些问题。

## Chrome

一些用户报告有说存在[问题](https://github.com/bitwarden/clients/issues/12929)。

## Firefox

在 Windows 系统中，首次登录时会出现一个提示，以确认应启动哪个应用程序（但目前存在一个 bug，即登录后可能会出现空密码库）。

在 Linux 系统上，稍微麻烦些。首先，您需要在 `about:config` 中添加一些配置：

```systemd
network.protocol-handler.expose.bitwarden=false
network.protocol-handler.external.bitwarden=true
```

如果您有任何疑问，可以检查 `mailto` 以查看它是如何配置的。

重定向仍然不起作用，因为与应用程序的关联似乎只能通过链接/点击来完成。您可以用一个虚拟页面来触发它，例如：

```html
data:text/html,<a href="bitwarden:///dummy">Click me to register Bitwarden</a>
```

从现在起，重定向应该可以正常工作了。如果需要更改启动的应用程序，现在可以使用搜索功能在 `Settings`  中输入 `application` 进行查找。

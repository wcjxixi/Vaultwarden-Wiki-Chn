# =8.使用 OpenId Connect 启用 SSO 支持

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-SSO-support-using-OpenId-Connect)
{% endhint %}

{% hint style="danger" %}
‼️ ‼️ ‼️

SSO 目前仅适用于 `:testing` 标签的镜像！\
当前的稳定版 `v1.34.3` **不包含** SSO 功能。

‼️ ‼️ ‼️
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
* `SSO_AUTHORITY` ：您的 SSO 的 OpenID Connect 发现端点
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

回调 URL： `https://your.domain/identity/connect/oidc-signin`

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

如果提供程序不发送电子邮件的验证状态（`email_verified` [claim](https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims)），则需要激活此设置。

如果设置为 `SSO_SIGNUPS_MATCH_EMAIL=true`（默认值），那么即使用户不控制电子邮箱地址，也可以与现有的非 SSO 账户关联。这样，用户就可以访问敏感信息，但仍需要主密码才能读取密码。

因此，在使用 `SSO_ALLOW_UNKNOWN_EMAIL_VERIFICATION` 时，建议禁用 `SSO_SIGNUPS_MATCH_EMAIL`。如果需要关联非 SSO 用户，请尽量在最短时间内同时激活这两个设置。

## 客户端缓存 <a href="#client-cache" id="client-cache"></a>

### Google 示例（滚动密钥） <a href="#google-example-rolling-keys" id="google-example-rolling-keys"></a>

### 手动滚动密钥 <a href="#rolling-keys-manually" id="rolling-keys-manually"></a>

## Keycloak

### 测试 <a href="#testing" id="testing"></a>

## Auth0

## Authelia

## Authentik

### 疑难解答 <a href="#troubleshooting" id="troubleshooting"></a>

## Casdoor

## GitLab

## Google Auth

## Kanidm

## Microsoft Entra ID

## Rauthy

## Slack

## Zitadel

## 会话生命周期 <a href="#session-lifetime" id="session-lifetime"></a>

### 禁用 SSO 会话处理 <a href="#disabling-sso-session-handling" id="disabling-sso-session-handling"></a>

### 调试信息 <a href="#debug-information" id="debug-information"></a>

## 桌面客户端 <a href="#desktop-client" id="desktop-client"></a>

### Chrome

## Firefox

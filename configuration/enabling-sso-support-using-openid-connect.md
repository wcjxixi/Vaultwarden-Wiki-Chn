# =8.使用 OpenId Connect 启用 SSO 支持

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-SSO-support-using-OpenId-Connect)
{% endhint %}

{% hint style="danger" %}
‼️ ‼️ ‼️

SSO 目前仅适用于 `:testing` 标记的镜像！\
当前的稳定版 v1.34.3 **不包含** SSO 功能。

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

回调 URL 是： `https://your.domain/identity/connect/oidc-signin`

## 账户和电子邮箱处理 <a href="#account-and-email-handling" id="account-and-email-handling"></a>

### 对于 `SSO_ALLOW_UNKNOWN_EMAIL_VERIFICATION` <a href="#on-sso_allow_unknown_email_verification" id="on-sso_allow_unknown_email_verification"></a>

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

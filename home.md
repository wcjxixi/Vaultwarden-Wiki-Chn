# 首页

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki)
{% endhint %}

Vaultwarden 是一个使用 Rust 编写的非官方 Bitwarden 服务器实现，它与[官方 Bitwarden 客户端](https://bitwarden.com/download/)兼容，非常适合不希望运行官方资源密集型服务的自托管部署。

Vaultwarden 面向个人、家庭和小型组织。尽管开发主要针对大型组织的功能（例如单点登录、目录同步等）并不是优先事项，但欢迎能实现此类功能的高质量 PR。

Vaultwarden 已经进行了多项审计，其中一些是公开的，请在我们的 [Vaultwarden 审计](faq/audits.md) wiki 页面上阅读更多相关信息。

> \[**译者注**]：PR 指 GitHub 中的 [pull requests](https://docs.github.com/cn/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)（拉取请求）。

## 支持的功能 <a href="#supported-features" id="supported-features"></a>

Vaultwarden 实现了大多数功能所需的 Bitwarden API，其中包括：

* 网页界面（等效于 [https://vault.bitwarden.com/](https://vault.bitwarden.com/\))）
* 个人密码库支持
* [组织](https://help.ppgg.in/admin-console/organizations-quick-start)密码库支持
* [群组](https://help.ppgg.in/admin-console/organization-basics/groups)（需要设置[环境变量](https://github.com/dani-garcia/vaultwarden/blob/main/.env.template#L409-L414)才能启用它）
* [事件日志](https://help.ppgg.in/admin-console/reporting/event-logs)（需要设置[环境变量](https://github.com/dani-garcia/vaultwarden/blob/main/.env.template#L275-L278)才能启用它）
* [密码共享](https://help.ppgg.in/password-manager/vault-basics/sharing)和[访问控制](https://help.ppgg.in/admin-console/user-management/member-roles-and-permissions)
* [集合](https://help.ppgg.in/admin-console/organization-basics/collections)
* [文件附件](https://help.ppgg.in/password-manager/vault-basics/file-attachments)
* [文件夹](https://help.ppgg.in/password-manager/vault-administration/folders)
* [收藏](https://help.ppgg.in/password-manager/vault-administration/favorites)
* [网站图标](https://help.ppgg.in/security/website-icons)
* [Bitwarden 验证器 (TOTP)](https://help.ppgg.in/password-manager/vault-basics/totp)
* [Bitwarden Send](https://help.ppgg.in/password-manager/bitwarden-send/about-send)
* [紧急访问](https://help.ppgg.in/my-account/more/emergency-access)
* [回收站](https://help.ppgg.in/password-manager/vault-basics/vault-items#vault-trash)（软删除，您可以[配置自动删除的等待天数](https://github.com/dani-garcia/vaultwarden/blob/main/.env.template#L234-L237)）
* [主密码重新提示](https://help.ppgg.in/password-manager/vault-basics/vault-items#protect-individual-items)
* [个人 API 密钥](https://help.ppgg.in/password-manager/developer-tools/personal-api-key-for-cli-authentication)
* [电子邮件](https://help.ppgg.in/my-account/two-step-login/setup-guides/two-step-login-via-email)、[Duo](https://help.ppgg.in/my-account/two-step-login/setup-guides/two-step-login-via-duo)、[Yubikey](https://help.ppgg.in/my-account/two-step-login/setup-guides/two-step-login-via-yubikey) 和 [FIDO2 WebAuthn](https://help.ppgg.in/my-account/two-step-login/setup-guides/two-step-login-via-fido)（包括 Nitrokeys 和 Solokeys）方式的两步登录
* SimpleLogin、AnonAddy 或 Firefox Relay 的用户名生成器集成
* [目录连接器](https://help.ppgg.in/admin-console/user-management/directory-connector/about-directory-connector)支持
* [账户恢复](https://help.ppgg.in/docs/admin-console/manage-members/account-recovery/about-account-recovery)（这需要[启用电子邮件功能](configuration/smtp-configuration.md)）
* 用于桌面端/浏览器客户端/扩展的[实时同步](https://bitwarden.com/blog/live-sync/)（仅 WebSocket）
* 用于移动客户端 (Android/iOS) 的[实时同步](https://bitwarden.com/blog/live-sync/)（[推送通知](configuration/enabling-mobile-client-push-notification.md)）
* [单点登录 (SSO)](https://help.ppgg.in/admin-console/login-with-sso/about-login-with-sso)，请参阅[文档](configuration/enabling-sso-support-using-openid-connect.md)
* 某些企业策略：
  * [要求两步登录](https://help.ppgg.in/admin-console/organization-basics/enterprise-policies#require-two-step-login)
  * [主密码要求](https://help.ppgg.in/admin-console/organization-basics/enterprise-policies#master-password-requirements)
  * [账户恢复管理](https://help.ppgg.in/docs/admin-console/oversight-visibility/enterprise-policies#account-recovery-administration)（仅在启用了电子邮件时可用）
  * [密码生成器](https://help.ppgg.in/admin-console/organization-basics/enterprise-policies#password-generator)
  * [单一组织](https://help.ppgg.in/admin-console/organization-basics/enterprise-policies#single-organization)
  * [禁用个人密码库](https://help.ppgg.in/admin-console/organization-basics/enterprise-policies#remove-individual-vault)
  * [禁用 Send](https://help.ppgg.in/admin-console/organization-basics/enterprise-policies#disable-send)
  * [Send 选项](https://help.ppgg.in/admin-console/organization-basics/enterprise-policies#send-options)
  * [禁用支付卡项目类型](https://help.ppgg.in/docs/admin-console/oversight-visibility/enterprise-policies#remove-card-item-type)
  * [默认 URI 匹配检查](https://help.ppgg.in/docs/admin-console/oversight-visibility/enterprise-policies#default-uri-match-detection)

## 缺少的功能 <a href="#missing-features" id="missing-features"></a>

话题 [#246](https://github.com/dani-garcia/vaultwarden/issues/246) 包含了完整的功能请求列表，其中既有官方服务器在 Vaultwarden 中缺失的功能，也有 Vaultwarden 中特有的增强功能。

为了与官方服务器做简单的比较，本章节总结了官方服务器中已经实现但在 Vaultwarden 中目前还没有实现的功能。

在时间允许的情况下，可能会添加的功能（欢迎贡献）：

* [Bitwarden 公共 API](https://help.ppgg.in/organizations/bitwarden-public-api) / [组织 API 密钥](https://help.ppgg.in/admin-console/bitwarden-public-api#authentication)。此功能做了部分添加，并且仅支持 Bitwarden Directory Connector
* [使用通行密钥登录](https://help.ppgg.in/docs/account/log-in-and-unlock/more-log-in-options/log-in-with-passkeys)。参阅 [#5929](https://github.com/dani-garcia/vaultwarden/pull/5929)
* [新设备登录保护](https://help.ppgg.in/docs/account/log-in-and-unlock/new-device-protection)

除非做出贡献，否则可能不会添加的功能：

* [自定义角色](https://help.ppgg.in/admin-console/user-management/member-roles-and-permissions#custom-role)
* 某些企业策略（[UI 非开源](https://github.com/bitwarden/clients/tree/main/bitwarden_license/bit-web/src/app/admin-console/policies)。可能需要通过管理页面进行配置）：
  * [要求单点登录验证](https://help.ppgg.in/admin-console/organization-basics/enterprise-policies#require-single-sign-on-authentication)
  * [密码库超时](https://help.ppgg.in/admin-console/organization-basics/enterprise-policies#vault-timeout)
  * [禁用个人密码库导出](https://help.ppgg.in/admin-console/organization-basics/enterprise-policies#disable-personal-vault-export)

## 联系我们 <a href="#get-in-touch" id="get-in-touch"></a>

要提出问题、提供建议、请求新功能或获得有关配置或安装软件的帮助，请[使用论坛](https://vaultwarden.discourse.group)。

如果您发现任何与 Vaultwarden 本身有关的 bug 或崩溃，请[创建一个话题](https://github.com/dani-garcia/vaultwarden/issues)。并确保不存在任何类似的话题！

我们通常在 [#vaultwarden:matrix.org](https://matrix.to/#/#vaultwarden:matrix.org) 房间闲逛，如果您喜欢聊天，欢迎随时加入我们！

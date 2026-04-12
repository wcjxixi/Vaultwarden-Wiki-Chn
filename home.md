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
* [电子邮件](https://help.ppgg.in/my-account/two-step-login/setup-guides/two-step-login-via-email)、[Duo](https://help.ppgg.in/my-account/two-step-login/setup-guides/two-step-login-via-duo)、[YubiKey](https://help.ppgg.in/my-account/two-step-login/setup-guides/two-step-login-via-yubikey) 和 [FIDO2 WebAuthn](https://help.ppgg.in/my-account/two-step-login/setup-guides/two-step-login-via-fido)（包括 Nitrokeys 和 Solokeys）方式的两步登录
* SimpleLogin、AnonAddy 或 Firefox Relay 的用户名生成器集成
* [Directory Connector](https://help.ppgg.in/docs/admin-console/manage-members/directory-connector/about-directory-connector) 支持
* [账户恢复](https://help.ppgg.in/docs/admin-console/manage-members/account-recovery/about-account-recovery)（这需要[启用电子邮箱](configuration/smtp-configuration.md)）
* 用于桌面端/浏览器客户端/扩展的[实时同步](https://bitwarden.com/blog/live-sync/)（仅 WebSocket）
* 用于移动客户端 (Android/iOS) 的[实时同步](https://bitwarden.com/blog/live-sync/)（[推送通知](configuration/enabling-mobile-client-push-notification.md)）
* [单点登录 (SSO)](https://help.ppgg.in/admin-console/login-with-sso/about-login-with-sso)，请参阅[文档](configuration/enabling-sso-support-using-openid-connect.md)
* 某些企业策略：
  * [要求两步登录](https://help.ppgg.in/admin-console/organization-basics/enterprise-policies#require-two-step-login)
  * [主密码要求](https://help.ppgg.in/admin-console/organization-basics/enterprise-policies#master-password-requirements)
  * [账户恢复管理](https://help.ppgg.in/docs/admin-console/oversight-visibility/enterprise-policies#account-recovery-administration)（仅在启用了电子邮箱时可用）
  * [密码生成器](https://help.ppgg.in/admin-console/organization-basics/enterprise-policies#password-generator)
  * [单一组织](https://help.ppgg.in/admin-console/organization-basics/enterprise-policies#single-organization)
  * [禁用个人密码库](https://help.ppgg.in/admin-console/organization-basics/enterprise-policies#remove-individual-vault)
  * [禁用 Send](https://help.ppgg.in/admin-console/organization-basics/enterprise-policies#disable-send)
  * [Send 选项](https://help.ppgg.in/admin-console/organization-basics/enterprise-policies#send-options)
  * [禁用支付卡项目类型](https://help.ppgg.in/docs/admin-console/oversight-visibility/enterprise-policies#remove-card-item-type)
  * [默认 URI 匹配检测](https://help.ppgg.in/docs/admin-console/oversight-visibility/enterprise-policies#default-uri-match-detection)

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

***

<p align="center">🛡️ Vaultwarden — 一款使用 Rust 重构的 Bitwarden 服务器</p>

<p align="center"><sup><sub><strong>🏠</strong></sub></sup> <a href="home.md"><sup><sub><strong>Wiki 首页</strong></sub></sup></a> <sup><sub>·</sub><sub> </sub><sub><strong>📖</strong></sub></sup><a href="faq/faqs.md"> <sup><sub><strong>FAQ</strong></sub></sup></a> <sup><sub>· <strong>⚙️</strong></sub></sup> <a href="configuration/configuration-overview.md"><sup><sub><strong>配置</strong></sub></sup></a> <sup><sub>· <strong>🔒</strong></sub></sup> <a href="configuration/security/hardening-guide.md"><sup><sub><strong>强化指南</strong></sub></sup></a> <sup><sub>· <strong>🐳</strong></sub></sup> <a href="container-image-usage/using-docker-compose.md"><sup><sub><strong>Docker</strong></sub></sup></a></p>

***

<p align="center"><sup><sub><strong>💬 联系我们</strong></sub></sup></p>

<p align="center"><a href="https://vaultwarden.discourse.group/"><img src="https://camo.githubusercontent.com/765e304c5ba051b74fd1ed0262d20e8c08523e0855b802e23d9d88c5c4ad85f0/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f466f72756d2d446973636f757273652d3235393662653f7374796c653d666c61742d737175617265266c6f676f3d646973636f75727365" alt="Forum"></a> <a href="https://matrix.to/#/#vaultwarden:matrix.org"><img src="https://camo.githubusercontent.com/49c753898876a5f895bfdd8ee1ebfabdcca4f1789652998fd55e400ce4cda88a/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f436861742d2532337661756c7477617264656e2533416d61747269782e6f72672d3064626438623f7374796c653d666c61742d737175617265266c6f676f3d6d6174726978" alt="Matrix"></a> <a href="https://github.com/dani-garcia/vaultwarden/issues"><img src="https://camo.githubusercontent.com/fbdda3c37b06d0bb5d65af5d0621973db08d963f4e4cb1de525f4ad50c59f680/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f6973737565732f64616e692d6761726369612f7661756c7477617264656e3f7374796c653d666c61742d737175617265266c6f676f3d676974687562" alt="Issues"></a> <a href="https://github.com/dani-garcia/vaultwarden/stargazers"><img src="https://camo.githubusercontent.com/325e1350f77def0d7dc32c11fc5ba604fb0456bc1aa2fd42dda84ee6177b7d5b/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f73746172732f64616e692d6761726369612f7661756c7477617264656e3f7374796c653d666c61742d737175617265266c6f676f3d676974687562" alt="Stars"></a> <a href="https://github.com/dani-garcia/vaultwarden/blob/main/LICENSE.txt"><img src="https://camo.githubusercontent.com/21f489b23b45334005af96c4ea5b058b0628f4eddbf59d8b40ce40e187791e51/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f6c6963656e73652f64616e692d6761726369612f7661756c7477617264656e3f7374796c653d666c61742d737175617265" alt="License"></a></p>

***

<p align="center"><sup><sub><strong>❤️ 喜欢</strong></sub><sub> </sub><sub>Vaultwarden</sub><sub> </sub><sub><strong>吗？</strong></sub><sub>考虑</sub></sup><a href="faq/supporting-upstream-development.md"><sup><sub>支持上游的 Bitwarden</sub></sup></a> <sup><sub>— 没有他们的工作，这个项目就不会存在。</sub></sup></p>

<p align="center"><sup><sub>Vaultwarden 是一个</sub><sub><strong>非官方</strong></sub><sub>的、由社区驱动的兼容 Bitwarden 的服务器。它与 Bitwarden, Inc 无任何关联、附属，亦未获得其认可 —「Bitwarden」是 Bitwarden, Inc 的商标。</sub></sup></p>

<p align="center"><sup><sub>由</sub></sup> <a href="https://github.com/dani-garcia"><sup><sub>@dani-garcia</sub></sup></a> <sup><sub>和</sub></sup><a href="https://github.com/dani-garcia/vaultwarden/graphs/contributors"><sup><sub>贡献者</sub></sup></a><sup><sub>精心维护 · Wiki 内容遵循项目条款授权</sub></sup></p>

***

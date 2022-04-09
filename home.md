# 主页

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki)
{% endhint %}

Vaultwarden 是一个使用 Rust 编写的非官方 Bitwarden 服务器实现，它与[官方 Bitwarden 客户端](https://bitwarden.com/download/)兼容，对于不希望使用官方的占用大量资源的自托管部署而言，它是理想的选择。

Vaultwarden 面向个人、家庭和小型组织。尽管开发主要对大型组织有用的功能（例如，单点登录、目录同步等）并不是优先事项，但欢迎实现此类功能的高质量 PR。

## 支持的特性 <a href="#supported-features" id="supported-features"></a>

Vaultwarden 实现了 Bitwarden API 所需的大部分功能，其中包括：

* 网页界面（等效于 [https://vault.bitwarden.com/](https://vault.bitwarden.com/\))）
* 个人密码库支持
* [组织](https://help.ppgg.in/getting-started/getting-started-with-organizations)密码库支持
* [密码共享](https://help.ppgg.in/organizations/sharing)和[访问控制](https://help.ppgg.in/organizations/user-types-and-access-control)
* [集合](https://help.ppgg.in/organizations/collections)
* [文件附件](https://help.ppgg.in/your-vault/file-attachments)
* [文件夹](https://help.ppgg.in/your-vault/folders)
* [收藏](https://help.ppgg.in/your-vault/favorites)
* [网站图标](https://help.ppgg.in/security/privacy-when-using-website-icons)
* [Bitwarden 验证器（TOTP）](https://help.ppgg.in/your-vault/bitwarden-authenticator-totp)
* [Bitwarden Send](https://help.ppgg.in/bitwarden-send/about-send)
* [紧急访问](https://help.ppgg.in/security/emergency-access)
* 用于桌面端/浏览器客户端/扩展的[实时同步](https://bitwarden.com/blog/post/live-sync/)（仅 WebSocket）
* [回收站](https://help.ppgg.in/your-vault/vault-items#items-in-the-trash)（软删除）
* [主密码重新提示](https://help.ppgg.in/your-vault/vault-items#protect-individual-items)
* [个人 API 密钥](https://help.ppgg.in/miscellaneous/personal-api-key-for-cli-authentication)
* [电子邮件](https://help.ppgg.in/two-step-login/two-step-login-via-email)、[Duo](https://help.ppgg.in/two-step-login/two-step-login-via-duo)、[Yubikey](https://help.ppgg.in/two-step-login/two-step-login-via-yubikey) 和 [FIDO2 WebAuthn](https://help.ppgg.in/two-step-login/two-step-login-via-fido2-webauthn)（包括 Nitrokeys 和 Solokeys）方式的两步登录
* [目录连接器支持](https://help.ppgg.in/directory-connector/about-directory-connector)（基本实现，但不支持群组）\
  仅支持 [v2.9.2](https://github.com/bitwarden/directory-connector/releases/tag/v2.9.2) 及更低版本，v2.9.3 及更高版本使用不同的登录方式，因此尚不被支持。
* 某些企业策略：
  * [两步登录](https://help.ppgg.in/organizations/enterprise-policies#two-step-login)
  * [主密码](https://help.ppgg.in/organizations/enterprise-policies#two-step-login)
  * [密码生成器](https://help.ppgg.in/organizations/enterprise-policies#password-generator)
  * [个人所有权](https://help.ppgg.in/organizations/enterprise-policies#personal-ownership)
  * [禁用 Send](https://help.ppgg.in/organizations/enterprise-policies#disable-send)
  * [Send 选项](https://help.ppgg.in/organizations/enterprise-policies#send-options)
  * [单一组织](https://help.ppgg.in/organizations/enterprise-policies#single-organization)

## 缺少的特性 <a href="#missing-features" id="missing-features"></a>

话题 [#246](https://github.com/dani-garcia/vaultwarden/issues/246) 包含了完整的功能请求列表，既有官方服务器中缺少的功能，也有 Vaultwarden 中特有的增强功能。

为了与官方服务器做简单的比较，本章节总结了官方服务器中已经实现但在 Vaultwarden 中目前还没有实现的功能。

在时间允许的情况下，可能会添加的功能（欢迎贡献）：

* [Bitwarden 公共 API](https://help.ppgg.in/organizations/bitwarden-public-api) / [组织 API 密钥](https://help.ppgg.in/organizations/bitwarden-public-api#authentication)
* [事件日志](https://help.ppgg.in/organizations/event-logs)
* 用于移动客户端（Android/iOS）的[实时同步](https://bitwarden.com/blog/post/live-sync/)（推送通知）
* [管理员密码重置](https://help.ppgg.in/organizations/admin-password-reset)
* 某些企业策略：
  * [主密码重置](https://help.ppgg.in/organizations/enterprise-policies#master-password-reset)

除非做出贡献，否则可能不会添加的功能：

* [单点登录（SSO）](https://help.ppgg.in/login-with-sso/about-login-with-sso)
* [群组](https://help.ppgg.in/organizations/groups)
* [自定义角色](https://help.ppgg.in/organizations/user-types-and-access-control#custom-role)
* 某些企业策略（[UI 不是开源的](https://github.com/bitwarden/web/tree/master/bitwarden\_license/src/app/policies)，可能需要通过管理页面进行配置）：
  * [密码库超时](https://help.ppgg.in/organizations/enterprise-policies#vault-timeout)
  * [禁用个人密码库导出](https://help.ppgg.in/organizations/enterprise-policies#disable-personal-vault-export)

## 保持联系 <a href="#get-in-touch" id="get-in-touch"></a>

要提出问题、提供建议、请求新功能或获得有关配置或安装软件的帮助，请[使用论坛](https://vaultwarden.discourse.group)。

如果您发现任何与 Vaultwarden 本身有关的 bug 或崩溃，请[创建一个话题](https://github.com/dani-garcia/vaultwarden/issues)。并先确保不存在任何类似的话题！

我们通常在 [#vaultwarden:matrix.org](https://matrix.to/#/#vaultwarden:matrix.org) 房间闲逛，如果您喜欢聊天，欢迎随时加入我们！

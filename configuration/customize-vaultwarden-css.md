# 3.自定义 Vaultwarden CSS

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Customize-Vaultwarden-CSS/)
{% endhint %}

{% hint style="info" %}
此功能仅适用于 v1.33.0 及更高版本。
{% endhint %}

从 v1.33.0 版本开始，您可以修改 Vaultwarden 之前嵌入到 web-Vault 中的 CSS。这样，用户可以方便地调整样式和布局，甚至隐藏项目。

要修改 CSS，您需要在 `data` 目录中添加 `templates` 目录，或通过 `TEMPLATES_FOLDER` 环境变量提供正确的路径。

在此目录中，您需要创建另一个名为 `scss` 的目录，该目录将保存用于修改 Vaultwarden 服务样式表的文件。

您可以在此处放置两个文件：

* **`user.vaultwarden.scss.hbs`**：该文件是您要编辑并向其中添加自定义样式的文件。
* **`vaultwarden.scss.hbs`**：该文件不应该存在，因为它将覆盖内置默认值。_**只有当您真正知道自己在做什么时才可以覆盖此设置！**_

```
.
├── templates
│   └── scss
│       ├── user.vaultwarden.scss.hbs
│       └── vaultwarden.scss.hbs
```

**可以将其放置在 `user.vaultwarden.scss.hbs` 中的一些示例：**

```css
/* 隐藏 "验证器 App" 2FA (列表第一项) */
app-two-factor-setup ul.list-group.list-group-2fa li.list-group-item:nth-child(1) {
  @extend %vw-hide;
}

/* 隐藏 "YubiKey OTP 安全钥匙" 2FA (列表第二项) */
/* 注意：如果 Yubikey 配置为净设置，该选项也将自动隐藏 */
app-two-factor-setup ul.list-group.list-group-2fa li.list-group-item:nth-child(2) {
  @extend %vw-hide;
}

/* 隐藏 "Duo" 2FA (列表第三项) */
app-two-factor-setup ul.list-group.list-group-2fa li.list-group-item:nth-child(3) {
  @extend %vw-hide;
}

/* 隐藏 "FIDO2 WebAuthn" 2FA (列表第四项) */
app-two-factor-setup ul.list-group.list-group-2fa li.list-group-item:nth-child(4) {
  @extend %vw-hide;
}

/* 隐藏 "Email" 2FA (列表第五项) */
/* 注意：如果未启用电子邮箱功能，该选项也将自动隐藏。 */
app-two-factor-setup ul.list-group.list-group-2fa li.list-group-item:nth-child(5) {
  @extend %vw-hide;
}

/* 使用自定义登录 logo */
app-root img.new-logo-themed {
	content: url(../images/my-custom-login-logo.png) !important;
}

/* 在登录和锁定界面的左上角使用自定义 logo */
auth-anon-layout > main > a > bit-icon > svg {
  display: none !important;
}
auth-anon-layout > main > a > bit-icon::before {
  display: block;
  content: "" !important;
  width: 175px !important;
  height: 36px !important;
  background-image: url(../images/my-custom-global-logo.png) !important;
  background-repeat: no-repeat !important;
  background-size: contain;
}

/* 使用自定义左上角徽标，密码库和管理页面不一样 */
app-user-layout bit-nav-logo bit-icon > svg,
app-organization-layout bit-nav-logo bit-icon > svg {
  display: none !important;
}
app-user-layout bit-nav-logo bit-icon::before,
app-organization-layout bit-nav-logo bit-icon::before {
  display: block;
  content: "" !important;
  width: 200px !important;
  height: 50px !important;
  background-repeat: no-repeat !important;
  background-size: contain;
}
app-user-layout bit-nav-logo bit-icon::before {
  background-image: url(../images/my-custom-password-manager-logo.png) !important;
}
app-organization-layout bit-nav-logo bit-icon::before {
  background-image: url(../images/my-custom-admin-console-logo.png) !important;
}
```

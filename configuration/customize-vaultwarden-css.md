# 15.自定义 Vaultwarden CSS

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Customize-Vaultwarden-CSS/)
{% endhint %}

{% hint style="info" %}
此功能仅适用于 v1.33.0 及更高版本。
{% endhint %}

从 v1.33.0 版本开始，您可以修改 Vaultwarden 之前嵌入到 web-Vault 中的 CSS。

这样，用户可以更轻松地调整样式和布局，甚至隐藏项目。

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

**您可以将一些示例放置在 `user.vaultwarden.scss.hbs` 中：**

```css
/* Hide `Authenticator app` 2FA (First item of the list) */
app-two-factor-setup ul.list-group.list-group-2fa li.list-group-item:nth-child(1) {
  @extend %vw-hide;
}

/* Hide `YubiKey OTP security key` 2FA (Second item of the list) */
/* Note: This will also be hidden automatically if the Yubikey config is net set */
app-two-factor-setup ul.list-group.list-group-2fa li.list-group-item:nth-child(2) {
  @extend %vw-hide;
}

/* Hide `Duo` 2FA (Third item of the list) */
app-two-factor-setup ul.list-group.list-group-2fa li.list-group-item:nth-child(3) {
  @extend %vw-hide;
}

/* Hide `FIDO2 WebAuthn` 2FA (Fourth item of the list) */
app-two-factor-setup ul.list-group.list-group-2fa li.list-group-item:nth-child(4) {
  @extend %vw-hide;
}

/* Hide `Email` 2FA (Fifth item of the list) */
/* Note: This will also be hidden automatically if email is not enabled */
app-two-factor-setup ul.list-group.list-group-2fa li.list-group-item:nth-child(5) {
  @extend %vw-hide;
}

/* Use a custom login logo */
app-root img.new-logo-themed {
	content: url(../images/my-custom-login.logo.png) !important;
}

/* Use a custom top left logo on login and locked screen page */
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

/* Use a custom top left logo different per vault/admin */
app-user-layout bit-icon > svg,
app-organization-layout bit-icon > svg {
  display: none !important;
}
app-user-layout bit-icon::before,
app-organization-layout bit-icon::before {
  display: block;
  content: "" !important;
  width: 200px !important;
  height: 50px !important;
  background-repeat: no-repeat !important;
  background-size: contain;
}
app-user-layout bit-icon::before {
  background-image: url(../images/my-custom-password-manager-logo.png) !important;
}
app-organization-layout bit-icon::before {
  background-image: url(../images/my-custom-admin-console-logo.png) !important;
}
```

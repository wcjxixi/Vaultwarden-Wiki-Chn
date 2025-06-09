# 3.自定义 Vaultwarden CSS

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Customize-Vaultwarden-CSS/)
{% endhint %}

{% hint style="info" %}
**此功能仅适用于 v1.33.0 及更高版本。**
{% endhint %}

从 v1.33.0 版本开始，您可以修改 Vaultwarden 的 CSS 样式（以前是嵌入在 web-vault 中）。这可以让用户能够轻松调整样式、布局，甚至隐藏特定元素。

要修改 CSS，您需要在 `data` 目录中创建 `templates` 目录，或通过 `TEMPLATES_FOLDER` 环境变量指定正确的路径。

在 `templates` 目录中，还需要创建另一个名为 `scss` 的目录，该目录将用于存放修改的 Vaultwarden CSS 文件。

您可以将以下两个文件放置在 `scss` 目录中：

* **`user.vaultwarden.scss.hbs`**：此文件是您要编辑并向其中添加自定义样式的文件。
* **`vaultwarden.scss.hbs`**：此文件不应存在，因为它将覆盖内置默认值。_**除非您完全清楚操作后果，否则不要覆盖此文件！**_

```
.
├── templates
│   └── scss
│       ├── user.vaultwarden.scss.hbs
│       └── vaultwarden.scss.hbs
```

**以下是可以将其放入 `user.vaultwarden.scss.hbs` 中的示例代码片段：**

```css
/* 隐藏 "验证器 App" 2FA (列表第一项) */
app-two-factor-setup ul.list-group.list-group-2fa li.list-group-item:nth-child(1) {
  @extend %vw-hide;
}

/* 隐藏 "YubiKey OTP 安全密钥" 2FA (列表第二项) */
/* 注意：如果未配置 Yubikey，该选项将自动隐藏 */
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
/* 注意：如果未启用电子邮箱功能，该选项将自动隐藏。 */
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

**固定搜索筛选器（使用「Stylus」浏览器扩展测试）：**

```css
/*
使 "筛选器" 框固定不动，只有条目列表滚动。
这样就能始终看到您筛选的内容，并能快速访问搜索筛选器，而无需滚动返回顶部。
*/
#main-content > app-vault > .tw-flex .tw-basis-1\/4
{
    position: fixed;
    width: calc(25% - 55px);
}

#main-content > app-vault > .tw-flex .tw-basis-3\/4
{
    position: relative;
    left: calc(25% + 10px);
}
```

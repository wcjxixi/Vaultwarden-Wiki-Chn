# 3.自定义 Vaultwarden CSS

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Customize-Vaultwarden-CSS/)
{% endhint %}

{% hint style="info" %}
**此功能仅适用于 v1.33.0 及更高版本。**
{% endhint %}

从 v1.33.0 版本开始，您可以修改 Vaultwarden 的 CSS 样式（预先嵌入在 web-vault 中）。这可以让用户能够轻松调整样式和布局，甚至隐藏特定的元素。

要修改 CSS，您需要在 `data` 目录中创建 `templates` 目录（或通过 `TEMPLATES_FOLDER` 环境变量指定正确的路径）。

在此目录中，还需要创建另一个名为 `scss` 的目录，该目录将用于存放修改的 Vaultwarden CSS 文件。

您可以将以下两个文件放置在 `scss` 目录中：

* **`user.vaultwarden.scss.hbs`**：此文件是您要编辑并向其中添加自定义样式的文件。
* **`vaultwarden.scss.hbs`**：此文件不应存在，因为它将覆盖内置的默认值。_**除非您完全清楚操作后果，否则不要覆盖此文件！**_

```
.
├── templates
│   └── scss
│       ├── user.vaultwarden.scss.hbs
│       └── vaultwarden.scss.hbs
```

**以下是可以将其放入 `user.vaultwarden.scss.hbs` 中的示例代码片段：**

```css
/* 文件位置: /data/templates/scss/user.vaultwarden.scss.hbs */

/* --- 变量 --- */
/* 您可以为密码库（用户）和管理控制台（组织）设置不同的 Logo */
$logo-default: url('/vw_static/logo-gray.png');
$logo-admin:   url('/vw_static/logo-gray.png'); 

/* 侧边栏定制 */
$sidebar-width: 15rem; /* 如果需要，将其设置为匹配您的徽标的宽度 */

/* --- 混入 --- */
@mixin hide-element { display: none !important; }

/* --- 隐藏 2FA 提供程序 --- */
/* 0: Authenticator App, 1: Email, 2: Duo, 3: YubiKey OTP, 7: FIDO2 WebAuthn */
/*
 .providers-2fa-0, .providers-2fa-1, .providers-2fa-2, .providers-2fa-3, .providers-2fa-7 {
  @include hide-element;
}
*/

/* --- 加载界面 --- */
app-root img.new-logo-themed { content: $logo-default !important; }

/* --- 登录界面 --- */
auth-anon-layout bit-landing-header {
  bit-svg {
    /* 隐藏原始 SVG */
    > svg { @include hide-element; }

    /* 注入自定义 Logo */
    &::before {
      display: block !important;
      content: "" !important;
      width: 100% !important;
      height: 42px !important;
      background: $logo-default no-repeat center left !important;
      background-size: contain !important;
    }
  }
}

/* --- 仪表板侧边栏 --- */
bit-nav-logo {
  /* 
    如果您希望 Logo 在最小化时减少裁剪，请应
    如果您有类似 Vaultwarden 和 Bitwarden 的徽标
  */
  /* > div { padding-right: 2px !important; } */

  bit-svg {
    > svg { @include hide-element; }

    &::before {
      display: block !important;
      content: "" !important;
      width: 100% !important;
      height: 42px !important;
      background-repeat: no-repeat !important;
      background-size: auto 42px !important;
      background-position: center left !important;
    }
  }
}

/* --- 仪表板侧边栏 --- */
app-user-layout bit-nav-logo bit-svg::before { background-image: $logo-default !important; }
app-organization-layout bit-nav-logo bit-svg::before { background-image: $logo-admin !important; }

/* --- 侧边栏布局 & 逻辑 --- */
#bit-side-nav {
  /*
    仅当宽度与默认的 '18rem' 内联样式匹配时才覆盖宽度。
    这确保我们不会破坏 '折叠' 状态 (4.5rem) 或手动调整大小。
  */
  &[style*="18rem"] { max-width: $sidebar-width !important; }

  /*
    当侧边栏折叠时（宽度通常为 4.5rem），隐藏自定义 Logo
    以便让它看起来不会破坏或被裁剪。
  */
  &[style*="4.5rem"] {
    bit-nav-logo bit-svg::before { display: none !important; }
    /*
      可选：最小化时再次显示原始图标？
      移除下面的注释以启用：
    */
    /* bit-nav-logo bit-svg > svg { display: block !important; } */
  }
}
```

**固定搜索筛选器（使用「Stylus」浏览器扩展测试）：**

```css
/*
使 "筛选器" 框固定不动，只有条目列表滚动。
这样就能始终看到您筛选的内容，并能快速访问搜索筛选器，而无需滚动返回顶部。
*/
#main-content {
    /*Edit 2025.09.12*/
    contain: none;
}

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

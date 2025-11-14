# 关于

{% hint style="success" %}
2021-04-27：从 [v1.21.0](https://github.com/dani-garcia/vaultwarden/releases/tag/1.21.0) 开始，bitwarden\_rs 项目更名为 Vaultwarden。参阅 [#1642](https://github.com/dani-garcia/vaultwarden/discussions/1642) 了解更多说明。
{% endhint %}

这里是对官方 [Vaultwarden](https://github.com/dani-garcia/vaultwarden)（以前叫 bitwarden\_rs）[Wiki](https://github.com/dani-garcia/vaultwarden/wiki) 的中文翻译。

原文有太多口语化内容，翻译起来比较费脑，这里我尽力翻译准确并使之不那么生硬。

译者：[@wcjxixi](mailto:wcjxixi@gmail.com)

致谢 [Google Translate](https://translate.google.com) 以及 [DeepL](https://www.deepl.com)！

{% hint style="warning" %}
个人能力有限，具体请以官方 [Vaultwarden Wiki](https://github.com/dani-garcia/vaultwarden/wiki) 页面为准。使用本内容所产生的一切后果，与 @wcjxixi 无关。Use at your own risk！！！
{% endhint %}

{% hint style="info" %}
**备注：**&#x6807;题前有 \* 的表示官方曾经有但现已移除的页面和/或分类，我将其保留仅作为参考之用。
{% endhint %}

## Vaultwarden 是什么 <a href="#what-is-vaultwarden" id="what-is-vaultwarden"></a>

Vaultwarden 是一个用于本地搭建 Bitwarden 服务器的第三方 Docker 项目。仅在部署的时候使用 Vaultwarden 镜像，桌面端、移动端、浏览器扩展等客户端均使用官方 Bitwarden 客户端。

Vaultwarden 很轻量，对于不希望使用占用大量资源的官方 Bitwarden 自托管部署而言，它是理想的选择。

## Vaultwarden 与 Bitwarden 的区别 <a href="#difference-between-vaultwarden-and-bitwarden" id="difference-between-vaultwarden-and-bitwarden"></a>

* 除不支持 Bitwarden 官方企业版的部分功能（详情见[这里](home.md#missing-features)）外，其他大部分功能均**免费**支持。并跟随官方版本保持及时更新。
* Vaultwarden 比 Bitwarden 官方版更轻量。官方版使用 .Net 开发，使用 MSSQL 数据库，要求至少 2GB 内存；Vaultwarden 使用 Rust 编写，改用 SQLite 数据库（现在也支持 MySQL 和 PostgreSQL），运行时只需要 10M 内存，可以说对硬件基本没有要求。

## 免费公共实例 <a href="#public-instances" id="public-instances"></a>

使用 Vaultwarden 搭建的免费公共实例。更多实例请自行使用关键词「Vaultwarden Web Vault」进行网页搜索。

{% hint style="danger" %}
注意：请自行决定使用这些公共实例所存在的安全风险。
{% endhint %}

* [https://bitwarden.garudalinux.org/](https://bitwarden.garudalinux.org)
* [https://vault.tedomum.net/](https://vault.tedomum.net)
* [https://passwd.hostux.net/](https://passwd.hostux.net)
* [https://bitwarden.scutech.com](https://bitwarden.scutech.com)

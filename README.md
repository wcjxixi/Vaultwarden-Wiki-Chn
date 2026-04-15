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



***

<p align="center"><strong>🛡️ Vaultwarden — 一款使用 Rust 重构的 Bitwarden 服务器</strong></p>

<p align="center"><a href="home.md"><sup><sub><strong>🏠 Wiki 首页</strong></sub></sup></a> <sup><sub>·</sub></sup> <a href="faq/faqs.md"><sup><sub><strong>📖 FAQ</strong></sub></sup></a> <sup><sub>·</sub></sup> <a href="configuration/configuration-overview.md"><sup><sub><strong>⚙️ 配置</strong></sub></sup></a> <sup><sub>·</sub></sup> <a href="configuration/security/hardening-guide.md"><sup><sub><strong>🔒 强化指南</strong></sub></sup></a> <sup><sub>·</sub></sup> <a href="container-image-usage/using-docker-compose.md"><sup><sub><strong>🐳 Docker</strong></sub></sup></a></p>

***

<p align="center"><sup><sub><strong>💬 保持联系</strong></sub></sup></p>

<p align="center"><a href="https://vaultwarden.discourse.group/"><img src="https://camo.githubusercontent.com/765e304c5ba051b74fd1ed0262d20e8c08523e0855b802e23d9d88c5c4ad85f0/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f466f72756d2d446973636f757273652d3235393662653f7374796c653d666c61742d737175617265266c6f676f3d646973636f75727365" alt="Forum"></a> <a href="https://matrix.to/#/#vaultwarden:matrix.org"><img src="https://camo.githubusercontent.com/49c753898876a5f895bfdd8ee1ebfabdcca4f1789652998fd55e400ce4cda88a/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f436861742d2532337661756c7477617264656e2533416d61747269782e6f72672d3064626438623f7374796c653d666c61742d737175617265266c6f676f3d6d6174726978" alt="Matrix"></a> <a href="https://github.com/dani-garcia/vaultwarden/issues"><img src="https://camo.githubusercontent.com/fbdda3c37b06d0bb5d65af5d0621973db08d963f4e4cb1de525f4ad50c59f680/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f6973737565732f64616e692d6761726369612f7661756c7477617264656e3f7374796c653d666c61742d737175617265266c6f676f3d676974687562" alt="Issues"></a> <a href="https://github.com/dani-garcia/vaultwarden/stargazers"><img src="https://camo.githubusercontent.com/325e1350f77def0d7dc32c11fc5ba604fb0456bc1aa2fd42dda84ee6177b7d5b/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f73746172732f64616e692d6761726369612f7661756c7477617264656e3f7374796c653d666c61742d737175617265266c6f676f3d676974687562" alt="Stars"></a> <a href="https://github.com/dani-garcia/vaultwarden/blob/main/LICENSE.txt"><img src="https://camo.githubusercontent.com/21f489b23b45334005af96c4ea5b058b0628f4eddbf59d8b40ce40e187791e51/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f6c6963656e73652f64616e692d6761726369612f7661756c7477617264656e3f7374796c653d666c61742d737175617265" alt="License"></a></p>

***

<p align="center"><sup><sub><strong>❤️ 喜欢</strong></sub><sub> </sub><sub>Vaultwarden</sub><sub> </sub><sub><strong>吗？</strong></sub><sub>考虑</sub></sup><a href="faq/supporting-upstream-development.md"><sup><sub>支持上游的 Bitwarden</sub></sup></a> <sup><sub>— 没有他们的工作，这个项目就不会存在。</sub></sup></p>

<p align="center"><sup><sub>Vaultwarden 是一款<strong>非官方</strong>的、由社区驱动的兼容 Bitwarden 的服务器。它与 Bitwarden, Inc 无任何关联、附属，亦未获得其认可。</sub></sup><br><sup><sub>—「Bitwarden」是 Bitwarden, Inc 的商标。</sub></sup></p>

<p align="center"><sup><sub>由</sub></sup> <a href="https://github.com/dani-garcia"><sup><sub>@dani-garcia</sub></sup></a> <sup><sub>和</sub></sup><a href="https://github.com/dani-garcia/vaultwarden/graphs/contributors"><sup><sub>贡献者</sub></sup></a><sup><sub>精心维护 · Wiki 内容遵循项目条款许可</sub></sup></p>

***

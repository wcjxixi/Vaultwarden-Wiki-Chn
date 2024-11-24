# 审计

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Audits)
{% endhint %}

## Vaultwarden 审计 <a href="#vaultwarden-audits" id="vaultwarden-audits"></a>

Vaultwarden 已经过安全公司的审计，这有助于确保 Vaultwarden 的安全。

有些审计是在没有公开发布任何数据的情况下完成的，因为要求这些安全公司进行审计的公司不允许这样做，但这些研究人员确实提供了审计结果。

有些审计是公开发布的，任何人都可以访问。

## 由 BSI 进行的审计 <a href="#audit-by-bsi" id="audit-by-bsi"></a>

{% hint style="info" %}
网站和报告均为德语
{% endhint %}

德国安全机构 [BSI (Bundesamt für Sicherheit in der Informationstechnik)](https://www.bsi.bund.de/EN/Home/home_node.html) 在 [CAOS (Codeanalyse von Open Source Software)  项目](https://www.bsi.bund.de/DE/Service-Navi/Publikationen/Studien/Projekt_P486/projekt_P486_node.html)下对 [Vaultwarden v1.30.3](https://github.com/dani-garcia/vaultwarden/releases/tag/1.30.3) 进行了审计。

新闻稿（包含 Vaultwarden 结果的 PDF）可在此处找到：[https: //www.bsi.bund.de/DE/Service-Navi/Presse/Alle-Meldungen-News/Meldungen/Codeanalysis-KeePass-Vaultwarden\_241014 .html](https://www.bsi.bund.de/DE/Service-Navi/Presse/Alle-Meldungen-News/Meldungen/Codeanalyse-KeePass-Vaultwarden_241014.html)

他们甚至还有一个更详细的 ZIP 文件，其中包含所有原始信息： [https://www.bsi.bund.de/SharedDocs/Downloads/DE/BSI/Downloadserver/P486/CAOS\_Vaultwarden.html](https://www.bsi.bund.de/SharedDocs/Downloads/DE/BSI/Downloadserver/P486/CAOS_Vaultwarden.html)

作为参考，您可以在这里下载报告：

* [原文 - 德语 - Vaultwarden-Passwordmanager.pdf](https://github.com/user-attachments/files/17805671/Vaultwarden-Passwortmanager.pdf)
* [翻译 - 英语 - Vaultwarden-Passwortmanager.en.pdf](https://github.com/user-attachments/files/17805672/Vaultwarden-Passwortmanager.en.pdf)

## ERNW Enno Rey Netzwerke GmbH 进行的渗透测试 <a href="#penetration-test-by-ernw-enno-rey-netzwerke-gmbh" id="penetration-test-by-ernw-enno-rey-netzwerke-gmbh"></a>

[ERNW Enno Rey Netzwerke GmbH](https://ernw.de/) 在 2024 年 10 月为客户进行的渗透测试中对 Vaultwarden 进行了评估。2024 年 6 月，德国联邦信息安全办公室 (BSI) 公布了 Vaultwarden 服务器组件的静态和动态测试结果。因此，在评估过程中仅进行了部分源代码审计，同时还重点关注了其他软件和基础设施。ERNW 发现了 3 个漏洞，包括身份验证绕过，并负责任地向 Vaultwarden 披露了这些漏洞。ERNW 的博客 Insinuator 对漏洞进行了描述：

* [https://insinuator.net/2024/11/vulnerability-disclosure-authentication-bypass-in-vaultwarden-versions-1-32-5/](https://insinuator.net/2024/11/vulnerability-disclosure-authentication-bypass-in-vaultwarden-versions-1-32-5/)

# 2.翻译电子邮件模板

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Translating-the-email-templates)
{% endhint %}

如果您[设置了电子邮件](../smtp-configuration.md)，Vaultwarden 将以英语发送邮件。由于服务器不知道用户的首选语言设置（完全在客户端完成），因此目前不太可能在同一服务器上为不同用户提供多种语言。

如果您的用户不懂英语，您可以将提供的标头模板翻译成您的首选语言。

## 如何翻译/定制模板？ <a href="#how-to-translate-customize-the-templates" id="how-to-translate-customize-the-templates"></a>

您可以通过如下方式使用自定义模板：

1. 将相应文件从存储库（`src/static/templates/email`）复制到具有相同文件夹结构的对应的 `TEMPLATES_FOLDER` 下（例如复制到 `data/templates/email`）
2. 更改它们（比如翻译它们 - 但一定要保持链接中的 `{{variables}}` 完好无损，这样才能确保有效），然后
3. 重新启动 Vaultwarden 以加载新的（覆盖）模板。

**注意**：为确保兼容性，您应该首先下载适合您的版本的模板，并且如果它们发生变化（或添加了新的模板），您还必须自行更新它们。

## 翻译 <a href="#translations" id="translations"></a>

* 德语 by @kennymc-c：[https://github.com/kennymc-c/vaultwarden-lang-de](https://github.com/kennymc-c/vaultwarden-lang-de)
* 法语 by @YoanSimco：[https://github.com/YoanSimco/vaultwarden-lang-fr](https://github.com/YoanSimco/vaultwarden-lang-fr)
* 波兰语 by @olokelo：[https://github.com/olokelo/vaultwarden-lang-pl](https://github.com/olokelo/vaultwarden-lang-pl)
* 简体中文 by @wcjxixi：[https://github.com/wcjxixi/vaultwarden-lang-zhcn](https://github.com/wcjxixi/vaultwarden-lang-zhcn)
* 简体中文 by @zituoguan：[https://github.com/zituoguan/vaultwarden-lang-zh\_CN](https://github.com/zituoguan/vaultwarden-lang-zh_CN)
* 简体中文 by @JinkaiNiu: [https://github.com/JinkaiNiu/vaultwarden-zh-cn](https://github.com/JinkaiNiu/vaultwarden-zh-cn)
* 意大利语 by @rizlas：[https://github.com/rizlas/vaultwarden-lang-it](https://github.com/rizlas/vaultwarden-lang-it)
* 西班牙语 by @javier-varez：[https://github.com/javier-varez/vaultwarden-lang-es](https://github.com/javier-varez/vaultwarden-lang-es)
* 俄语 by @marat2509：[https://github.com/marat2509/vaultwarden-lang-ru](https://github.com/marat2509/vaultwarden-lang-ru)
* 巴西葡萄牙语 by @marivaldojr：[https://github.com/marivaldojr/vaultwarden-lang-pt\_br](https://github.com/marivaldojr/vaultwarden-lang-pt_br)

{% hint style="danger" %}
翻译由社区成员按原样提供，我们尚未对其进行测试。因此，使用它们需要您自担风险。如果发生重大的变更（例如，由 Vaultwarden 的新版本引起），请通知维护者和/或在此处做一个注释。
{% endhint %}

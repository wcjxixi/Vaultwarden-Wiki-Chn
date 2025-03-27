# 1.翻译管理页面

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Translating-admin-page)
{% endhint %}

由于 Vaultwarden 管理页面独立于官方的 Bitwarden 自托管[系统管理员门户](https://help.ppgg.in/self-hosting/system-administrator-portal)，因此目前只有英文版，并且目前不太可能在同一服务器上为不同用户提供多种语言。

如果您或您的用户不懂英语，或者您希望显示自己喜欢的语言，您可以将管理页面翻译成您的首选语言。

## 如何翻译/定制管理页面？ <a href="#how-to-translate-customize-the-admin-page" id="how-to-translate-customize-the-admin-page"></a>

您可以通过如下方式使用自定义模板：

1. 将相应文件从存储库（`src/static/templates/admin`）复制到具有相同文件夹结构的对应的 `TEMPLATES_FOLDER` 下（例如复制到 `data/templates/admin`）
2. 翻译它们，然后
3. 重新启动 Vaultwarden 以加载新的（覆盖）模板。

**注意**：为确保兼容性，您应该首先下载适合您的版本的模板，并且如果它们发生变化（或添加了新的模板），您还必须自行更新它们。

## 翻译 <a href="#translations" id="translations"></a>

* 简体中文 by @wcjxixi：[https://github.com/wcjxixi/vaultwarden-lang-zhcn](https://github.com/wcjxixi/vaultwarden-lang-zhcn)
* 简体中文 by @zituoguan：[https://github.com/zituoguan/vaultwarden-lang-zh\_CN](https://github.com/zituoguan/vaultwarden-lang-zh_CN)
* 简体中文 by @JinkaiNiu: [https://github.com/JinkaiNiu/vaultwarden-zh-cn](https://github.com/JinkaiNiu/vaultwarden-zh-cn)
* 俄语 by @marat2509：[https://github.com/marat2509/vaultwarden-lang-ru](https://github.com/marat2509/vaultwarden-lang-ru)

{% hint style="danger" %}
翻译由社区成员按原样提供，我们尚未对其进行测试。因此，使用它们需要您自担风险。如果发生重大的变更（例如，由 Vaultwarden 的新版本引起），请通知维护者和/或在此处做一个注释。
{% endhint %}

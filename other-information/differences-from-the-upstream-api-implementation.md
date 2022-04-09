# 3.与上游 API 实现的区别

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Differences-from-the-upstream-API-implementation)
{% endhint %}

## 邀请用户加入组织 <a href="#inviting-users-into-organization" id="inviting-users-into-organization"></a>

### 启用 S​​MTP 时 <a href="#with-smtp-enabled" id="with-smtp-enabled"></a>

被邀请的用户将收到一封包含有效期为 5 天的链接的电子邮件。单击链接后，用户可以选择创建一个帐户或登录。新​​用户需要创建一个新帐户；被邀请新加入组织的现有用户只需登录即可。之后，他们将在管理界面中显示为「已接受」，在组织管理员确认后他们将被添加到组织中。

### 未启用 SMTP 时 <a href="#without-smtp-enabled" id="without-smtp-enabled"></a>

被邀请的用户将不会收到邀请电子邮件；而是所有已经注册的用户都将显示在界面中，就像他们已经接受邀请一样。然后，组织管理员只需确认他们成为组织的成员，并授予他们访问共享密码的权限即可。

尚未注册的受邀用户将在组织管理界面中显示为「受邀」，同时创建邀请「记录」，以允许用户注册，即使[用户注册被禁用](../configuration/disable-registration-of-new-users.md)。（除非[禁用邀请功能](../configuration/disable-invitations.md)，否则）它们一旦注册将自动变为「已接受」。然后组织管理员可以确认他们以授予他们访问组织的权限。

## 在未加密的连接上运行 <a href="#running-on-unencrypted-connection" id="running-on-unencrypted-connection"></a>

强烈建议通过 HTTPS 运行 Vaultwarden 服务。但是，服务器本身在[支持上](../deployment/https/enabling-https.md)并不严格要求进行此类设置。如果您使用信任的连接（比如内部的安全网络、通过 VPN 访问等），或者想要将该服务置于 HTTP 代理之后，从而在代理端进行加密。这些情况下启动服务会更加简单和容易。

通过 HTTP 运行仍然是相当安全的，前提是您使用了非常强大的主密码，并且避免使用易受 MITM 攻击的基于网页密码库的连接，攻击者可能会在该接口中注入 javascript。但是，某些形式的两步登录可能无法在此设置中使用，并且[在此配置下的密码库无法在 Chrome 浏览器中使用](https://github.com/bitwarden/web/issues/254)。

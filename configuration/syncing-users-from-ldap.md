# 18.从 LDAP 同步用户

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Syncing-users-from-LDAP)
{% endhint %}

LDAP 集成使用一个小型服务来执行，该小型服务用于查询 LDAP 并邀请用户加入您的 Vaultwarden 实例。该服务的名称被非正式地命名为 [bitwarden\_rs\_ldap](https://github.com/ViViDboarder/bitwarden\_rs\_ldap)。

由于 Vaultwarden 的零信任架构，此服务不提供密码同步，只提供对新 LDAP 成员的邀请。

它尚未以二进制形式分发，但是有可用的 Docker 镜像 [vividboarder/vaultwarden\_ldap](https://hub.docker.com/r/vividboarder/vaultwarden\_ldap)。

部署之前，您必须[启用 Vaultwarden 管理页面](enabling-admin-page.md)，这将启用 API，以便 LDAP 同步服务使用 API 来邀请用户。配置 LDAP 同步服务时，将使用您设置的 `ADMIN_TOKEN`。您还必须确保**未禁用**邀请功能。请再次检查环境变量 `INVITATIONS_ALLOWED` 未被设置为 `false`。

另外也建议在您的 Vaultwarden 实例中[启用电子邮件发送功能](smtp-configuration.md)，以便通知您的用户注册他们的账户。如果不这样做，虽然他们也可以使用他们的 LDAP 电子邮件地址注册，但是您必须自己通知他们。

完成这些步骤后，您就可以配置和部署 LDAP 同步服务了。最新的说明文档可在它的[自述文件](https://github.com/ViViDboarder/vaultwarden\_ldap)中找到，此文档也涉及了使用 Vaultwarden 实例、LDAP 实例以及您用于查找用户的 LDAP 查询等连接信息来创建 `config.toml` 文件的说明。

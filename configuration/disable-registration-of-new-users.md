# 2.禁用新用户注册

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Disable-registration-of-new-users)
{% endhint %}

默认情况下，可以访问实例的任何人均可以注册新的账户。要禁用该功能，请将 `SIGNUPS_ALLOWED` 环境变量设置为 `false`：

```docker
docker run -d --name vaultwarden \
  -e SIGNUPS_ALLOWED=false \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

## 禁用组织邀请 <a href="#disabling-organization-invitations" id="disabling-organization-invitations"></a>

即使 `SIGNUPS_ALLOWED=false`，作为组织的所有者或管理员的现有用户仍然可以邀请新用户。如果您也想禁用此功能，请参阅[禁用邀请](disable-invitations.md)。

## 将注册限制为某些电子邮件域名 <a href="#restricting-registrations-to-certain-email-domains" id="restricting-registrations-to-certain-email-domains"></a>

您可以通过设置 `SIGNUPS_DOMAINS_WHITELIST` 来限制只能某些域名的电子邮件地址可以注册。示例：

* `SIGNUPS_DOMAINS_WHITELIST=example.com` （单个域名）
* `SIGNUPS_DOMAINS_WHITELIST=example.com,example.net,example.org` （多个域名）

{% hint style="warning" %}
如果设置了 `SIGNUPS_DOMAINS_WHITELIST`，`SIGNUPS_ALLOWED=false`的值将被忽略。
{% endhint %}

你可能还想设置 `SIGNUPS_VERIFY=true`，这要求新注册的用户在成功登录前进行电子邮件验证。这可以防止有人用一个拥有正确域名的假电子邮件地址注册。

## 通过管理页面发出邀请 <a href="#invitations-via-the-admin-page" id="invitations-via-the-admin-page"></a>

Vaultwarden 管理员可以通过[管理页面](enabling-admin-page.md)邀请任何人，不受以上限制。

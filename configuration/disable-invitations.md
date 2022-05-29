# 3.禁用邀请

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Disable-invitations)
{% endhint %}

即使禁用了注册功能，组织管理员或所有者也可以邀请用户加入组织。受邀请后，他们也可以使用受邀请的电子邮件来注册，即使 `SIGNUPS_ALLOWED` 已设置为 `false`。您可以通过将 `INVITATIONS_ALLOWED` 环境变量设置为 `false` 来完全禁用此功能：

```shell
docker run -d --name vaultwarden \
  -e SIGNUPS_ALLOWED=false \
  -e INVITATIONS_ALLOWED=false \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

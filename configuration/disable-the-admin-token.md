# 5.禁用管理令牌

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Disable-admin-token)
{% endhint %}

**重要提示**：任何人都将可以访问管理页面。

如果您有其他方式对 `/admin` 页面进行身份验证，则可以将 `DISABLE_ADMIN_TOKEN` 变量设置为 `true`。这将禁用内置的 `ADMIN_TOKEN` 身份验证功能，同时启用管理面板。有权访问此 URL 的任何人都可以访问管理面板。您需要采取额外的步骤（包括外部和本地）来对它提供保护。

```docker
docker run -d --name vaultwarden \
  -e DISABLE_ADMIN_TOKEN=true \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

# 13.显示密码提示

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Password-hint-display)
{% endhint %}

> \[**译者注**]：此配置默认为 `true`，但当您未配置 SMTP 时才会起作用。输入您的电子邮件后，右上角将显示一条错误信息，其内容即为您的密码提示信息。

通常，密码提示是通过电子邮件发送的。但是，由于 Vaultwarden 是为小型或个人部署而设计的，所以密码提示在密码提示页面上也是可用的，因此您不需要非得配置电子邮件服务。如果要禁用此功能，可以使用 `SHOW_PASSWORD_HINT` 变量：

```docker
docker run -d --name vaultwarden \
  -e SHOW_PASSWORD_HINT=false \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

# 8.启用 YubiKey OTP 身份验证

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-Yubikey-OTP-authentication)
{% endhint %}

要启用 YubiKey 身份验证，必须设置 `YUBICO_CLIENT_ID` 和 `YUBICO_SECRET_KEY` 变量。

如果 `YUBICO_SERVER` 未指定，它将使用默认的 YubiCloud 服务器。您可以在[这里](https://upgrade.yubico.com/getapikey/)使用默认的 YubiCloud 生成 `YUBICO_CLIENT_ID` 和 `YUBICO_SECRET_KEY`。

备注：

* 为了生成 API 密钥或在 OTP 服务器上使用 YubiKey，必须对其进行注册。在 [YubiKey 个性化工具](https://www.yubico.com/products/services-software/personalization-tools/use/)（或 [YubiKey Manager](https://www.yubico.com/support/download/yubikey-manager/)）中配置好你的密钥后，然后在[这里](https://upload.yubico.com)使用默认服务器注册。
* 由于上游的问题，服务器版本为 1.6.0 或更低的 aarch64 不支持 YubiKey 功能（请参阅 [＃262](https://github.com/dani-garcia/bitwarden\_rs/issues/262)）。

```python
docker run -d --name vaultwarden \
  -e YUBICO_CLIENT_ID=12345 \
  -e YUBICO_SECRET_KEY=ABCDEABCDEABCDEABCDE \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

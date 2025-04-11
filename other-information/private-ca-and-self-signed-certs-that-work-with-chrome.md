# \*使私有 CA 和自签名证书兼容 Chrome

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Private-CA-and-self-signed-certs-that-work-with-Chrome)
{% endhint %}

{% hint style="danger" %}
⚠️ 💩 ⚠️此方法仅用于测试和开发。绝大多数用户不应该使用这种方法，因为它需要在您的每台设备上加载证书，这既容易出错，又需要后期的维护。相反，应把精力集中在通过 [Let's Encrypt](https://letsencrypt.org/getting-started/) 获取的真实证书上。如果您的 Vaultwarden 实例不处于公共互联网中，此方法甚至也可以工作（[示例](../deployment/https/running-a-private-vaultwarden-instance-with-lets-encrypt-certs.md)）。⚠️ 💩 ⚠️
{% endhint %}

{% hint style="danger" %}
☠️ ☠️ ☠️ **此方法不受支持。请不要开启 GitHub 话题，也不要在讨论区发帖询问如何让这个方法能正常工作。**☠️ ☠️ ☠️
{% endhint %}

***

为了使 Vaultwarden 能够正常地使用自签名证书，Chrome 要求该证书在证书的备用名称字段中包含域名。

创建 CA 密钥（您自己的小型本地证书颁发机构）：

```sh
openssl genpkey -algorithm RSA -aes128 -out private-ca.key -outform PEM -pkeyopt rsa_keygen_bits:2048
```

> 您也可以使用较旧的 `-des3` 来代替 `-aes128`。

创建 CA 证书：

```sh
openssl req -x509 -new -nodes -sha256 -days 3650 -key private-ca.key -out self-signed-ca-cert.crt
```

> `-nodes` 参数用于阻止在测试/安全环境中为私钥（密钥对）设置密码短语，否则每次启动/重启服务器时都必须输入密码短语。

创建一个 Vaultwarden 密钥：

```sh
openssl genpkey -algorithm RSA -out vaultwarden.key -outform PEM -pkeyopt rsa_keygen_bits:2048
```

创建 Vaultwarden 证书请求文件：

```sh
openssl req -new -key vaultwarden.key -out vaultwarden.csr
```

使用以下内容创建文本文件 `vaultwarden.ext`，请将域名更改为您设置的域名：

```systemd
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:TRUE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = vaultwarden.local
DNS.2 = www.vaultwarden.local
# 如果不使用 DNS 名称，也可选择添加 IP
IP.1 = 192.168.1.3
```

创建从根 CA 签名的 Vaultwarden 证书：

```sh
openssl x509 -req -in vaultwarden.csr -CA self-signed-ca-cert.crt -CAkey private-ca.key -CAcreateserial -out vaultwarden.crt -days 365 -sha256 -extfile vaultwarden.ext
```

> **注意**：自 2019 年 4 月起，iOS 13+ 和 macOS 15+ 的服务器证书的有效期不能大于 825 天，并且必须包含 ExtendedKeyUsage (EKU) 扩展。详见 [https://support.apple.com/zh-cn/HT210176](https://support.apple.com/zh-cn/HT210176)。

> **注意**：从 Android 11 开始，`basicConstraints` 值必须设置为 `CA:TRUE` 才能通过 「设置」 应用程序导入。

将根证书和 Vaultwarden 证书添加到客户端计算机。

更多参考，请参阅这里：[https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/](https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/)。

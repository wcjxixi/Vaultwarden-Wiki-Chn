# 11.更改 API 请求大小限制

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Changing-the-API-request-size-limit)
{% endhint %}

默认情况下，API 调用被限制为 10MB。在大多数情况下，这应该足够了，但是，如果要支持大量访问，则可能会有影响。另一方面，您可能希望将请求大小限制为更小，以防止 API 滥用和可能的 DOS 攻击，尤其是在资源有限的情况下。

要设置限制，可以使用 `ROCKET_LIMITS` 变量。此处的示例设置限制为 10MB（这是默认设置）：

```shell
docker run -d --name vaultwarden \
  -e ROCKET_LIMITS={json=10485760} \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

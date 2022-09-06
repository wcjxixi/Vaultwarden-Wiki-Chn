# 4.启用管理页面

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-admin-page)
{% endhint %}

**重要说明**：强烈建议在启用此功能之前激活 HTTPS，以避免潜在的 [MITM](https://zh.wikipedia.org/wiki/%E4%B8%AD%E9%97%B4%E4%BA%BA%E6%94%BB%E5%87%BB) 攻击。

该页面允许服务器管理员查看并删除所有已注册的用户。它也允许邀请新用户，即使禁用了注册功能。

要启用管理页面，您需要设置一组身份验证令牌。该令牌可以是任何字符，但建议使用随机生成的长字符串，比如运行 `openssl rand -base64 48` 命令生成的。**此令牌是您访问服务器管理区域的密码！将其保存在安全的地方。**

要设置令牌，请使用 `ADMIN_TOKEN` 变量：

```shell
docker run -d --name vaultwarden \
  -e ADMIN_TOKEN=some_random_token_as_per_above_explanation \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

此后，管理页面将在 `/admin` 子目录中可用。

在管理页面首次保存设置时，将自动在 `DATA_FOLDER` 文件夹中生成 `config.json` 文件。该文件中的值优先于环境变量值。

需要注意的是，在您点击 `Save` 按钮之前，管理页面中的配置更改是不会生效的。例如，如果您正在测试 SMTP 设置，您更改了 `SMTP Auth mechanism` 设置，然后点击 `Send test email` 来测试更改，这将不会像预期的那样工作 -- 因为您没有点击 `Save`，`SMTP Auth mechanism` 的更改不会生效。

**注意**：更改 `ADMIN_TOKEN` 后，当前已登录管理页面的人仍可以使用旧的登录令牌[长达 20 分钟时间](https://github.com/dani-garcia/vaultwarden/blob/main/src/api/admin.rs#L87)。

**注意**：如果环境变量 `ADMIN_TOKEN` 的值持续保留在上述的 `config.json` 文件中，则移除环境变量 `ADMIN_TOKEN` 并不会禁用管理页面。**要禁用管理页面**，请确保没有设置 `ADMIN_TOKEN` 环境变量，并且 `config.json`（如果该文件存在）中不存在 `"admin_token"` 键。

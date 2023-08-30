# 7.启用移动客户端推送通知

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-Mobile-Client-push-notification)
{% endhint %}

从 Vaultwarden 1.29.0 版本开始，您可以激活移动客户端的推送通知，以在移动应用程序、网页扩展程序和网页密码库之间[自动同步](https://help.ppgg.in/password-manager/vault-administration/syncing-your-vault#automatic-sync)您的个人密码库，而无需手动同步。

### 启用移动客户端推送通知 <a href="#enable-mobile-client-push-notification" id="enable-mobile-client-push-notification"></a>

1、访问 [https://bitwarden.com/host/](https://bitwarden.com/host/)，输入您的电子邮件地址，然后您将获得一个 INSTALLATION ID 和 KEY。

{% hint style="info" %}
在 [#3752](https://github.com/dani-garcia/vaultwarden/pull/3752) 实施之前，请确保选择 `bitwarden.com（美国）`作为数据区域。
{% endhint %}

{% hint style="warning" %}
&#x20;Vaultwarden 目前**不支持**欧盟数据区域。如果您已请求 `bitwarden.eu（欧盟）`的 INSTALLATION ID 和 KEY，您需要等到 PR 合并和发布后，自己[构建 Vaultwarden](../deployment/building-your-own-docker-image.md) 并进行必要的更改，或者您可以简单地请求一个用于美国数据区域的新的 ID/KEY 对。
{% endhint %}

2、将以下设置添加到 `docker-compose.yml`（并确保插入上一步获取到的正确 ID 和 KEY）：

```
      - PUSH_ENABLED=true
      - PUSH_INSTALLATION_ID=
      - PUSH_INSTALLATION_KEY=
```

3、重新创建您的容器，例如：

```
docker compose up -d vaultwarden
```

4、将您的应用程序连接到您的 Vaultwarden 实例。

{% hint style="warning" %}
除非您使用的最新安装的 Bitwarden 应用程序，否则已连接的客户端上的推送通知**将不起作用**。您必须**清除应用程序数据**（或**重新安装应用程序**）然后再次连接您的 Vaultwarden 账户，才能[向 Bitwarden 的 Azure 通知中心注册推送令牌](https://contributing.bitwarden.com/architecture/deep-dives/push-notifications/mobile/#self-hosted-implementation)。
{% endhint %}

{% hint style="warning" %}
推送通知仅适用于从官方移动商店（App Store、Google Play 商店）或使用 Google Play 商店的替代客户端（如 Aurora 商店）安装的 Bitwarden 应用程序。从 [F-Droid](https://mobileapp.bitwarden.com/fdroid/)、NeoStore 或其他替代商店安装的 Bitwarden，推送通知将不起作用。因为这些应用程序是在不支持 Firebase Messaging 的情况下构建的。
{% endhint %}

5、测试移动端的推送通知是否正常工作，例如通过重命名网页密码库中的文件夹，然后查看移动应用程序中的文件夹在几秒钟后是否发生变化。

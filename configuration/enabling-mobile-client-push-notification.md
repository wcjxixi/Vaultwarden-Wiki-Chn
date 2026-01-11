# 7.启用移动客户端推送通知

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-Mobile-Client-push-notification)
{% endhint %}

从 Vaultwarden 1.29.0 版本开始，您可以激活移动客户端的推送通知，以在移动应用程序、网页扩展程序和网页密码库之间[自动同步](https://help.ppgg.in/password-manager/vault-administration/syncing-your-vault#automatic-sync)您的个人密码库，而无需手动同步。

### 启用移动客户端推送通知 <a href="#enable-mobile-client-push-notification" id="enable-mobile-client-push-notification"></a>

1、访问 [https://bitwarden.com/host/](https://bitwarden.com/host/)，输入您的电子邮箱地址，然后您将获得一个 INSTALLATION ID 和 KEY。

{% hint style="info" %}
~~在~~ [~~#3752~~](https://github.com/dani-garcia/vaultwarden/pull/3752) ~~实施之前，请确保选择 `bitwarden.com（美国）`作为数据区域。~~
{% endhint %}

{% hint style="warning" %}
~~Vaultwarden 目前**不支持**欧盟数据区域。如果您已请求 `bitwarden.eu（欧盟）`的 INSTALLATION ID 和 KEY，您需要等到 PR 合并和发布后，自己~~[~~构建 Vaultwarden~~](../deployment/building-your-own-docker-image.md) ~~并进行必要的更改，或者您可以简单地请求一个用于美国数据区域的新的 ID/KEY 对。~~
{% endhint %}

2、将以下设置添加到 `docker-compose.yml`（并确保插入上一步获取到的正确 ID 和 KEY）：

```yaml
    environment:
      - PUSH_ENABLED=true
      - PUSH_INSTALLATION_ID=
      - PUSH_INSTALLATION_KEY=
```

{% hint style="info" %}
如果您在上一步中请求了 `bitwarden.eu（欧盟）` 的安装 ID 和 kEY，您还必须设置

```yaml
      - PUSH_RELAY_URI=https://api.bitwarden.eu
      - PUSH_IDENTITY_URI=https://identity.bitwarden.eu
```
{% endhint %}

3、重新创建您的容器，例如：

```sh
docker compose up -d vaultwarden
```

4、注销然后重新登录 Bitwarden 客户端，这样它们就能从服务器上获取推送配置。

{% hint style="danger" %}
如果您在 [v1.30.2](https://github.com/dani-garcia/vaultwarden/releases/tag/1.30.2) 之前已经连接了 Bitwarden 应用程序，则推送通知**将不适用于**您的设备（因为设备令牌从未保存）。您必须**清除应用程序数据**（或**重新安装应用程序**）然后再次连接您的 Vaultwarden 账户，才能向 [Bitwarden 的 Azure 通知中心](https://contributing.bitwarden.com/architecture/deep-dives/push-notifications/mobile/#self-hosted-implementation)注册推送令牌。
{% endhint %}

{% hint style="warning" %}
推送通知**仅适用于**从官方移动商店（App Store、Google Play 商店）或使用 Google Play 商店的替代客户端（如 Aurora 商店）获取的 Bitwarden 应用程序。从 [F-Droid](https://mobileapp.bitwarden.com/fdroid/)、NeoStore 或其他替代商店安装的 Bitwarden，推送通知**将不起作用**。因为这些应用程序是在不支持 Firebase Messaging 的情况下构建的。为保证推送通知正常，请确保 `firebaseinstallations.googleapis.com` 未被阻止，因为该功能需要它才能正常工作。
{% endhint %}

5、测试移动端的推送通知是否正常工作，例如通过重命名网页密码库中的文件夹，然后查看移动应用程序中的文件夹在几秒钟后是否发生变化。

## 从美国服务器切换到欧盟服务器（反之亦然） <a href="#switching-from-us-to-eu-servers-or-vice-versa" id="switching-from-us-to-eu-servers-or-vice-versa"></a>

{% hint style="danger" %}
在进行此更改之前，请确保您使用的是最新版本 [![GitHub Release](https://img.shields.io/github/release/dani-garcia/vaultwarden.svg)](https://github.com/dani-garcia/vaultwarden/releases/latest)。
{% endhint %}

要从一个数据区域切换到另一数据区域，您必须：

1. 取消所有会话授权并清除移动应用程序上的应用程序数据
2. 使用不同的数据区域重复上一节中的步骤 1 到步骤 5

除了步骤 1 之外，您还可以清除数据库中 `devices` 表的 `push_uuid` 字段，例如：

```sql
UPDATE devices SET push_uuid = NULL;
```

这将触发在您下次登录该设备时重新注册您的推送设备。

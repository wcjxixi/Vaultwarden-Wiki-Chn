# 1.Bitwarden Android 故障排除

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Bitwarden-Android-troubleshooting)
{% endhint %}

## 通用 <a href="#general" id="general"></a>

自从新的 Bitwarden 移动客户端发布以后，一些问题不断出现。这些问题主要是因为新客户端对服务器返回的 JSON 更加严格。

由于旧客户端不太严格，以及 Vaultwarden 没有（现在仍然没有完全）对所有客户端的输入进行筛选或纠正，以便在 Bitwarden 添加新功能时保持尽可能的灵活。有时存储的数据可能包含无效值。我们会尝试在同步过程中纠正所有这些项目，以尽可能修复所有旧的无效值，并相信客户端会发送正确的新数据。

由于我们无法知道任何客户端生成的所有无效数据，因此开发人员有时很难找出具体的问题所在。为此，我们需要社区的帮助来排查问题并找出罪魁祸首。

以下是一些故障排除指南，可以帮助您找到问题所在，从而自行修复，或帮助开发人员在服务器端修复。

## 首先尝试的事情 <a href="#things-to-try-first" id="things-to-try-first"></a>

首先要确保客户端本地没有无效的缓存 JSON。这是在同步请求完成之前加载的，可能会导致我们无法在服务器端解决的问题。

1. 从移动客户端注销
2. 清除 Bitwarden App 的缓存和数据
3. 卸载 Bitwarden App
4. 为了确保彻底清除，请重启设备
5. 重新安装 Bitwarden App
6. 配置 App 以连接到您的自托管实例
7. 登录并祈祷

如果上述步骤没有解决您的问题，那么可能 Vaultwarden 返回的数据还是存在问题。这种情况下，请继续以下步骤。

## 安装 Android 调试工具 (adb) <a href="#install-android-debugging-tools-adb" id="install-android-debugging-tools-adb"></a>

大多数（并非全部）的 Android 设备都可以通过将设备连接到装有合适工具的计算机来提取设备日志。您可以在以下链接中找到各个平台的详细指南以了解如何安装这些工具：[https://www.xda-developers.com/install-adb-windows-macos-linux/](https://www.xda-developers.com/install-adb-windows-macos-linux/)。

**在实际安装工具之前，请先阅读与您的平台相关的所有内容，有时会有多种方法介绍，而第一种方法可能不是最简单的。**

安装这些工具并能连接到您的手机后，继续下一步。

## 实际的调试 <a href="#the-actual-debugging" id="the-actual-debugging"></a>

现在一切设置完成，我们开始提取日志，希望这些日志能够帮助追踪问题。

运行以下命令以仅显示 Bitwarden 客户端日志：

```sh
adb logcat --pid=$(adb shell pidof -s com.x8bit.bitwarden)
```

{% hint style="info" %}
您可以通过按 ctrl+c（或 cmd+c）来退出 logcat。
{% endhint %}

{% hint style="success" %}
如果您想在文件中记录所有内容，您至少可以在 Linux 使用如下的附加命令：

```sh
# 直接输出到文件：
> bitwarden-android.log
# 或者，输出到文件并同时输出到屏幕上：
> | tee -a bitwarden-android.log
# 完整的示例：
adb logcat --pid=$(adb shell pidof -s com.x8bit.bitwarden) > bitwarden-android.log
adb logcat --pid=$(adb shell pidof -s com.x8bit.bitwarden) | tee -a bitwarden-android.log
```
{% endhint %}

这应该会开始显示一些日志，如果是，请继续，如果没有，请检查错误信息并尝试解决问题。

现在，随着屏幕上的日志输出，请尝试触发客户端中的错误并检查输出。开发人员需要这些输出来找出问题所在。

### 获取更多详细信息 <a href="#getting-more-details" id="getting-more-details"></a>

如果没有有用的输出，可以通过使用 Bitwarden Dev/Debug Android 客户端来获取更多详细信息。

这些版本是通过 GitHub Actions 构建的，可以在以下链接找到：[https://github.com/bitwarden/android/actions/workflows/build.yml?query=is%3Asuccess](https://github.com/bitwarden/android/actions/workflows/build.yml?query=is%3Asuccess)

基本上，任何成功的构建都应包含一个名为 `com.x8bit.bitwarden.dev.apk` 的工件文件。下载此文件，它是一个 zip 文件，解压该 zip 文件，然后安装解压后的 apk 文件。如果您的 Android 设备上有支持 zip 文件的文件管理器，您可以直接在设备上完成此操作。

或者，使用 `adb` 按如下方式安装：

```sh
adb install com.x8bit.bitwarden.dev.apk
```

完成此操作后，您就安装了一个额外的 Bitwarden 客户端，该客户端会输出更详细的日志。

按照您通常登录自托管实例的步骤进行操作。

要从该客户端提取日志，您需要稍微调整一下 `logcat` 命令，使其看起来像这样：

```sh
adb logcat --pid=$(adb shell pidof -s com.x8bit.bitwarden.dev)
```

这会提供更多详细信息，对于追踪问题将非常有帮助。

{% hint style="danger" %}
**dev 客户端的输出包含由 Vaultwarden 服务器发送的响应，可能包含敏感数据！**

虽然大多数项目都进行了加密，但一些项目如电子邮箱地址或您的 Vaultwarden 域名则没有加密！

**请谨慎分享此输出！**

虽然完整的输出对我们开发者非常有用，有助于故障排除，以及我们无法解密数据，但仍需小心！
{% endhint %}

## 分享结果 <a href="#sharing-the-results" id="sharing-the-results"></a>

我们建议使用以下安全的方式之一分享这些文件：

1. （ _**推荐**_ ）通过我们的 Matrix 聊天室：[ ![Matrix Chat](https://camo.githubusercontent.com/9394bf2c04102038a60688af025f04eab4cf6306f4241ac21a8f42927d403401/68747470733a2f2f696d672e736869656c64732e696f2f6d61747269782f7661756c7477617264656e3a6d61747269782e6f72672e7376673f7374796c653d666c61742d737175617265266c6f676f3d6d6174726978266c6f676f436f6c6f723d66666626636f6c6f723d3935334230302663616368655365636f6e64733d3134343030)](https://matrix.to/#/#vaultwarden:matrix.org)
2. 通过电子邮件，发送到 [![security-contact](https://github.com/dani-garcia/vaultwarden/raw/refs/heads/main/.github/security-contact.gif)](https://github.com/dani-garcia/vaultwarden/raw/refs/heads/main/.github/security-contact.gif)，以提供详细信息
3. 通过您自托管的 Vaultwarden 的带有密码的 **Send**（使用 Matrix 或电子邮件分享）

如果您对此主题有任何疑问，请在我们的 [![GitHub Discussions](https://camo.githubusercontent.com/e318009d35d82b3f9e13b41af9e84f8ee9ade6b9d5d4311c1dcb2364de6cfe09/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f64697363757373696f6e732f64616e692d6761726369612f7661756c7477617264656e3f7374796c653d666c61742d737175617265266c6f676f3d676974687562266c6f676f436f6c6f723d66666626636f6c6f723d3935334230302663616368655365636f6e64733d333030) ](https://github.com/dani-garcia/vaultwarden/discussions)上发起一个新的主题。

# 4.第三方软件包

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Third-party-packages)
{% endhint %}

{% hint style="danger" %}
本页面是一个第三方 Vaultwarden 软件包的索引。

由于这些软件包不是由 Vaultwarden 维护或控制的，因此它们可能会比官方的发行版本滞后，有时甚至会明显滞后。如果你依赖这些软件包，你可能需要为新的 Vaultwarden 发行版本[启用监视](https://docs.github.com/en/github/managing-subscriptions-and-notifications-on-github/viewing-your-subscriptions#configuring-your-watch-settings-for-an-individual-repository)功能，并让维护者知道该软件包未保持更新。
{% endhint %}

## 非官方软件包状态

Server：[https://repology.org/project/vaultwarden/versions](https://repology.org/project/vaultwarden/versions)

Web-Vault：[https://repology.org/project/vaultwarden-web/versions](https://repology.org/project/vaultwarden-web/versions)

## Arch Linux

可在[官方仓库](https://archlinux.org/packages/community/x86\_64/vaultwarden/)中获取，同时包含了[网页版密码库](https://archlinux.org/packages/community/any/vaultwarden-web/)。

## Debian

基于工具链的 docker 可用于构建 debian 软件包：[https://github.com/greizgh/bitwarden\_rs-debian](https://github.com/greizgh/bitwarden\_rs-debian)。它将服务器和网页版密码库捆绑在一起。

## CentOS7 / RHEL7

RPM 库由 @MrMEEE 打包在这里：[https://copr.fedorainfracloud.org/coprs/mrmeee/bitwarden\_rs/](https://copr.fedorainfracloud.org/coprs/mrmeee/bitwarden\_rs/)...在附加包里同样包含有网页界面。

安装说明：[https://github.com/MrMEEE/bitwarden\_rs\_rpm/blob/master/README.md](https://github.com/MrMEEE/bitwarden\_rs\_rpm/blob/master/README.md)

任何 RPM 话题可以提交到这里：[https://github.com/MrMEEE/bitwarden\_rs\_rpm/issues](https://github.com/MrMEEE/bitwarden\_rs\_rpm/issues)

## CentOS 8 / RHEL 8

克隆的一个分支，它构建一个 RPM 并使用 Docker 将其推送到 COPR。

[https://github.com/alexpdp7/bitwarden\_rs/tree/rpm/packages/centos8](https://github.com/alexpdp7/bitwarden\_rs/tree/rpm/packages/centos8) [https://copr.fedorainfracloud.org/coprs/koalillo/bitwarden\_rs/](https://copr.fedorainfracloud.org/coprs/koalillo/bitwarden\_rs/)

## Nix (OS)

Vaultwarden 被同时打包用于 mysql、sqlite、postgresql，以及用于 Vault。还有一个用于声明式配置的 NixOS 模块（请参阅 `services.vaultwarden`）

## Cloudron

[Cloudron](https://cloudron.io) 是一个帮助你在服务器上运行 Web 应用的平台。使用 Cloudron，你可以从 [App Library](https://cloudron.io/store/com.github.bitwardenrs.html) 中轻松地安装自定义域名的 Bitwarden\_rs。该应用包与上游网页密码库捆绑在一起，安装后不需要任何进一步的配置即可开始使用。Cloudron 团队会保持发行版跟踪并提供自动更新。

软件包代码和话题跟踪器可以在这里找到：[https://git.cloudron.io/cloudron/bitwardenrs-app](https://git.cloudron.io/cloudron/bitwardenrs-app)。

## Home Assistant <a href="#home-assistant" id="home-assistant"></a>

[Home Assistant](https://www.home-assistant.io) 是一个开源的家庭自动化平台。在这里可找到 bitwarden\_rs 社区插件：[https://github.com/hassio-addons/addon-bitwarden](https://github.com/hassio-addons/addon-bitwarden)。

## 用于 Ubuntu 20.04 的编译脚本 <a href="#build-script-for-ubuntu-20-04" id="build-script-for-ubuntu-20-04"></a>

Dinger1986 创建了一个在 Ubuntu 20.04 上从源代码安装 bitwarden\_rs 的脚本，参见：[https://github.com/dinger1986/bitwardenrs\_install\_script](https://github.com/dinger1986/bitwardenrs\_install\_script)

## FreeBSD

在 [FreeBSD 端口树](https://www.freshports.org/security/vaultwarden/)中可用，并在 FreeBSD pkg 存储库中作为二进制包提供：`pkg install vaultwarden`

## 多个 RPM 和 DEB 发行版 <a href="#multiple-rpm-and-deb-distributions" id="multiple-rpm-and-deb-distributions"></a>

openSUSE 构建服务项目，支持 CentOS、Debian、Fedora、RHEL、SUSE、Ubuntu。

您可以直接下载软件包或使用可用的存储库。

**警告**：目前这些软件包包含预构建的二进制文件，无法使用此构建服务构建 rust-nightly 包。

[vaultwarden](https://build.opensuse.org/package/show/home:Masgalor:Vaultwarden/vaultwarden) [vaultwarden-webvault](https://build.opensuse.org/package/show/home:Masgalor:Vaultwarden/vaultwarden-webvault)

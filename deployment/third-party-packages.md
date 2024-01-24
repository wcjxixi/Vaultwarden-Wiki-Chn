# 4.第三方包

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Third-party-packages)
{% endhint %}

{% hint style="danger" %}
本页面是一个第三方 Vaultwarden 包的索引。

由于这些包不是由 Vaultwarden 维护或控制的，因此它们可能会比官方的发行版本滞后，有时甚至会滞后很多。如果您依赖这些包，您可能需要为新的 Vaultwarden 发行版本[启用监视](https://docs.github.com/en/github/managing-subscriptions-and-notifications-on-github/viewing-your-subscriptions#configuring-your-watch-settings-for-an-individual-repository)功能，并告诉维护者该包未保持最新。
{% endhint %}

## 非官方包状态

<table data-full-width="false"><thead><tr><th>Server</th><th>Web-Vault</th></tr></thead><tbody><tr><td><a href="https://repology.org/project/vaultwarden/versions"><img src="https://repology.org/badge/vertical-allrepos/vaultwarden.svg" alt="Packaging status"></a></td><td><a href="https://repology.org/project/vaultwarden-web/versions"><img src="https://repology.org/badge/vertical-allrepos/vaultwarden-web.svg" alt="Packaging status" data-size="original"></a></td></tr></tbody></table>

{% hint style="warning" %}
请注意，最新的 Vaultwarden 版本并不总是与最新的 web-vault 版本前向兼容，因此您可能需要使[用旧版本](https://github.com/dani-garcia/bw\_web\_builds/releases)的 vaultwarden-web 以确保兼容性。
{% endhint %}

## Arch Linux

可在[官方仓库](https://archlinux.org/packages/community/x86\_64/vaultwarden)中获取，同时包含了[网页版密码库](https://archlinux.org/packages/extra/any/vaultwarden-web/)。

## Debian

一个基于 docker 的工具链，可用于构建 debian 包：[https://github.com/greizgh/vaultwarden-debian](https://github.com/greizgh/vaultwarden-debian)。它捆绑了服务器和网页版密码库。

带有纯编译工具链的 Debian 源代码（无 docker）：[https://github.com/dionysius/vaultwarden-deb](https://github.com/dionysius/vaultwarden-deb) 和 [https://github.com/dionysius/vaultwarden-web-vault-deb](https://github.com/dionysius/vaultwarden-web-vault-deb)，它为最新的 Ubuntu LTS 和 Debian 稳定版提供预构建包。

## DietPi（高度优化的最小化 Debian 操作系统） <a href="#dietpi-highly-optimised-minimal-debian-os" id="dietpi-highly-optimised-minimal-debian-os"></a>

[DietPi](https://dietpi.com/) 是一个基于 Debian 的轻量级发行版（镜像），适用于各种设备，如 Raspberry Pi、Odroid、NanoPi 等。它提供了一个用于安装包括 Vaultwarden 在内的各种程序的软件脚本。这使用户无需了解安装命令。

要在 DietPi 上安装 Vaultwarden，只需在命令行中键入 `dietpi-software install 183`。有关安装过程和首次访问 DietPi 上的 Vaultwarden 的更多信息，请访问 [https://dietpi.com/docs/software/cloud/#vaultwarden](https://dietpi.com/docs/software/cloud/#vaultwarden)。

## CentOS 8 / RHEL 8

一个使用 SQLite 的 hacky 包。它还不包含密码库，并且在很明显的地方仍然使用旧的名称。

[https://github.com/alexpdp7/vaultwarden-rpm](https://github.com/alexpdp7/vaultwarden-rpm)

## Fedora (current release, x86\_64)

此 Vaultwarden 包被构建为一个通用二进制文件，其用于 SQLite、MySQL 和 PostgreSQL。它还创建一个 `vaultwarden` 用户/组和一个 systemd 服务。

```batch
dnf config-manager --add-repo https://evermeet.cx/pub/repo/fedora/evermeet.repo
dnf install vaultwarden vaultwarden-webvault
```

## Nix (OS)

此 Vaultwarden 被同时打包用于 mysql、sqlite、postgresql，以及 Vault。还有一个用于声明式配置的 NixOS 模块（请参阅 `services.vaultwarden`）

## Cloudron

[Cloudron](https://cloudron.io/) 是一个帮助您在服务器上运行 Web 应用程序的平台。使用 Cloudron，你可以从 [App Library](https://cloudron.io/store/com.github.bitwardenrs.html) 中轻松地安装自定义域名的 Vaultwarden。该应用包与上游网页密码库捆绑在一起，安装后不需要任何进一步的配置即可开始使用。Cloudron 团队会保持发行版跟踪并提供自动更新。

包代码和话题跟踪器可以在这里找到：[https://git.cloudron.io/cloudron/bitwardenrs-app](https://git.cloudron.io/cloudron/vaultwarden-app)。

## Home Assistant <a href="#home-assistant" id="home-assistant"></a>

[Home Assistant](https://www.home-assistant.io/) 是一个开源的家庭自动化平台。在这里可找到 Vaultwarden 社区插件：[https://github.com/hassio-addons/addon-bitwarden](https://github.com/hassio-addons/addon-bitwarden)。

## 用于 Ubuntu 20.04 的构建脚本 <a href="#build-script-for-ubuntu-20-04" id="build-script-for-ubuntu-20-04"></a>

Dinger1986 创建了一个在 Ubuntu 20.04 上从源代码安装 Vaultwarden 的脚本，参阅：[https://github.com/dinger1986/bitwardenrs\_install\_script](https://github.com/dinger1986/bitwardenrs\_install\_script)

## FreeBSD

在 [FreeBSD 端口树](https://www.freshports.org/security/vaultwarden/)中可用，并在 FreeBSD pkg 仓库中作为二进制包提供：`pkg install vaultwarden`

`/usr/local/etc/rc.conf.d/vaultwarden.sample` 是示例配置文件。将此文件复制到 `/usr/local/etc/rc.conf.d/vaultwarden` 并编辑其内容以[配置 Vaultwarden](../configuration/configuration-overview.md#configuration-options)。然后就可以像平常那样（`service(8)` 等）启动 `vaultwarden` 服务了。

## Syncloud

[Syncloud](https://syncloud.org/) 是一个自托管平台，可以帮助没有设备管理经验的人在他们的设备上运行流行的服务。

Bitwarden 可在设备上的应用商店中安装，并且无需配置。

## 用于最常见发行版的 RPM 和 DEB 包 <a href="#rpm-and-deb-packages-for-most-common-distributions" id="rpm-and-deb-packages-for-most-common-distributions"></a>

openSUSE 构建服务项目，支持：

> \[**译者注**]：[什么是 openSUSE 构建服务](https://zh.wikipedia.org/wiki/Open\_Build\_Service)

| RPM    | 版本                           |
| ------ | ---------------------------- |
| SUSE   | 15.4; 15.5; Tumbleweed       |
| RHEL   | 7\*; 8                       |
| CentOS | 7\*; 8; 8\_Stream; 9\_Stream |
| Fedora | 36; 37; 38; Rawhide          |

_（\* 仅限 vaultwarden-1.28.0，因为 GCC-4.9 不可用。）_

| DEB    | 版本                         |
| ------ | -------------------------- |
| Debian | 10; 11; 12; Testing        |
| Ubuntu | 18.04; 20.04; 22.04; 23.04 |

您可以直接下载包或使用可用的仓库。

[vaultwarden](https://build.opensuse.org/package/show/home:Masgalor:Vaultwarden/vaultwarden)、[vaultwarden-webvault](https://build.opensuse.org/package/show/home:Masgalor:Vaultwarden/vaultwarden-webvault)、[vaultwarden-webvault-dark](https://build.opensuse.org/package/show/home:Masgalor:Vaultwarden/vaultwarden-webvault-dark)

## CentOS 9 / RHEL 9

从 Masgalor 包中为 EL9 构建的包：

[https://rpm.awx.wiki/vaultwarden/](https://rpm.awx.wiki/vaultwarden/)

每晚自动构建新版本。

## Void Linux

在 void-packages 中作为 [vaultwarden](https://github.com/void-linux/void-packages/tree/master/srcpkgs/vaultwarden) 可用：`xbps-install vaultwarden`\
还可以选择安装网页密码库 ([vaultwarden-web](https://github.com/void-linux/void-packages/tree/master/srcpkgs/vaultwarden-web))：`xbps-install vaultwarden-web`

## Snapcraft

通过 [Snap Store](https://snapcraft.io/vaultwarden) 以 [snap](https://github.com/DownThePark/snapcraft-vaultwarden) 形式提供。网页密码库也包含在内，且默认已启用。

可以使用以下命令从命令行安装 Vaultwarden：`snap install vaultwarden`

其配置文件位于：`/var/snap/vaultwarden/current/.env`

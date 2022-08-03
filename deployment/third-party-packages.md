# 4.第三方包

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Third-party-packages)
{% endhint %}

{% hint style="danger" %}
本页面是一个第三方 Vaultwarden 包的索引。

由于这些包不是由 Vaultwarden 维护或控制的，因此它们可能会比官方的发行版本滞后，有时甚至会明显滞后。如果您依赖这些包，您可能需要为新的 Vaultwarden 发行版本[启用监视](https://docs.github.com/en/github/managing-subscriptions-and-notifications-on-github/viewing-your-subscriptions#configuring-your-watch-settings-for-an-individual-repository)功能，并告诉维护者该包未保持最新。
{% endhint %}

## 非官方包状态

| Server                                                                                                                                 | Web-Vault                                                                                                                                      |
| -------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| [![Packaging status](https://repology.org/badge/vertical-allrepos/vaultwarden.svg)](https://repology.org/project/vaultwarden/versions) | [![Packaging status](https://repology.org/badge/vertical-allrepos/vaultwarden-web.svg)](https://repology.org/project/vaultwarden-web/versions) |

## Arch Linux

可在[官方仓库](https://archlinux.org/packages/community/x86\_64/vaultwarden/)中获取，同时包含了[网页版密码库](https://archlinux.org/packages/community/any/vaultwarden-web/)。

## Debian

一个基于 docker 的工具链，可用于构建 debian 包：[https://github.com/greizgh/bitwarden\_rs-debian](https://github.com/greizgh/bitwarden\_rs-debian)。它捆绑了服务器和网页版密码库。

## CentOS7 / RHEL7

由 @MrMEEE 打包的一个 RPM 仓库：[https://copr.fedorainfracloud.org/coprs/mrmeee/bitwarden\_rs/](https://copr.fedorainfracloud.org/coprs/mrmeee/bitwarden\_rs/) ... 在附加包里同样包含有网页界面。

安装说明：[https://github.com/MrMEEE/bitwarden\_rs\_rpm/blob/master/README.md](https://github.com/MrMEEE/bitwarden\_rs\_rpm/blob/master/README.md)

任何 RPM 话题可以提交到这里：[https://github.com/MrMEEE/bitwarden\_rs\_rpm/issues](https://github.com/MrMEEE/bitwarden\_rs\_rpm/issues)

## CentOS 8 / RHEL 8

一个使用 SQLite 的 hacky 包。它还不包含密码库，并且在很明显的地方仍然使用旧的名称。

[https://github.com/alexpdp7/vaultwarden-rpm](https://github.com/alexpdp7/vaultwarden-rpm)

## Nix (OS)

Vaultwarden 被同时打包用于 mysql、sqlite、postgresql，以及用于 Vault。还有一个用于声明式配置的 NixOS 模块（请参阅 `services.vaultwarden`）

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

## 用于最常见发行版的 RPM 和 DEB 包 <a href="#rpm-and-deb-packages-for-most-common-distributions" id="rpm-and-deb-packages-for-most-common-distributions"></a>

openSUSE 构建的服务项目，支持：



| RPM    |                                   |
| ------ | --------------------------------- |
| SUSE   | <p>15.3<br>15.4<br>Tumbleweed</p> |
| RHEL   | 7                                 |
| CentOS | <p>7<br>8<br>8_Stream</p>         |
| Fedora | <p>34<br>35<br>36<br>Rawhide</p>  |

| DEB    |                                |
| ------ | ------------------------------ |
| Debian | <p>10<br>11<br>Testing</p>     |
| Ubuntu | <p>18.04<br>20.04<br>22.04</p> |

您可以直接下载包或使用可用的仓库。

[vaultwarden](https://build.opensuse.org/package/show/home:Masgalor:Vaultwarden/vaultwarden) [vaultwarden-webvault](https://build.opensuse.org/package/show/home:Masgalor:Vaultwarden/vaultwarden-webvault) [vaultwarden-webvault-dark](https://build.opensuse.org/package/show/home:Masgalor:Vaultwarden/vaultwarden-webvault-dark)

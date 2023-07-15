# 1.容器镜像的选择

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Which-container-image-to-use)
{% endhint %}

从版本 1.17.0 开始，`vaultwarden` 提供了一个单一的 Docker 镜像 ([`vaultarden/server`](https://hub.docker.com/r/vaultwarden/server))，该镜像对 SQLite、MySQL 和 PostgreSQL 数据库后端提供统一的支持。在该版本之前，每一种数据库后端都有单独的镜像（请参阅[历史镜像](which-container-image-to-use.md#historical-images)）。

`vaultwarden/server` 是一个多架构镜像，这意味着它在一个镜像名下支持多种 CPU 架构。假设您运行的是支持的架构之一，简单地拉取 `vaultwarden/server` 会自动产生适合您的环境的特定架构的镜像。如果您使用的是 ARMv6 开发板，比如 Raspberry Pi 1 和 Zero，您必须使用 Docker 20.10.0 以及更高版本才能使其正常运行（请参阅 [moby/moby#41017](https://github.com/moby/moby/issues/41017)）。

SQLite 后端是最广泛被使用/测试的后端，除非有特殊需要使用其他数据库后端，否则建议大多数用户使用它。

## 镜像标签 <a href="#image-tags" id="image-tags"></a>

`vaultwarden/server` 镜像有一系列标签，每种标签都代表了镜像的一些变体或属性（例如特定的版本）。

* `latest` -- 跟踪最新发布的版本（即带有版本号的标签）。推荐大多数用户使用这个标签，因为它通常是最稳定的。
* `testing` -- 跟踪源代码库的最新提交的版本。这个标签推荐给想要提前获取最新功能或增强功能的用户。测试版一般都很稳定，但不可避免它偶尔也会出现一些问题。
* `x.y.z` (例如 `1.16.0`) -- 代表一个特定的发布版本。
* `alpine` -- 除了少数例外，该镜像功能上与 `latest` 相同，但它是基于 Alpine 而非 Debian，因此镜像更小。选择 `latest` 或 `alpine` 主要是一个喜好问题，但请注意 `alpine` 标签目前只支持 `amd64` 和 `arm32v7` 架构，并且仅支持 SQLite 和 PostgreSQL 数据库后端。
* `x.y.z-alpine` (例如 `1.16.0-alpine`) -- 与 `alpine` 类似，但它代表一个特定的发布版本。
* ~~`latest-arm32v6` -- 与 `latest` 相同，但明确表示为 `arm32v6` 镜像。目前，对于使用 Armv6 板卡（如 Raspberry Pi 1 和 Zero）的用户来说，需要使用此标签。否则，Docker 会尝试拉取 `arm32v7` 镜像，这将无法正常工作（参阅~~ [~~moby/moby#41017~~](https://github.com/moby/moby/issues/41017)~~）。~~
* ~~`testing-arm32v6` -- 与 `testing` 相同，但明确表示为 `arm32v6` 镜像。~~
* ~~`x.y.z-arm32v6` (例如 `1.16.0-arm32v6`) -- 与 `latest-arm32v6` 类似，但它代表一个特定的发布版本。~~

## 镜像更新 <a href="#image-updates" id="image-updates"></a>

偶尔，上游的 Bitwarden 项目（即 Bitwarden 公司）会对客户端做一些向后不兼容的改动，这就需要对服务器的实现做相应的改动。Vaultwarden 一般会及时推送新的版本来适应这些改动。

然而，由于上游控制着客户端的发布，而移动应用和浏览器扩展通常会自己自动更新，因此，对于 Vaultwarden 用户来说，保持更新为最新的 Vaultwarden 版本非常重要。否则，客户端与服务器版本不兼容可能会导致出现意外中断或异常。

网页密码库是唯一的例外：由于网页密码库与 Vaultwarden 镜像捆绑在一起，它的版本总是与 Vaultwarden 服务器的版本相匹配。如果您只把网页密码库用作客户端（可能性不大），那么你就不需要担心这些兼容性问题。

## 历史镜像 <a href="#historical-images" id="historical-images"></a>

在增加对多数据库支持的 1.17.0 版本之前，MySQL 和 PostgreSQL 的支持仅包含在单独的特定数据库镜像中。您仍可以在 Docker Hub 中找到它们，并且它们现在仍然在更新，但是，这些特定数据库的镜像将来会被移除，因此您应尽快过渡到使用统一的 `vaultwarden/server` 镜像。

* [`bitwardenrs/server-mysql`](https://hub.docker.com/r/bitwardenrs/server-mysql) -- 基于 Debian 的 `vaultwarden` 镜像，仅支持 MySQL（不支持 SQLite 和 PostgreSQL）。
* [`bitwardenrs/server-postgresql`](https://hub.docker.com/r/bitwardenrs/server-postgresql) -- 基于 Debian 的 `vaultwarden` 镜像，仅支持 PostgreSQL（不支持 SQLite 和 MySQL）。

## 历史标签 <a href="#historical-tags" id="historical-tags"></a>

在增加对多架构支持的 1.16.0 版本之前，所有特定架构镜像都有其自己的特定架构标签。自 2021-01-14 以来，这些标签已被移除，由于遵循过时的教程或未阅读发行说明，许多用户仍然最终拉取了这些旧的标签。

* `raspberry` -- Armv7hf 镜像。可以运行在 Raspberry Pi 2 或更新的版本上，也可以运行在任何其他兼容的板子上 。这个镜像不能在 Raspberry Pi 1 或 Raspberry Pi Zero 上运行，因为他们使用的是 armv6 CPU。
* `armv6` -- Armv6 镜像。可以运行在 Raspberry Pi 1 和 Raspberry Pi Zero 上。
* `aarch64` -- Aarch64 镜像。可以运行在 ARMv8 设备上，例如 Raspberry Pi 3 或其他基于 ARMv8 的设备。

需要**注意**的是，如果你在 Raspberry Pi 3 上使用 Raspbian，它要求在设备上安装 aarch64 发行版，但因为 Raspbian 是一个 `armv7hf` 发行版，您仍然需要使用 `raspberry` 标签。

## 已报告的兼容性表 <a href="#reported-compatibility-table" id="reported-compatibility-table"></a>

如果您在下表中尚未列出的硬件上可以正常运行镜像，请在此处添加您的详细信息。

<table><thead><tr><th width="165">使用的硬件</th><th width="150">OS</th><th width="188">Docker 架构</th><th width="196">使用的镜像</th><th width="150">状态</th><th>备注</th></tr></thead><tbody><tr><td>常规 64bit 服务器</td><td>Ubuntu 18.04</td><td>x86_64</td><td><code>vaultwarden/server</code></td><td>OK</td><td></td></tr><tr><td>O-Droid HC2</td><td>Armbian</td><td>arm7l (arm32)</td><td><code>registry.lollipopcloud.solutions/arm32v7/bitwarden</code> (see notes)</td><td>OK</td><td>从上游资源建立的非官方镜像；<code>vaultwarden/server:raspberry</code> 是官方的等效镜像</td></tr><tr><td>Raspberry Pi Zero W</td><td>Raspbian (4.14.98+)</td><td>linux/arm (armv6l)</td><td><code>vaultwarden/server:armv6</code></td><td>OK</td><td></td></tr><tr><td>Raspberry Pi Zero W</td><td>Raspbian (4.19.66+)</td><td>linux/arm (armv6l)</td><td><code>vaultwarden/server:latest</code> (Multiarch)</td><td>OK</td><td>只有在使用 docker 实验性功能 "docker pull --platform=linux/arm/v6"时，才能使用。否则会选择错误的镜像(<a href="https://github.com/dani-garcia/vaultwarden/issues/1064">https://github.com/dani-garcia/vaultwarden/issues/1064</a>)</td></tr><tr><td>Raspberry Pi 1 B</td><td>Raspbian (4.19.97+)</td><td>linux/arm (armv6l)</td><td><code>vaultwarden/server:armv6</code></td><td>OK</td><td></td></tr><tr><td>Raspberry Pi 3 B</td><td>Raspbian (4.14.98-v7+)</td><td>linux/arm (armv7l)</td><td><code>vaultwarden/server:raspberry</code></td><td>OK</td><td></td></tr><tr><td>Raspberry Pi 4</td><td>Raspbian (4.19.118-v7l+)</td><td>linux/arm (armv7l)</td><td><code>vaultwarden/server:raspberry</code></td><td>OK</td><td>4go 版本, rev 1.1</td></tr><tr><td>Synology</td><td>DSM (DSM 6.2.1-23824 Update 6)</td><td>Docker-x64-17.05.0-0367</td><td><code>vaultwarden/server:latest</code></td><td>OK</td><td></td></tr><tr><td>Synology</td><td>DSM (DSM 6.2.2-24922 Update 4)</td><td>Docker-x64-18.09.0-0506</td><td><code>vaultwarden/server:1.13.0-alpine</code></td><td>OK</td><td></td></tr><tr><td>Synology</td><td>DSM (DSM 7.2-64570 Update 1)</td><td>Docker-20.10.23</td><td><code>vaultwarden/server:latest</code></td><td>OK</td><td></td></tr><tr><td>常规 64bit 服务器</td><td>Unraid 6.8.0</td><td>19.03.5</td><td><code>vaultwarden/server:latest</code></td><td>OK</td><td></td></tr><tr><td>QNAP TS-451DEU (Intel Celeron J4025)</td><td>QTS 5.0.0.1891</td><td>x86_64</td><td><code>vaultwarden/server:latest</code></td><td>OK</td><td></td></tr><tr><td>常规 64bit 服务器</td><td>OpenMediaVault 6 (Debian Bullseye)</td><td>x86_64</td><td><code>vaultwarden/server:alpine</code></td><td>OK</td><td></td></tr></tbody></table>

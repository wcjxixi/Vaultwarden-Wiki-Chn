# 2.Fail2ban 设置

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Fail2Ban-Setup)
{% endhint %}

设置 Fail2ban 可以阻止攻击者暴力破解您的密码库登录。如果您的实例是公开的，这一点尤其重要。

## 目录 <a href="#table-of-contents" id="table-of-contents"></a>

* [预先说明](fail2ban-setup.md#pre-requisite)
* [安装](fail2ban-setup.md#installation)
  * [Debian / Ubuntu / Raspian](fail2ban-setup.md#debian-ubuntu-raspian)
  * [Fedora / Centos](fail2ban-setup.md#fedora-centos)
  * [群晖 DSM](fail2ban-setup.md#synology-dsm)
* [为网页密码库设置](fail2ban-setup.md#setup-for-web-vault)
  * [Filter](fail2ban-setup.md#filter)
  * [Jail](fail2ban-setup.md#jail)
* [为管理页面设置](fail2ban-setup.md#setup-for-admin-page)
  * [Filter](fail2ban-setup.md#filter-1)
  * [Jail](fail2ban-setup.md#jail-1)
* [测试 Fail2ban](fail2ban-setup.md#testing-fail-2-ban)
* [SELinux 中的问题](fail2ban-setup.md#selinux-problems)

## 预先说明 <a href="#pre-requisite" id="pre-requisite"></a>

* 文件名位于每个代码块的顶部。
* 从 1.5.0 版开始，Vaultwarden 支持记录到文件。请设置[日志记录](../logging.md)。
* 尝试使用错误的账户信息登录到网页版密码库，并检查日志文件中如下格式的记录项：

```
[YYYY-MM-DD hh:mm:ss][vaultwarden::api::identity][ERROR] Username or password is incorrect. Try again. IP: XXX.XXX.XXX.XXX. Username: email@domain.com.
```

## 安装 <a href="#installation" id="installation"></a>

### Debian / Ubuntu / Raspian Pi OS

```shell
sudo apt-get install fail2ban -y
```

### Fedora / Centos

需要 EPEL 库 (CentOS 7)

```shell
sudo yum install epel-release
sudo yum install fail2ban -y
```

### 群晖 DSM <a href="#synology-dsm" id="synology-dsm"></a>

使用 Synology 的话，由于各种原因需要做更多的工作。使用 Docker Compose 的完整的解决方案发布在[这里](https://github.com/sosandroid/docker-fail2ban-synology)。主要的问题是：

1. 嵌入式 IP 禁令系统不适用于 Docker 容器
2. 嵌入式 iptables 不支持 `REJECT` 块类型
3. Docker GUI 不允许某些高级设置
4. 修改系统配置不符合升级要求

因此，我们将在 Docker 容器中使用 Fail2ban。[Crazy-max/docker-fail2ban](https://github.com/crazy-max/docker-fail2ban) 提供了一个很好的解决方案，并且 Synology 的 Docker GUI 将被忽略。通过 SSH 的命令行，执行下列步骤（根据您的 Synology 配置调整 `volumeX`）：

1、获取 root 权限

```shell
sudo -i
```

2、创建持久性文件夹

```shell
mkdir -p /volumeX/docker/fail2ban/action.d/
mkdir -p /volumeX/docker/fail2ban/jail.d/
mkdir -p /volumeX/docker/fail2ban/filter.d/
```

3、将 blocktype 的 `REJECT` 替换为 `DROP` 块类型

```systemd
# /volumeX/docker/fail2ban/action.d/iptables.local

[Init]
blocktype = DROP
[Init?family=inet6]
blocktype = DROP
```

4、创建 docker-compose 文件

```batch
# /volumeX/docker/fail2ban/docker-compose.yml

version: '3'
services:
	fail2ban:
		container_name: fail2ban
		restart: always
		image: crazymax/fail2ban:latest
		environment: 
		- TZ=Europe/Paris
		- F2B_DB_PURGE_AGE=30d
		- F2B_LOG_TARGET=/data/fail2ban.log
		- F2B_LOG_LEVEL=INFO
		- F2B_IPTABLES_CHAIN=INPUT

		volumes:
		- /volumeX/docker/fail2ban:/data
		- /volumeX/docker/vw-data:/vaultwarden:ro

		network_mode: "host"

		privileged: true
		cap_add:
			- NET_ADMIN
			- NET_RAW
```

5、使用命令行启动容器

```shell
cd /volumeX/docker/fail2ban
docker-compose up -d
```

您现在应该看到该容器在 Synolog 的 Docker GUI 中运行了。在配置筛选器和 jail 后，您必须重新加载。

## 为网页密码库设置 <a href="#setup-for-web-vault" id="setup-for-web-vault"></a>

按照惯例，`path_f2b` 代表 Fail2ban 工作所需的路径。这取决于您的系统，例如在 Synology 上是 `/volumeX/docker/fail2ban/`，但在其他系统上是 `/etc/fail2ban/`。

### Filter <a href="#filter" id="filter"></a>

使用如下内容创建文件：

```systemd
# path_f2b/filter.d/vaultwarden.local

[INCLUDES]
before = common.conf

[Definition]
failregex = ^.*Username or password is incorrect\. Try again\. IP: <ADDR>\. Username:.*$
ignoreregex =
```

**提示**：如果在 `fail2ban.log` 中出现以下错误消息 (CentOS 7, Fail2Ban v0.9.7) \
`fail2ban.filter [5291]: ERROR No 'host' group in '^.*Username or password is incorrect\. Try again\. IP: <ADDR>\. Username:.*$'`\
请将 `vaultwarden.local` 中的 `<ADDR>` 改为 `<HOST>`。

**提示**：对于 Cloudflare 用户，请确保在**管理面板** -> **高级设置** -> **客户端 IP 标头**中将客户端 IP 标头设置为 `CF-Connecting-IP`，否则客户端的真实 IP 将不会被识别和阻止。

**提示**：如果您在 `vaultwarden.log` 中看到 127.0.0.1 是登录失败的 IP 地址，那么您可能正在使用反向代理，而 Fail2ban 无法正常工作：

```
[YYYY-MM-DD hh:mm:ss][vaultwarden::api::identity][ERROR] Username or password is incorrect. Try again. IP: 127.0.0.1. Username: email@example.com.
```

要解决这个问题，需要通过 X-Real-IP 头将真实的远程地址转发给 Vaultwarden。如何操作呢？根据你使用的代理服务器不同而不同。例如，在 Caddy 2.x 中，当您定义反向代理时，同时定义 `header_up X-Real-IP {remote_host}`。更多信息请参阅[代理示例](../../deployment/proxy-examples.md)。

### Jail

> \[**译者注**]：[什么是 Jail](https://docs.freebsd.org/zh-cn/books/arch-handbook/jail/)

使用如下内容创建文件：

```systemd
# path_f2b/jail.d/vaultwarden.local

[vaultwarden]
enabled = true
port = 80,443,8081
filter = vaultwarden
banaction = %(banaction_allports)s
logpath = /path/to/vaultwarden.log
maxretry = 3
bantime = 14400
findtime = 14400
```

**Docker 用户注意：**

Docker 使用 FORWARD 链而不是默认的 INPUT 链。如果接收请求的机器将他们直接映射到 Docker 容器，那么无论容器里有什么（反向代理、Vaultwarden 等），链都需要适当地设置。默认的 `action` 被设置为`action_`（它使用 `banaction`，其别名我们设置为 `banaction_allports`），`action_` 已经考虑了链的问题，因此，只需设置 `chain` 即可。参阅[这个类似的问题](https://forum.openwrt.org/t/resolved-fail2ban-and-iptables-ip-bans-not-blocked/90057)。

```systemd
chain = FORWARD
```

{% hint style="info" %}
如果您使用 systemd 来管理 Vaultwarden，您可以为 Fail2ban 使用 systemd-journal：

```
backend = systemd
filter = vaultwarden[journalmatch='_SYSTEMD_UNIT=your_vaultwarden.service']
```

使用它们来代替 `logpath =` 和 `filter =` 变量。
{% endhint %}

**Cloudflare 用户注意：**

如果您使用 Cloudflare 代理，您需要将 Cloudflare 添加到您的操作列表中，如[这个指南](https://niksec.com/using-fail2ban-with-cloudflare/)中所示。

重新加载 Fail2ban 使更改生效：

```shell
sudo systemctl reload fail2ban
```

随意更改您认为合适的选项。

## 为管理页面设置 <a href="#setup-for-admin-page" id="setup-for-admin-page"></a>

如果您通过设置 `ADMIN_TOKEN` 环境变量启用了管理控制台，则可以使用 Fail2ban 来阻止攻击者暴力破解您的管理令牌。该过程与网页密码库相同。

### Filter <a href="#filter" id="filter"></a>

使用如下内容创建文件：

```systemd
# path_f2b/filter.d/vaultwarden-admin.local

[INCLUDES]
before = common.conf

[Definition]
failregex = ^.*Invalid admin token\. IP: <ADDR>.*$
ignoreregex =
```

**提示**：如果在 `fail2ban.log` 中出现以下错误消息：`ERROR NOK: ("No 'host' group in '^.*Invalid admin token\\. IP: <ADDR>.*$'")`，请将 `vaultwarden-admin.local` 中的 `<ADDR>` 改为 `<HOST>`

### Jail

使用如下内容创建文件：

```systemd
# path_f2b/jail.d/vaultwarden-admin.local

[vaultwarden-admin]
enabled = true
port = 80,443
filter = vaultwarden-admin
banaction = %(banaction_allports)s
logpath = /path/to/vaultwarden.log
maxretry = 3
bantime = 14400
findtime = 14400
```

**注意**：Docker 使用 FORWARD 链而不是默认的 INPUT 链。因此，当使用 Docker 时，请使用下面的 `action` 行替换掉 `banaction` 行：

```systemd
action = iptables-allports[name=vaultwarden, chain=FORWARD]
```

{% hint style="info" %}
如果您使用 systemd 来管理 Vaultwarden，您同样可以在这里为 Fail2ban 使用 systemd-journal：

```
backend = systemd
filter = vaultwarden-admin[journalmatch='_SYSTEMD_UNIT=your_vaultwarden.service']
```

使用它们来代替 `logpath =` 和 `filter =` 变量。
{% endhint %}

**Cloudflare 用户请注意：**

如果您使用 Cloudflare 代理，您需要将 Cloudflare 添加到您的操作列表中，如[本指南](https://niksec.com/using-fail2ban-with-cloudflare/)中所示。

重新加载 Fail2ban 使更改生效：

```shell
sudo systemctl reload fail2ban
```

## 测试 Fail2ban <a href="#testing-fail-2-ban" id="testing-fail-2-ban"></a>

现在，尝试使用任何电子邮件地址登录 Vaultwarden（不必是有效电子邮件，只需是电子邮件格式即可）。如果它可以正常工作，您的 IP 将被阻止。运行以下命令来取消阻止的 IP：

```shell
# 使用 Docker
sudo docker exec -t fail2ban fail2ban-client set vaultwarden unbanip XX.XX.XX.XX
# 未使用 Docker
sudo fail2ban-client set vaultwarden unbanip XX.XX.XX.XX
```

如果 Fail2ban 无法正常运行，请检查 Vaultwarden 日志文件的路径是否正确。对于 Docker：如果指定的日志文件未生成和/或更新，请确保将 `EXTENDED_LOGGING` 变量设置为 `true`（默认值），并且确保日志文件的路径是 Docker 内部的路径（当您使用 `/vw-data/:/data/` 时，日志文件应位于容器外部的 `/data/...` 中）。

还要确认 Docker 容器的时区与主机的时区是否一致。通过将日志文件中显示的时间与主机操作系统的时间进行比较来进行检查。如果它们不一致，则有多种解决方法。一种是使用 `-e "TZ = <timezone>"` 选项启动 Docker 。可用的时区（比如 `-e TZ = "Australia/Melbourne"`）列表在[这里](https://en.wikipedia.org/wiki/List\_of\_tz\_database\_time\_zones)查看。

如果您使用的是 podman 而不是 Docker，则无法通过 `-e "TZ = <timezone>"` 来设置时区。可以按照以下指南解决此问题（当使用 alpine 镜像时）：[https://wiki.alpinelinux.org/wiki/Setting\_the\_timezone](https://wiki.alpinelinux.org/wiki/Setting\_the\_timezone)。

## SELinux 中的问题 <a href="#selinux-problems" id="selinux-problems"></a>

当使用 SELinux 时，SELinux 可能会阻止 Fail2ban 读取日志。如果是这样，请运行此命令： `sudo tail /var/log/audit/audit.log`。您应该会看到如下类似内容（当然，实际的审核 ID (pid) 会因您的情况而不一样）：

```systemd
type=AVC msg=audit(1571777936.719:2193): avc:  denied  { search } for  pid=5853 comm="fail2ban-server" name="containers" dev="dm-0" ino=1144588 scontext=system_u:system_r:fail2ban_t:s0 tcontext=unconfined_u:object_r:container_var_lib_t:s0 tclass=dir permissive=0
```

您可以使用 `grep 'type=AVC msg=audit(1571777936.719:2193)' /var/log/audit/audit.log | audit2why` 来找出真正的原因。`audit2allow -a` 将为您提供有关如何创建模块并允许 Fail2ban 访问日志的具体说明。

按照这些步骤操作后就结束了！Fail2ban 现在应该可以正常工作了。

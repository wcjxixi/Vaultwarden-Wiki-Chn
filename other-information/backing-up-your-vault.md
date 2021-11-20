# 2.备份您的密码库

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Backing-up-your-vault)
{% endhint %}

## 概览 <a href="overview" id="overview"></a>

应该定期备份 Vaultwarden 数据，并且最好是通过自动化的流程（例如，cron 作业）。理想情况下，应该远程（例如，云存储或不同的计算机）存储至少一个副本。避免依赖文件系统或虚拟机快照作为备份方法，因为这些都是比较复杂的操作，可能会出现更多的问题，在这种情况下的恢复操作对普通用户来说是困难的，甚至是不可能的。在备份上添加额外的加密层通常是个好主意（尤其是当您的备份还包含配置数据时，例如您的[管理员令牌](../configuration/enabling-admin-page.md)），但如果您确信您的主密码（以及您的其他用户的主密码，如果有的话）足够强大，也可以选择跳过这一步。

## 备份您的数据 <a href="backing-up-data" id="backing-up-data"></a>

默认情况下，Vaultwarden 将所有的数据存储在一个名为 `data` 的目录下（与 `vaultwarden` 可执行文件位于同一目录下）。这个位置可以通过设置 [DATA\_FOLDER](../configuration/changing-persistent-data-location.md) 环境变量来改变。如果你使用 SQLite 运行 Vaultwarden（这是最常见的设置），那么 SQL 数据库只是 data 文件夹中的一个文件。如果你使用 MySQL 或 PostgreSQL 运行，则必须单独转储这些数据 -- 这超出了本文的范围，但在网上搜索会发现有许多涵盖了这个主题的教程。

当使用默认的 SQLite 后端运行时，Vaultwarden 的 `data` 目录具有如下的结构：

```
data
├── attachments          # 每一个附件都作为单独的文件存储在此目录下。
│   └── <uuid>           # （如果未创建附件，则此 attachments 目录将不存在）
│       └── <random_id>
├── config.json          # 存储管理页面配置；仅在之前已启用管理页面的情况下存在。
├── db.sqlite3           # 主 SQLite 数据库文件。
├── db.sqlite3-shm       # SQLite 共享内存文件（并非始终存在）。
├── db.sqlite3-wal       # SQLite 预写日志文件（并非始终存在）。
├── icon_cache           # 站点图标（favicon）缓存在此目录下。
│   ├── <domain>.png
│   ├── example.com.png
│   ├── example.net.png
│   └── example.org.png
├── rsa_key.der          # ‘rsa_key.*’文件用于签署验证令牌。
├── rsa_key.pem
├── rsa_key.pub.der
└── sends                # 每一个 Send 的附件都作为单独的文件存储在此目录下。
    └── <uuid>           # （如果未创建 Send 附件，则这个 sends 目录将不存在）
        └── <random_id>
```

当使用 MySQL 或 PostgreSQL 后端运行时，目录结构是一样的，只是没有 SQLite 文件。你仍然需要备份数据目录中的文件，以及 MySQL 或 PostgreSQL 表的转储。

接下来详细讨论每一系列文件。

### SQLite 数据库文件 <a href="sqlite-database-files" id="sqlite-database-files"></a>

_**需要备份。**_

SQLite \_\*\*\*\*\_数据库文件（`db.sqlite3`）存储了几乎所有重要的 Vaultwarden 数据/状态（数据库条目、用户/org/设备元数据等），主要的例外是附件，附件作为单独的文件存储在文件系统中。

您一般应使用 SQLite CLI（`sqlite3`）中的 `.backup` 命令来备份数据库文件。该命令使用 [Online Backup API](https://www.sqlite.org/backup.html)，SQLite 文档是备份可能正在被使用的数据库文件的[最佳方式](https://www.sqlite.org/howtocorrupt.html#\_backup\_or\_restore\_while\_a\_transaction\_is\_active)。如果你能确保数据库在备份运行时未被使用，你也可以使用其他方式，例如 `.dump` 命令，或者简单地复制所有 SQLite 数据库文件（包括 `-wal` 文件，如果存在的话）。

假设你的数据文件夹是 `data`（默认），一个基本的备份命令看起来像这样：

```sql
sqlite3 data/db.sqlite3 ".backup '/path/to/backups/db-$(date '+%Y%m%d-%H%M').sqlite3'"
```

您也可以使用 `VACUUM INTO`，这将压缩空闲空间，但需要更多的处理时间：：

```sql
sqlite3 data/db.sqlite3 "VACUUM INTO '/path/to/backups/db-$(date '+%Y%m%d-%H%M').sqlite3'"
```

假设此命令在 2021 年 1 月 1 日中午 12:34（当地时间）运行，这将备份你的 SQLite 数据库文件到 `/path/to/backups/db-20210101-1234.sqlite3`。

你可以通过一个 cron 作业定期运行这个命令（最好每天至少一次）。如果您通过 Docker 运行，请注意 Docker 映像不包含 sqlite3 二进制文件或 cron 守护程序，因此通常会将它们安装在 Docker 主机本身上并在容器外运行 cron 作业。如果您出于某种原因确实想从容器内运行备份，您可以在[容器启动](../container-image-usage/starting-a-container.md#customizing-container-startup)期间安装任何必要的包，或者使用您首选的 `vaultwarden/server:<tag>` 镜像作为父镜像创建您自己的自定义 Docker 镜像。

如果你想把备份数据复制到云存储上，[Rclone](https://rclone.org) 是一个有用的工具，可以与各种云存储系统进行对接。[restic](https://restic.net) 是另一个不错的选择，特别是如果您有较大的附件，并想避免每次都将其作为备份的一部分的时候。

### `attachments` 目录 <a href="the-attachments-dir" id="the-attachments-dir"></a>

_**需要备份。**_

[文件附件](https://help.bitwarden.in/your-vault/file-attachments)是唯一不存储在数据库表中的重要数据，主要是因为它们可以是任意大小，而 SQL 数据库一般不是为了有效处理大的 blob 而设计的。如果未创建文件附件，则该目录将不存在。

### `sends` 目录 <a href="the-sends-dir" id="the-sends-dir"></a>

_**可选备份。**_

与常规文件附件一样，Send 文件附件也不存储在数据库表中（但 Send 的文本注释存储在数据库中）。

与常规附件不同，Send 附件的目的是短暂的。因此，如果要最小化备份的大小，则可以选择不备份此目录。另一方面，如果在还原过程中保持现有 Send 的正常功能更为重要，那么您应该备份此目录。

如果未创建任何 Send 附件，则该目录将不存在。

### `config.json` 文件 <a href="the-config-json-file" id="the-config-json-file"></a>

_**建议备份。**_

如果你使用管理页面来配置你的 Vaultwarden 实例，并且没有使用其他方式来备份你的配置，那么你可能需要备份此文件，这样你以后就不必重新配置你想要的配置了。

请记住，这个文件确实包含了一些可能被认为是敏感的明文数据（管理员令牌、SMTP 凭据等），所以如果你担心别人可能会访问这些数据（例如，上传到云存储时），一定要对这些数据进行加密。

### `rsa_key*` 文件 <a href="the-rsa_key-files" id="the-rsa_key-files"></a>

_**建议备份。**_

这些文件用于签署当前登录用户的 [JWT](https://en.wikipedia.org/wiki/JSON\_Web\_Token)（验证令牌）。删除这些文件将简单地注销每个用户，迫使他们重新登录。

**\[译者注]**：[JWT](https://jwt.io)（JSON Web Tokens），是一种基于 JSON 的、用于在网络上声明某种主张的令牌（token）。JWT 通常由三部分组成:：头信息（header）、消息体（payload）和签名（signature）。

### `icon_cache` 目录 <a href="the-icon_cache-dir" id="the-icon_cache-dir"></a>

_**可选备份。**_

图标缓存用于存储[网站图标](https://help.bitwarden.in/security/privacy-when-using-website-icons)，这样就不需要反复从登录网站获取图标。这可能不值得备份，除非你真的想避免重新获取大量的图标缓存。

## 恢复备份数据 <a href="restoring-backup-data" id="restoring-backup-data"></a>

确保 Vaultwarden 已经停止，然后简单地将 `data` 文件夹中的每个文件或目录替换为它的备份版本即可。

当恢复使用 `.backup` 或 `VACUUM INTO` 创建的备份时，确保首先删除任何已存在的 `db.sqlite3-wal` 文件，因为当 SQLite 试图使用陈旧/不匹配的 WAL 文件恢复 `db.sqlite3` 时，有可能导致数据库损坏。然而，如果你直接拷贝 `db.sqlite3` 文件和其匹配的 `db.sqlite3-wal` 文件的方式来备份数据库，那么你必须将两个文件作为一对来恢复。不需要备份或恢复 `db.sqlite3-shm` 文件。

为了验证你的备份是否能正常工作，定期运行从备份中恢复的过程是个好主意。这样做的时候，请确保移动或保留原始数据的副本，以防备份实际上不能正常工作。

## 示例 <a href="examples" id="examples"></a>

本部分是第三方备份示例的索引。在使用某个示例之前，您应该彻底阅读此示例并了解其功能。

* [https://github.com/ttionya/vaultwarden-backup](https://github.com/ttionya/vaultwarden-backup)
* [https://github.com/shivpatel/bitwarden\_rs-local-backup](https://github.com/shivpatel/bitwarden\_rs-local-backup)
* [https://github.com/shivpatel/bitwarden\_rs\_dropbox\_backup](https://github.com/shivpatel/bitwarden\_rs\_dropbox\_backup)
* [https://gitlab.com/1O/bitwarden\_rs-backup](https://gitlab.com/1O/bitwarden\_rs-backup)
* [https://github.com/jjlin/vaultwarden-backup](https://github.com/jjlin/vaultwarden-backup)
* [https://github.com/jmqm/vaultwarden\_backup](https://github.com/jmqm/vaultwarden\_backup)

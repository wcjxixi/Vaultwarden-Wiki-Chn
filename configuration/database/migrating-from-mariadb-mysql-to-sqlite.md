# 4.从 MariaDB (MySQL) 迁移到 SQLite

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Migrating-from-MariaDB-\(MySQL\)-to-SQLite)
{% endhint %}

{% hint style="danger" %}
使用这些命令的风险自负！

在做任何可能破坏整个密码库的事情之前，请务必创建备份！
{% endhint %}

## 常规 <a href="#general" id="general"></a>

Vaultwarden 最初设计时仅使用 SQLite，但当时 MariaDB (MySQL) 和 PostgreSQL 也被同时添加到了组合中。对于 SQLite，您不必运行单独的服务器或容器，而对于其他两个，您确实需要运行一些额外的东西。

如果您一开始使用的是 MariaDB 并想回到 SQLite，现在该怎么办呢？嗯，这是可能的，但是使用以下步骤可能会出现一些我们不知道的奇怪故障。如果您遇到任何奇怪的问题然后需要帮助，或者您解决了这些问题，请在此处开启一个新的讨论：[https://github.com/dani-garcia/vaultwarden/discussions](https://github.com/dani-garcia/vaultwarden/discussions)，以帮助您和其他人。

## 如何从 MariaDB 迁移到 SQLite <a href="#how-to-migrate-from-mariadb-to-sqlite" id="how-to-migrate-from-mariadb-to-sqlite"></a>

确保您对 SQLite 和 MariaDB 使用的是相同版本的 Vaultwarden（Docker 或自定义构建），不要在这些步骤之间更新 Docker 镜像。要迁移到 SQLite，我们首先需要有一个 SQLite 数据库文件，我们可以用它来传输数据。要创建此文件，您需要停止当前的 Vaultwarden 实例，并将其配置为使用 SQLite。例如，您可以通过将 `DATABASE_URL` 从 `DATABASE_URL=mysql://<vaultwarden_user>:<vaultwarden_pw>@mariadb/vaultwarden` 更改为 `DATABASE_URL=/data/db.sqlite3` 来实现。（ `/data` 是您使用的 `-v` 值的 Docker 容器内的内部路径）。

更改此配置后，启动 Vaultwarden，检查以 `Executing migration script .....` 开头的行的日志信息，这些信息显示它执行了一些迁移。

现在再次停止 Vaultwarden，以便您可以开始迁移过程。需要 MariaDB 的数据库主机和凭据才能继续。

现在运行以下单行程序并将 `<dbhost>`、`<dbuser>` 以及 `<database>` 调整为您用于 MariaDB 连接的实际内容：

```shell
mysqldump \
  --host=<dbhost> \
  --user=<dbuser> --password \
  --skip-create-options \
  --compatible=ansi \
  --skip-extended-insert \
  --compact \
  --single-transaction \
  --no-create-db \
  --no-create-info \
  --hex-blob \
  --skip-quote-names <database> \
  | grep -a "^INSERT INTO" | grep -a -v "__diesel_schema_migrations" \
  | sed 's#\\"#"#gm' \
  | sed -sE "s#,0x([^,]*)#,X'\L\1'#gm" \
   > mysql-to-sqlite.sql
```

系统会提示您输入密码，输入密码然后按回车键。

这一步将生成一个用于保存您的数据库的 `mysql-to-sqlite.sql` 文件。现在查找上一步中在您第一次使用 SQLite 作为数据库启动 Vaultwarden 时由 Vaultwarden 创建的 db.sqlite3 文件。复制或移动 `mysql-to-sqlite.sql`，让 db.sqlite3 和导出位于同一目录中。现在您可以执行以下命令：

```shell
sqlite3 db_new.sqlite3 < mysql-to-sqlite.sql
```

这一步将使用转储填充 SQLite 数据库，您现在可以使用 SQLite 而非 MySQL 再次启动 Vaultwarden 了。

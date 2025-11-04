# 2.使用 PostgreSQL 后端

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Using-the-PostgreSQL-Backend)
{% endhint %}

要使用 PostgreSQ 后端，您可以使用[官方 Docker 镜像](https://hub.docker.com/r/bitwardenrs/server-postgresql)，也可以构建您自己的[启用了 PostgreSQL](../../deployment/building-binary.md#postgresql-backend) 的二进制。

要运行二进制或容器，请确保已设置 `DATABASE_URL` 环境变量（即 `DATABASE_URL='postgresql://<user>:<password>@postgresql/vaultwarden'`）。

**字符串连接语法：**

```systemd
DATABASE_URL=postgresql://[[user]:[password]@]host[:port][/database]
```

docker 运行环境变量的一个示例：`-e 'DATABASE_URL=postgresql://user_name:user_password@db_host:5432/vaultwarden'`。

如果您需要设置额外的连接参数，请注意 `DATABASE_URL` 的值最终会被 `libpq` 解析，因此您可以使用 libpq [文档](https://www.postgresql.org/docs/current/libpq-envars.html)中所列出的任何参数。您可以将连接参数添加到 `DATABASE_URL` 中或通过其相应的 `PG*` 环境变量指定它。如果在 Docker 下运行，请记住提供的任何路径都需要从 Docker 容器的角度来看，而不是 Docker 主机。

如果您要使用自定义架构/搜索路径，则需要使用以下连接字符串（注意 URL 编码的字符，比如 `%20` 表示空格，`%3D` 表示 `=` 符号）：

```systemd
DATABASE_URL=postgresql://user_name:user_password@db_host:5432/vaultwarden?application_name=vaultwarden&options=-c%20search_path%3Ddb_schema
```

如果您的密码包含特殊字符，则需要使用百分号编码。

| !   | #   | $   | %   | &   | '   | (   | )   | \*  | +   | ,   | /   | :   | ;   | =   | ?   | @   | \[  | ]   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| %21 | %23 | %24 | %25 | %26 | %27 | %28 | %29 | %2A | %2B | %2C | %2F | %3A | %3B | %3D | %3F | %40 | %5B | %5D |

完整的代码列表可以在 [Wikipedia 的百分号编码页面](https://zh.wikipedia.org/wiki/%E7%99%BE%E5%88%86%E5%8F%B7%E7%BC%96%E7%A0%81)上找到。

## **从 SQLite 迁移到 PostgreSQL** <a href="#migrating-from-sqlite-to-postgresql" id="migrating-from-sqlite-to-postgresql"></a>

> 使用 SQLite3 3.37.2、PostgreSQL 16.10 和 Vaultwarden 1.34.3 测试

从 SQLite 迁移到 PostgreSQL 或 MySQL 的方法比较简单，但请注意，**使用此方法风险自负，并且强烈建议备份您的安装和数据**！这**不受支持**，也没有经过强有力的测试。

1、创建一个新的数据库用户：

```sql
CREATE USER vaultwarden WITH ENCRYPTED PASSWORD 'yourpassword';
```

2、为 Vaultwarden 创建一个新的（空）数据库，将该用户设置为数据库所有者：

```sql
CREATE DATABASE vaultwarden OWNER vaultwarden;
```

3、配置 Vaultwarden 并启动它，以便 [diesel](http://diesel.rs/) 可以运行迁移并设置正确的模式。除此之外不要做别的。

4、停止 Vaultwarden。

5、安装 [pgloader](http://pgloader.io/) 。

6、对 SQLite 数据库[禁用 WAL](running-without-wal-enabled.md#id-1-disable-wal-on-old-db)。

7、使用如下内容创建 `vaultwarden.load` 文件：

```sql
load database
     from sqlite:///where/you/keep/your/vaultwarden/db.sqlite3 
     into postgresql://yourpgsqluser:yourpgsqlpassword@yourpgsqlserver:yourpgsqlport/yourpgsqldatabase
     WITH data only, include no drop, reset sequences
     EXCLUDING TABLE NAMES LIKE '__diesel_schema_migrations'
     ALTER SCHEMA 'vaultwarden' RENAME TO 'public'
;
```

8、运行 `pgloader vaultwarden.load` 命令，您可能会看到一些警告，（不用理会）迁移会成功完成。

9、重新启动 Vaultwarden。

## 从 MySQL 迁移到 PostgreSQL <a href="#migrating-from-mysql-to-postgresql" id="migrating-from-mysql-to-postgresql"></a>

> 使用 MariaDB 10.11.9、PostgreSQL 15.8-1 和 Vaultwarden 1.32.0 测试

请注意，**使用此方法风险自负，并且强烈建议备份您的安装和数据**！这**不受支持**，也没有经过强有力的测试。

1、创建一个新的数据库用户：

```sql
CREATE USER vaultwarden WITH ENCRYPTED PASSWORD 'yourpassword';
```

2、为 Vaultwarden 创建一个新的（空）数据库，将该用户设置为数据库所有者：

```sql
CREATE DATABASE vaultwarden OWNER vaultwarden;
```

3、配置 Vaultwarden 并启动它，以便 [diesel](http://diesel.rs/) 可以运行迁移并设置正确的模式。除此之外不要做别的。

4、停止 Vaultwarden。

5、安装 [pgloader](http://pgloader.io/)。确保您使用的是最新版本的 pgloader，官方的 Ubuntu 存储库有一个过时的版本，它不能与新版本的 PostgreSQL 一起正常工作。最新版本可以从 [PostgreSQL Apt 存储库](https://www.postgresql.org/download/linux/ubuntu/)获取。

6、使用如下内容创建 `vaultwarden.load` 文件：

```sql
load database
     from mysql://yourmysqluser:yourmysqlpassword@yourmysqlserver:yourmysqlport/yourmysqldatabase 
     into postgresql://yourpgsqluser:yourpgsqlpassword@yourpgsqlserver:yourpgsqlport/yourpgsqldatabase
     WITH data only
     EXCLUDING TABLE NAMES MATCHING '__diesel_schema_migrations'
     ALTER SCHEMA 'vaultwarden' RENAME TO 'public'
;
```

_如果您的连接需要 SSL，可以选择将 `?sslmode=require` 添加到 PostgreSQL 连接字符串中。_

7、运行 `pgloader vaultwarden.load` 命令，您可能会看到一些警告，（不用理会）迁移会成功完成。如果有错误，很可能是您的 pgloader 版本过时了！

8、重新启动 Vaultwarden。

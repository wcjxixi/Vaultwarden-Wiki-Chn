# 1.使用 MariaDB (MySQL) 后端

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Using-the-MariaDB-\(MySQL\)-Backend)
{% endhint %}

{% hint style="warning" %}
⚠️ 💩 ⚠️我们的构建基于 MariaDB 客户端库，因为这是 Debian 提供的。

对最新 Oracle MySQLv8 版本的支持需要额外注意。

如果您坚持使用 MySQLv8 而不是 MariaDB，请使用旧的密码散列方法而不是默认方法创建用户！⚠️ 💩 ⚠️
{% endhint %}

要使用 MySQL 后端，你可以使用[官方 Docker 镜像](https://hub.docker.com/r/bitwardenrs/server-mysql)，也可以构建您自己的[启用了 MySQL](../../deployment/building-binary.md#mysql-backend) 的二进制。

要运行二进制或容器，请确保已设置 `DATABASE_URL` 环境变量（即 `DATABASE_URL='mysql://<user>:<password>@mysql/vaultwarden[?ssl_mode=(disabled|required|preferred)&ssl_ca=/path/to/cart.(crt|pem)]'`）。

**连接字符串语法：**

```systemd
DATABASE_URL=mysql://[[user]:[password]@]host[:port][/database]
```

如果密码包含特殊字符，则需要使用百分号编码。

<table><thead><tr><th width="150">!</th><th width="150">#</th><th width="150">$</th><th width="150">%</th><th width="150">&#x26;</th><th width="150">'</th><th width="150">(</th><th width="150">)</th><th width="150">*</th><th width="150">+</th><th width="150">,</th><th width="150">/</th><th width="150">:</th><th width="150">;</th><th width="150">=</th><th width="150">?</th><th width="150">@</th><th width="150">[</th><th>]</th></tr></thead><tbody><tr><td>%21</td><td>%23</td><td>%24</td><td>%25</td><td>%26</td><td>%27</td><td>%28</td><td>%29</td><td>%2A</td><td>%2B</td><td>%2C</td><td>%2F</td><td>%3A</td><td>%3B</td><td>%3D</td><td>%3F</td><td>%40</td><td>%5B</td><td>%5D</td></tr></tbody></table>

完整的代码列表可以在 [Wikipedia 的百分号编码页面](https://zh.wikipedia.org/wiki/%E7%99%BE%E5%88%86%E5%8F%B7%E7%BC%96%E7%A0%81)上找到。

## 使用 Docker 的示例 <a href="#example-using-docker" id="example-using-docker"></a>

```shell
# 启动 mysql 容器
docker run --name mysql --net <some-docker-network>\
 -e MYSQL_ROOT_PASSWORD=<my-secret-pw>\
 -e MYSQL_DATABASE=vaultwarden\
 -e MYSQL_USER=<vaultwarden_user>\
 -e MYSQL_PASSWORD=<vaultwarden_pw> -d mysql:5.7

# 使用 MySQL 环境变量值启动 vaultwarden
docker run -d --name vaultwarden --net <some-docker-network>\
 -v $(pwd)/vw-data/:/data/ -v <Path to ssl certs>:/ssl/\
 -p 443:80 -e ROCKET_TLS='{certs="/ssl/<your ssl cert>",key="/ssl/<your ssl key>"}'\
 -e RUST_BACKTRACE=1 -e DATABASE_URL='mysql://<vaultwarden_user>:<vaultwarden_pw>@mysql/vaultwarden'\
 -e ADMIN_TOKEN=<some_random_token_as_per_above_explanation>\
 -e ENABLE_DB_WAL='false' <you vaultwarden image name>
```

### 使用非 Docker MySQL 服务器的示例 <a href="#example-using-non-docker-mysql-server" id="example-using-non-docker-mysql-server"></a>

```systemd
Server IP/Port 192.168.1.10:3306 UN: dbuser / PW: yourpassword / DB: vaultwarden
mysql://dbuser:yourpassword@192.168.1.10:3306/vaultwarden
```

### 使用 docker-compose 的示例 <a href="#example-using-docker-compose" id="example-using-docker-compose"></a>

```batch
version: "3.7"
services:
 mariadb:
  image: "mariadb"
  container_name: "mariadb"
  hostname: "mariadb"
  restart: always
  env_file:
   - ".env"
  volumes:
   - "mariadb_vol:/var/lib/mysql"
   - "/etc/localtime:/etc/localtime:ro"
  environment:
   - "MYSQL_ROOT_PASSWORD=<my-secret-pw>"
   - "MYSQL_PASSWORD=<vaultwarden_pw>"
   - "MYSQL_DATABASE=vaultwarden_db"
   - "MYSQL_USER=<vaultwarden_user>"

 vaultwarden:
  image: "vaultwarden/server-mysql:latest"
  container_name: "vaultwarden"
  hostname: "vaultwarden"
  restart: always
  env_file:
   - ".env"
  volumes:
   - "vaultwarden_vol:/data/"
  environment:
## 当在 mysql URL 周围使用单括号时会出现问题，就像在普通的 docker 例子中的一样
   - "DATABASE_URL=mysql://<vaultwarden_user>:<vaultwarden_pw>@mariadb/vaultwarden_db"
   - "ADMIN_TOKEN=<some_random_token_as_per_above_explanation>"
   - "RUST_BACKTRACE=1"
  ports:
   - "80:80"

volumes:
 vaultwarden_vol:
 mariadb_vol:
```

## 手动创建数据库（例如，使用现有的数据库服务器） <a href="#manually-create-a-database-for-example-using-an-existing-database-server" id="manually-create-a-database-for-example-using-an-existing-database-server"></a>

{% hint style="danger" %}
要执行这些查询，您需要有一个可以创建新数据库和用户的用户。大多数情况下，这将是 `root` 用户，但根据您的数据库可能会有所不同。
{% endhint %}

> 使用上面的 docker-compose 示例使这些步骤变得不必要。数据库、排序规则和字符集在启动时将被自动创建。

### 创建数据库和用户 <a href="#create-database-and-user" id="create-database-and-user"></a>

1、为 Vaultwarden 创建一个新的（空）数据库（确保字符集和排序规则正确！）：

```sql
CREATE DATABASE vaultwarden CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

2a、创建一个新的数据库用户并授予数据库权限（对于 MariaDB，版本低于 v8 的 MySQL）：

```sql
CREATE USER 'vaultwarden'@'localhost' IDENTIFIED BY 'yourpassword';
GRANT ALL ON `vaultwarden`.* TO 'vaultwarden'@'localhost';
FLUSH PRIVILEGES;
```

2b、如果使用 MySQL v8.x，则需要这样创建用户：

```sql
-- 在 MySQLv8 安装上这样使用
CREATE USER 'vaultwarden'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
GRANT ALL ON `vaultwarden`.* TO 'vaultwarden'@'localhost';
FLUSH PRIVILEGES;
```

如果您已经创建了用户，想要更改密码类型：

```sql
-- 密码类型由 caching_sha2_password 更改未原生
ALTER USER 'vaultwarden'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
```

您可能想尝试一组受限的授权：

```sql
GRANT ALTER, CREATE, DELETE, DROP, INDEX, INSERT, REFERENCES, SELECT, UPDATE ON `vaultwarden`.* TO 'vaultwarden'@'localhost';
FLUSH PRIVILEGES;
```

## 从 SQLite 迁移到 MySQL <a href="#migrating-from-sqlite-to-mysql" id="migrating-from-sqlite-to-mysql"></a>

此[话题评论](https://github.com/dani-garcia/vaultwarden/issues/497#issuecomment-511827057)中描述了一种从 SQLite 迁移到 MySQL 的简单方法。下面重复这些步骤。请注意，使用此方法风险自负，强烈建议备份您的安装和数据！

1、首先遵循上面的步骤 1 和步骤 2。

2、配置 Vaultwarden 并启动它，以便 [diesel](http://diesel.rs/) 可以运行迁移并正确设置模式。除此之外不要做别的。

3、停止 Vaultwarden。

4、使用下面的命令转储您现有的 SQLite 数据库。再次检查您的 sqlite 数据库的名称，默认应该是 `db.sqlite`。

**注意**：在您的 Linux 系统上需要已经安装了 sqlite3 命令。

我们需要从 sqlite 转储的输出中移除一些查询，如创建表等，我们将在这里进行。

您可以使用以下单行命令：

```sql
sqlite3 db.sqlite3 .dump | grep "^INSERT INTO" | grep -v "__diesel_schema_migrations" > sqlitedump.sql ; echo -ne "SET FOREIGN_KEY_CHECKS=0;\n$(cat sqlitedump.sql)" > mysqldump.sql
```

或者逐行运行下列命令：

```sql
sqlite3 db.sqlite3 .dump | grep "^INSERT INTO" | grep -v "__diesel_schema_migrations" > sqlitedump.sql
echo "SET FOREIGN_KEY_CHECKS=0;" > mysqldump.sql
cat sqlitedump.sql >> mysqldump.sql
```

5、加载 MySQL 转储：

```sql
mysql --force --password --user=vaultwarden --database=vaultwarden < mysqldump.sql
```

6、重新启动 Vaultwarden。

_注意：使用_ _`--show-warnings`_ _加载_ _MySQL_ _转储时，会突出显示 datetime_ _字段在导入期间被截断了，这**似乎**也不会有问题。_

```
Note (Code 1265): Data truncated for column 'created_at' at row 1
Note (Code 1265): Data truncated for column 'updated_at' at row 1
```

_注意 1：加载 mysqldump.sql 数据过程中出现加载错误_

```
error (1064): Syntax error near '"users" VALUES('9b5c2d13-8c4f-47e9-bd94-f0d7036ff581'*********)
```

修复：

```bash
sed -i s#\"#\#g mysqldump.sql
```

```shell
mysql --password --user=vaultwarden
use vaultwarden
source /vw-data/mysqldump.sql
exit
```

_注意 2：如果 SQLite 数据库是从以前的某个旧版本迁移而来 ，MariaDB 可能会提示不匹配的值计数，例如：_

```
ERROR 1136 (21S01) at line ###: Column count doesn't match value count at row 1
```

由于版本跳转，可能添加了新的数据库列。首先使用 SQLite 后端升级 Vaultwarden 以在 SQLite 数据库上运行迁移，切换到 MariaDB 后端，然后重复上述迁移步骤。或者，查找自您安装的版本以来添加迁移的提交并使用 `sqlite3` 手动运行迁移。

## 外键错误、排列规则和字符集 <a href="#foreign-key-errors-collation-and-charset" id="foreign-key-errors-collation-and-charset"></a>

由于密码库中存储的某些数据是二进制或纯文本（如邮件地址、用户名或组织名称），其中可能包含 Unicode 字符，因此您需要确保正确设置数据库和表的排序规则和字符集。如果不是这种情况，则可能会在更新期间导致问题，然后生成诸如 `Cannot add or update a child row: a foreign key constraint fails ...`（无法添加或更新子行：外键约束失败...）或 `Row size too large. The maximum row size for the used table type, not counting BLOBs, is 8126.`（行大小太大。所用表类型的最大行大小（不包括 BLOB）为 8126。）之类的消息。

要解决此问题，您需要更新/更改整个数据库及其包含的表的排序规则和字符集。您可以通过您喜欢的 SQL 工具或使用 CLI 跟踪和执行以下设置来完成此操作。

在下面的示例中，我将使用数据库名称 `vaultwarden`，如果您使用不同的名称，请更改它。

在开始之前，通过运行以下两个查询来验证是否存在任何问题。它应该返回 `utf8mb4` 和 `utf8mb4_unicode_ci`。

同样要在下面的查询末尾运行这些查询以验证它是否有效！

```sql
SELECT DEFAULT_CHARACTER_SET_NAME, DEFAULT_COLLATION_NAME FROM information_schema.SCHEMATA WHERE SCHEMA_NAME = "vaultwarden";
SELECT CHARACTER_SET_NAME, COLLATION_NAME FROM information_schema.`COLUMNS` WHERE TABLE_SCHEMA = "vaultwarden" AND CHARACTER_SET_NAME IS NOT NULL;
```

首先更改数据库本身的排序规则和字符集：

```sql
ALTER DATABASE `vaultwarden` CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

然后转换所有表（包括文本字段）。执行以下命令，并复制输出：

```sql
SELECT CONCAT('ALTER TABLE `', TABLE_NAME,'` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;') AS CharSetConvert
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA="vaultwarden"
AND TABLE_TYPE="BASE TABLE";
```

这将生成几个查询，您需要执行这些查询来转换这些表的排序规则和字符集。为了使这些更改生效，我们需要暂时禁用外键检查。将上面查询生成的输出复制/粘贴到以下行的中间：

```sql
SET foreign_key_checks = 0;
-- 复制/粘贴上面的输出内容到这里
SET foreign_key_checks = 1;
```

最后，它看起来应该类似于以下内容（但根据数据库结构的更新或更改，可能会有所不同）：

```sql
SET foreign_key_checks = 0;
ALTER TABLE `__diesel_schema_migrations` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `attachments` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `ciphers_collections` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `ciphers` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `collections` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `devices` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `emergency_access` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `favorites` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `folders_ciphers` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `folders` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `invitations` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `org_policies` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `organizations` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `sends` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `twofactor_incomplete` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `twofactor` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `users_collections` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `users_organizations` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `users` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
SET foreign_key_checks = 1;
```

您需要运行这些查询以将它们转换为正确的排序规则和字符集。您可以通过对至少一张表运行以下查询来验证它是否有效：

```sql
SHOW CREATE TABLE `users`; 
```

它应该输出如下，注意最后的 `CHARSET=utf8mb4`：

```sql
CREATE TABLE `users` (
  `uuid` char(36) NOT NULL,
  `created_at` datetime NOT NULL,
  `updated_at` datetime NOT NULL,
  `email` varchar(255) NOT NULL,
  `name` text NOT NULL,
  --- CUT ---
  `enabled` tinyint(1) NOT NULL DEFAULT 1,
  `stamp_exception` text DEFAULT NULL,
  PRIMARY KEY (`uuid`),
  UNIQUE KEY `email` (`email`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
```

您可以对数据库执行相同的操作：

```sql
SHOW CREATE DATABASE `vaultwarden`;
```

它应该看起来像这样，注意 `DEFAULT CHARACTER SET utf8mb4`：

```sql
CREATE DATABASE `vaultwarden` /*!40100 DEFAULT CHARACTER SET utf8mb4 */
```

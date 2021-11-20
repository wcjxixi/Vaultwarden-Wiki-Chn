# 1.使用 MariaDB（MySQL）后端

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Using-the-MariaDB-\(MySQL\)-Backend)
{% endhint %}

{% hint style="warning" %}
我们的构建基于 MariaDB 客户端库，因为 Debian 和 Alpine 都提供这个库。我们不支持最新的 Oracle MySQLv8 版本。如果您想使用 MySQLv8 而不是 MariaDB，那么请使用旧的密码散列方式来创建用户，而不要使用默认的方式。
{% endhint %}

{% hint style="warning" %}
Alpine 目前**不支持** MariaDB（MySQL）。在 amd64 上，Alpine 支持 SQLite 和 PostgreSQL，在 arm32v7 上它仅支持 SQLite。
{% endhint %}

要使用 MySQL 后端，你可以使用[官方 Docker 镜像](https://hub.docker.com/r/bitwardenrs/server-mysql)，也可以构建你自己的启用了 [MySQL](../../deployment/building-binary.md#mysql-backend) 的二进制。

要运行二进制或容器，请确保已设置 `DATABASE_URL` 环境变量（即 `DATABASE_URL='mysql://<user>:<password>@mysql/vaultwarden'`）。

**连接字符串语法：**

```python
DATABASE_URL=mysql://[[user]:[password]@]host[:port][/database]
```

如果密码包含特殊字符，则需要使用百分号编码。

| !   | #   | $   | %   | &   | '   | (   | )   | \*  | +   | ,   | /   | :   | ;   | =   | ?   | @   | \[  | ]   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| %21 | %23 | %24 | %25 | %26 | %27 | %28 | %29 | %2A | %2B | %2C | %2F | %3A | %3B | %3D | %3F | %40 | %5B | %5D |

完整的代码列表可以在 [Wikipedia 的百分号编码页面](https://zh.wikipedia.org/wiki/%E7%99%BE%E5%88%86%E5%8F%B7%E7%BC%96%E7%A0%81)上找到。

## 使用 Docker 的示例 <a href="example-using-docker" id="example-using-docker"></a>

```python
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

### 使用非 Docker MySQL 服务器的示例 <a href="example-using-non-docker-mysql-server" id="example-using-non-docker-mysql-server"></a>

```python
Server IP/Port 192.168.1.10:3306 UN: dbuser / PW: yourpassword / DB: vaultwarden
mysql://dbuser:yourpassword@192.168.1.10:3306/vaultwarden
```

### 使用 docker-compose 的示例 <a href="example-using-docker-compose" id="example-using-docker-compose"></a>

```python
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

### 创建数据库和用户 <a href="create-database-and-user" id="create-database-and-user"></a>

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
GRANT ALTER, CREATE, DELETE, DROP, INDEX, INSERT, SELECT, UPDATE ON `vaultwarden`.* TO 'vaultwarden'@'localhost';
FLUSH PRIVILEGES;
```

### 从 SQLite 迁移到 MySQL <a href="migrating-from-sqlite-to-mysql" id="migrating-from-sqlite-to-mysql"></a>

此[话题评论](https://github.com/dani-garcia/vaultwarden/issues/497#issuecomment-511827057)中描述了一种从 SQLite 迁移到 MySQL 的简单方法。下面重复这些步骤。请注意，使用此方法风险自负，强烈建议备份您的安装和数据！

1、首先遵循上面的步骤 1 和步骤 2

2、配置 Vaultwarden 并启动它，以便 [diesel](http://diesel.rs) 可以运行迁移并正确设置模式。除此之外不要做别的。

3、停止 Vaultwarden。

4、使用下面的命令转储你现有的 SQLite 数据库。再次检查你的 sqlite 数据库的名称，默认应该是 db.sqlite。

\*\*注意：\*\*在您的 Linux 系统上需要已经安装了 sqlite3 命令。

我们需要从 sqlite 转储的输出中移除一些查询，如创建表等，我们将在这里进行。

你可以使用以下单行命令：

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

```python
Note (Code 1265): Data truncated for column 'created_at' at row 1
Note (Code 1265): Data truncated for column 'updated_at' at row 1
```

_注意 1：加载 mysqldump.sql 数据过程中出现加载错误_

```python
error (1064): Syntax error near '"users" VALUES('9b5c2d13-8c4f-47e9-bd94-f0d7036ff581'*********)
```

修复：

```python
sed -i s#\"#\#g mysqldump.sql
```

```python
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

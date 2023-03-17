# 2.构建二进制

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Building-binary)
{% endhint %}

{% hint style="info" %}
**注意**：最低支持 Rust 版本 (**MSRV**) 策略是 **N-2**，这意味着如果当前 Rust 版本是 **v1.67**，我们支持使用 **v1.65** 构建，如果是 **v1.69** 稳定版，MSRV 将是 **v1.67**。

这意味着新的稳定版发布时，新的稳定版 Rust 功能将无法使用，而必须等待另外两个版本。

为确保您使用的是稳定版本，我们强烈建议您使用 [rustup](https://rustup.rs/)，这使得安装和更新 Rust 变得非常容易。

低于 MSRV 的任何版本都会生成警告，在强制使用旧版本构建时，您只能靠自己了。
{% endhint %}

这个页面主要是给那些对 Vaultwarden 开发感兴趣，或者有特殊原因想要构建自己的二进制的用户。

普通用户应该使用从基于 Alpine 的 Docker 镜像中[提取的预构建二进制](pre-built-binaries.md)文件，[通过 Docker 部署](../container-image-usage/which-container-image-to-use.md)，或者[寻找第三方包](third-party-packages.md)。

## 依赖 <a href="#dependencies" id="dependencies"></a>

* `Rust stable`（强烈建议使用 [rustup](https://rustup.rs/)）\
  最低支持 Rust 版本 (**MSRV**) 策略是 **N-2**，这意味着如果当前 Rust 版本是 **v1.67**，我们支持使用 **v1.65** 构建
* 在基于 Debian 的发行版上，请安装以下包：`build-essential`、`git`，这些通用包可确保构建能正常进行
* `OpenSSL`（应在路径中是可用的，请参阅 [openssl crate 文档](https://docs.rs/openssl/latest/openssl/#automatic)）。在基于 Debian 的发行版上，需要安装 `pkg-config` 和 `libssl-dev`
* 对于基于 Debian 发行版上的 SQLite3 后端，需要安装 `libsqlite3-dev`
* 对于基于 Debian 发行版上的 MySQL 后端，需要安装 `libmariadb-dev-compat` 和`libmariadb-dev`
* 对于基于 Debian 发行版上的 PostgreSQL 后端，需要安装 `libpq-dev` 和 `pkg-config`
* `NodeJS`（仅当编译网页密码库时使用。使用[预构建的二进制](https://nodejs.org/en/download/)，通过系统的包管理器安装）或 [nodesource 二进制发行版](https://github.com/nodesource/distributions)。_备注：构建 web-vault 当前要求 NodeJS v16 和 NPM v8.11_

## 运行/编译 <a href="#run-compile" id="run-compile"></a>

### 所有后端 <a href="#all-backends" id="all-backends"></a>

```shell
# 使用所有后端编译并运行
cargo run --features sqlite,mysql,postgresql --release
# 或仅使用所有后端编译（二进制位于 target/release/vaultwarden）
cargo build --features sqlite,mysql,postgresql --release
```

### SQLite 后端 <a href="#sqlite-backend" id="sqlite-backend"></a>

```shell
# 使用 sqlite 后端编译并运行
cargo run --features sqlite --release
# 或仅使用 sqlite 编译（二进制位于 target/release/vaultwarden）
cargo build --features sqlite --release
```

### MySQL 后端 <a href="#mysql-backend" id="mysql-backend"></a>

```shell
# 使用 mysql 后端编译并运行
cargo run --features mysql --release
# 或仅使用 mysql 编译（二进制位于 target/release/vaultwarden）
cargo build --features mysql --release
```

### PostgreSQL 后端 <a href="#postgresql-backend" id="postgresql-backend"></a>

```shell
# 使用 postgresql 后端编译并运行
cargo run --features postgresql --release
# 或仅使用 postgresql 编译（二进制位于 target/release/vaultwarden）
cargo build --features postgresql --release
```

运行后，通过 [http://localhost:8000](http://localhost:8000/) 访问服务器。

{% hint style="warning" %}
~~**注意**：一个先前的~~[~~话题~~](https://github.com/rust-lang/rust/issues/62896)~~表明由于 Rust 编译器和 LLVM 之间存在不兼容，导致编译可能会因段错误而失败。作为解决方法，可以使用较旧版本的编译器，例如 `cargo +nightly-2019-08-27 build --features yourbackend --release`~~
{% endhint %}

### 安装 web-vault <a href="#install-the-web-vault" id="install-the-web-vault"></a>

可以从 [dani-garcia/bw\_web\_builds](https://github.com/dani-garcia/bw\_web\_builds/releases) 下载网页密码库的编译版本。

{% hint style="warning" %}
构建密码库需要约 1.5GB 的 RAM。在具有 1GB 或更小容量的 RaspberryPI 之类的系统上，请[启用交换功能](https://www.tecmint.com/create-a-linux-swap-file/)或在功能更强大的计算机上构建，然后从那里复制目录。仅构建时需要大量内存，而运行带密码库的 Vaultwarden 仅需要约 10MB 的 RAM。
{% endhint %}

如果您希望手动编译它，请遵循如下的步骤：

#### 新的（简单的方式） <a href="#new-easy-way" id="new-easy-way"></a>

克隆 [dani-garcia/bw\_web\_builds](https://github.com/dani-garcia/bw\_web\_builds) git 库：

```shell
# 克隆库
git clone https://github.com/dani-garcia/bw_web_builds.git bw_web_builds
cd bw_web_builds

# 使用 docker 作为构建环境（最安全的方式并使用正确的构建版本）
# 这将构建 web-vault，并将文件解压缩到 docker_build 目录
make docker-extract

# 改用主机提供的 npm 和节点
make full
```

#### 旧的（更手动的方式） <a href="#old-very-manual-way" id="old-very-manual-way"></a>

1、克隆 [bitwarden/clients](https://github.com/bitwarden/clients) git 库，并检查最新的发行标签（例如 v2022.6.0）：

```shell
# 克隆库
git clone https://github.com/bitwarden/clients.git web-vault
cd web-vault
# 切换到最新的标签
git -c advice.detachedHead=false checkout web-v2022.6.0
# 或者使用此版本的提交哈希
git -c advice.detachedHead=false checkout bb5f9311a776b94a33bcf0a7bff44cd87a2fcc92
```

2、根据 [apply\_patches script](https://github.com/dani-garcia/bw\_web\_builds/blob/master/scripts/apply\_patches.sh) 中的说明修补[资源](https://github.com/dani-garcia/bw\_web\_builds/tree/master/resources)中的所有镜像

3、从 [dani-garcia/bw\_web\_builds](https://github.com/dani-garcia/bw\_web\_builds/tree/master/patches) 下载补丁文件并将其复制到 `web-vault` 文件夹。补丁文件版本（假设网页密码库版本为 `vXXXX.Y.Z`）的选择：

* 如果有版本为 `vXXXX.Y.Z` 的补丁，则使用该版本
* 否则，选择小于 `vXXXX.Y.Z` 的最大的那一个版本

4、应用补丁：

```shell
# 在 web-vault 目录中
git apply vXXXX.Y.Z.patch
```

5、然后，构建密码库：

```shell
npm ci
# 阅读下方的备注（我们将其用于我们的 docker 构建）
# npm 审计修复

# 切换到 web-Vault 目录
cd apps/web
# 构建 web-Vault
npm run dist:oss:selfhost
```

{% hint style="warning" %}
可能会要求您运行 `npm audit fix` 以修复漏洞。这将自动尝试将包升级到较新的版本，该版本可能不兼容并破坏网页密码库功能。如果知道自己在做什么，请自行承担风险。顺便一提，我们会在自己的发行版中使用它！
{% endhint %}

6、最后将 `build` 文件夹的内容复制到目标文件夹中：

* 如果与 `cargo run --release` 一起运行，则目标文件夹为 `vaultwarden/web-vault`。
* 如果直接运行已编译的二进制，则它位于二进制旁，为 `vaultwarden/target/release/web-vault`。

## 配置 <a href="#configuration" id="configuration"></a>

可用的配置选项记录在默认的 `.env.template` 文件中，可以通过在该文件中取消注释所需的选项或设置它们各自的环境变量来对其进行修改。有关可用的主要配置选项，请参见本 wiki 的[配置](../configuration/)章节。

{% hint style="danger" %}
环境变量将覆盖 `.env.template` 文件中设置的值。
{% endhint %}

## 有关部署的更多信息 <a href="#more-information-for-deployment" id="more-information-for-deployment"></a>

* [配置反向代理](proxy-examples.md)
* [通过 systemd 设置自动启动](../configuration/creating-a-systemd-service.md)

## 如何为 SQLite 后端重建数据库模式（面向开发人员） <a href="#how-to-recreate-database-schemas-for-the-sqlite-backend-for-developers" id="how-to-recreate-database-schemas-for-the-sqlite-backend-for-developers"></a>

使用 cargo 安装 diesel\_cli：

```shell
cargo install diesel_cli --no-default-features --features sqlite-bundled
```

确保在 `.env` 文件中包含正确的数据库路径。

如果要修改模式，请使用以下命令创建新迁移：

```shell
diesel migration generate <name>
```

修改 `*.sql` 文件，确保在 `down.sql` 文件中还原了所有更改。

应用迁移并保存生成的模式，如下所示：

```shell
diesel migration redo

# 当使用的 diesel-cli > 1.3.0 时，此步骤会自动完成
# diesel print-schema > src/db/sqlite/schema.rs
```

## 如何从 SQLite 后端迁移到 MySQL 后端（面向开发人员） <a href="#how-to-migrate-from-sqlite-backend-to-mysql-backend-for-developers" id="how-to-migrate-from-sqlite-backend-to-mysql-backend-for-developers"></a>

如果要从 SQLite 迁移，请参考[使用 MariaDB (MySQL) 后端](../configuration/database/using-the-mariadb-mysql-backend.md)。

## 如何从 SQLite 后端迁移到 PostgreSQL 后端（面向开发人员） <a href="#how-to-migrate-from-sqlite-backend-to-postgresql-backend-for-developers" id="how-to-migrate-from-sqlite-backend-to-postgresql-backend-for-developers"></a>

如果要从 SQLite 迁移，请参考[使用 PostgreSQL 后端](../configuration/database/using-the-postgresql-backend.md)。

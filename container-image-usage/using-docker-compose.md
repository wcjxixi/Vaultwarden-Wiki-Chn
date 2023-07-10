# 4.使用 Docker Compose

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Using-Docker-Compose)
{% endhint %}

[Docker Compose](https://docs.docker.com/compose/) 是一个用于定义和配置多容器应用程序的工具。在我们的例子中，我们希望 Vaultwarden 服务器和代理都将 WebSocket 请求重定向到正确的地方。

## 带有 HTTP 挑战的 Caddy <a href="#caddy-with-http-challenge" id="caddy-with-http-challenge"></a>

本示例假定您[已安装](https://docs.docker.com/compose/install/) Docker Compose，并且您的 Vaultwarden 实例具有一个可以公开访问的域名（例如 `vaultwarden.example.com`）。

{% hint style="warning" %}
Docker Compose 可能以 `docker-compose <command> ...`（带破折号）或 `docker compose <command> ...`（带空格）运行，具体取决于您安装 Docker Compose 的方式。当 Docker Compose 作为独立的可执行文件分发时，`docker-compose` 是原始语法。您也可以选择进行[独立](https://docs.docker.com/compose/install/other/#install-compose-standalone)安装，在这种情况下将继续使用此语法。但是，Docker 目前建议将 Docker Compose 作为 Docker 插件安装，其中 `compose` 作为 `docker` 的子命令，其语法为 `docker compose <command> ...`。
{% endhint %}

首先创建一个新的目录，然后切换到该目录下。接下来，创建如下的 `docker-compose.yml` 文件，确保将 `DOMAIN` 和 `EMAIL` 变量替换为实际的值。

```yaml
version: '3'

services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: always
    environment:
      DOMAIN: "https://vaultwarden.example.com"  # 您的域名；Vaultwarden 需要知道它是 https 才能正确处理附件
    volumes:
      - ./vw-data:/data

  caddy:
    image: caddy:2
    container_name: caddy
    restart: always
    ports:
      - 80:80  #  ACME HTTP-01 验证需要
      - 443:443
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - ./caddy-config:/config
      - ./caddy-data:/data
    environment:
      DOMAIN: "https://vaultwarden.example.com"  # 您的域名，以 http 或 https 作为前缀
      EMAIL: "admin@example.com"       # 用于 ACME 注册的电子邮件地址
      LOG_FILE: "/data/access.log"
```

在相同的目录下创建如下的 `Caddyfile` 文件（此文件不需要做修改）：

```nginx
{$DOMAIN}:443 {
  log {
    level INFO
    output file {$LOG_FILE} {
      roll_size 10MB
      roll_keep 10
    }
  }

  # 使用 ACME HTTP-01 验证方式为已配置的域名获取证书
  tls {$EMAIL}

  # 此设置可能会在某些浏览器上出现兼容性问题（例如，在 Firefox 上下载附件）
  # 如果遇到问题，请尝试禁用此功能
  encode gzip

  # 将所有代理到 Rocket
  reverse_proxy vaultwarden:80 {
       # 把真实的远程 IP 发送给 Rocket，让 Vaultwarden 把其放在日志中
       # 这样 fail2ban 就可以阻止正确的 IP 了
       header_up X-Real-IP {remote_host}
  }
}
```

运行以下命令创建并启动容器。这将为 `docker-compose.yml` 文件中的服务创建私有网络，这样就只有 Caddy 暴露在外面了：

```shell
docker compose up -d # 或者 'docker-compose up -d' 如果使用独立的 Docker Compose 的话
```

停止并销毁容器：

```shell
docker compose down # 或者 'docker-compose down' 如果使用独立的 Docker Compose 的话
```

[此处](https://github.com/sosandroid/docker-bitwarden\_rs-caddy-synology)提供了一个类似的基于 Caddy 的适用于 Synology 的示例。

## 带有 DNS 挑战的 Caddy <a href="#caddy-with-dns-challenge" id="caddy-with-dns-challenge"></a>

这个示例和上一个示例一样，但适用于您不希望您的实例被公开访问的情况（即您只能从您的本地网络访问它）。这个示例使用 Duck DNS 作为 DNS 提供商。更多的背景资料，以及如何设置 Duck DNS 的细节，请参考[使用 Let's Encrypt 证书运行私有 Vaultwarden 实例](../deployment/https/running-a-private-vaultwarden-instance-with-lets-encrypt-certs.md)。

首先创建一个新的目录，然后切换到该目录下。接下来，创建如下的 `docker-compose.yml` 文件，确保将 `DOMAIN` 和 `EMAIL` 变量替换为实际的值。

```yaml
version: '3'

services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: always
    environment:
       DOMAIN: "https://vaultwarden.example.com"  # 您的域名；Vaultwarden 需要知道它是 https 才能正确处理附件
    volumes:
       - ./vw-data:/data

  caddy:
    image: caddy:2
    container_name: caddy
    restart: always
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./caddy:/usr/bin/caddy  # 您的 Caddy 自定义构建
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - ./caddy-config:/config
      - ./caddy-data:/data
    environment:
      DOMAIN: "https://vaultwarden.example.com"  # 您的域名，以 http 或 https 作为前缀
      EMAIL: "admin@example.com"        # 用于 ACME 注册的电子邮件地址
      DUCKDNS_TOKEN: "<token>"          # 您的 Duck DNS 令牌
      LOG_FILE: "/data/access.log"
```

原有的 Caddy 构建（包括 Docker 映像中的构建）不包含 DNS 挑战模块，因此接下来您需要[获取自定义 Caddy 构建](../deployment/https/running-a-private-vaultwarden-instance-with-lets-encrypt-certs.md#getting-a-custom-caddy-build)。将自定义构建重命名为 `caddy` 并将其移动到与 `docker-compose.yml` 相同的目录下。确保 `caddy` 文件是可执行的（例如 `chmod a + x caddy`）。上面的 `docker-compose.yml` 文件会将自定义构建绑定挂载到 `caddy:2` 容器中，并替换原有的构建。

在相同的目录下，创建如下的 `Caddyfile` 文件（此文件不需要做修改）。

```nginx
{$DOMAIN}:443 {
  log {
    level INFO
    output file {$LOG_FILE} {
      roll_size 10MB
      roll_keep 10
    }
  }

  # 使用 ACME HTTP-01 验证方式为已配置的域名获取证书
  tls {
    dns duckdns {$DUCKDNS_TOKEN}
  }

  # 此设置可能会在某些浏览器上出现兼容性问题（例如，在 Firefox 上下载附件）
  # 如果遇到问题，请尝试禁用此功能
  encode gzip

  # 将所有代理到 Rocket
  reverse_proxy vaultwarden:80
}
```

与 HTTP 挑战的示例一样，运行下面的命令以创建并启动容器：

```shell
docker compose up -d # 或者 'docker-compose up -d' 如果使用独立的 Docker Compose 的话
```
